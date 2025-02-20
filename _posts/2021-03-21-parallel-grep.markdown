---
title: "Parallel Grep and Awk"
date: 2021-03-21 12:00:00 -0700
last_modified_at: 2021-06-03 12:00:00 -0700
comments: true
author: "Will High"
categories: 
  - Engineering
  - Featured
excerpt: I get a nearly 6x speedup over standard grep by using GNU parallel.
classes: wide
---

{% include toc %}

# tl;dr 

<!-- 
2990406 appears at the top of kddb
13653924 appears in both kddb and kddb.t 
-->

Get 
[GNU parallel](https://www.gnu.org/software/parallel/)
(e.g. `brew install parallel`, `apt-get install parallel`, etc.).

Run grep in parallel blocks on a single file.

```bash
parallel --pipepart --block 10M -a <filename> -k grep <grep-args>
```

Run grep on multiple files in parallel, 
in this case all files in a directory and its subdirectories.
Add `/dev/null` to force grep to prepend the filename to the matching line.

```bash
find . -type f | xargs -n 1 -P 4 grep <grep-args> /dev/null
```

Run grep in parallel blocks on multiple files in serial.
Manually prepend the filename since grep can't do it in this case.

```bash
# for-loop
for filename in `find . -type f`
do 
  parallel --pipepart --block 10M -a $filename -k "grep <grep-args> | awk -v OFS=: '{print \"$filename\",\$0}'"
done

# using xargs
find . -type f | xargs -I filename parallel --pipepart --block 10M -a filename -k "grep <grep-args> | awk -v OFS=: '{print \"filename\",\$0}'"
```

Run grep in parallel blocks on multiple files in parallel.
Take care to prepend the filename since grep can't do it in this case.
Warning, this may be an inefficient use of multithreading.

```bash
find . -type f | xargs -n 1 -P 4 -I filename parallel --pipepart --block 10M -a filename -k "grep <grep-args> | awk -v OFS=: '{print \"filename\",\$0}'"
```

# Parallel grep on one file

Say I want to know how many times feature "15577606" appears in the KDD CUP 2010 
[kddb LIBSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary.html)
machine learning benchmark training set.
This is a binary classification data set 
containing 19 million lines (each line is a feature vector) and 30 million features  -- a large grep task.

```bash
wget http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary/kddb.bz2
bunzip2 kddb.bz2
time grep 15577606 kddb > /dev/null
```

A standard grep takes me 1m24s. Grep picks out just 199 lines containing that feature. 

[The GNU parallel utility](https://www.gnu.org/software/parallel/)
gives me a nice **5.6x speedup at 15s** using multiple threads.

```bash
time \
  parallel --pipepart --block 10M -a kddb -k grep 15577606 \
  > /dev/null
```

# Parallel feature cardinality with awk on one file

Now I'll use this to do something useful: count the occurrence of each of the 
$O(10^7)$ features in the training file. 
I'll use a map-reduce pattern. 
In the map phase I'll run "feature_count.awk" with the following contents.

```awk
#!/usr/bin/awk -f 

{
  # loop over each feature but skip the label
  for (i = 2; i <= NF; i++) {
    # split the feature at the ':' character
    split($i, a, ":")
    # count the number of times the feature appears
    n[a[1]]++
  }
}
END {
  for (i in n) {
  	# print out the feature and its count
    print i, n[i]
  }
}
```

The reduce stage is an awk one liner that adds the counts by feature. 
Naively you would run it like this.

```bash
time ./feature_count.awk kddb | \
  awk '{n[$1] += $2} END {for (i in n) {print i, n[i]}}' > \
  kddb_feature_count_naive.txt
```

This takes me 19m50s. With `parallel` you could do

```bash
time parallel --pipepart --block 10M -a kddb -k ./feature_count.awk | \
  awk '{n[$1] += $2} END {for (i in n) {print i, n[i]}}' > \
  kddb_feature_count.txt
```

GNU parallel gives me 4m50s -- a 4.1x speedup.

# What I learned about the data

A quick analysis of the feature statistics follows. 

First thing I learned, 72% of the features appear in the training set just once, as in, 
they appear in just one single feature vector. This is a red flag because
normally you'd want features to appear many times for the model to learn
something generalizable from them. 

I'll do a separate feature cardinality run on the test data set.

```bash
wget http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary/kddb.t.bz2
bunzip2 kddb.t.bz2
./feature_count.awk kddb.t > kddb.t_feature_count.txt
```

There are 2,990,384 features in the test set. 

The superset of all features in the training and test sets is 29,890,095.

```bash
# set union for first column of two files 
awk '
!($1 in n) {m++; n[$1]} 
END {print m}
' kddb.t_feature_count.txt kddb_feature_count.txt
```

But only 7% of the test set features appear in the training set.

```bash
# set intersection for first column of two files 
awk '
NR == FNR {n[$1]} 
NR > FNR && ($1 in n) {m++} 
END {print m}
' kddb.t_feature_count.txt kddb_feature_count.txt
```

Even fewer (4%) make more than 10 appearances in the training set.

```bash
# set intersection plus filter for first column of two files 
awk -v min_occurrences=10 '
NR == FNR && $2 > min_occurrences {n[$1]} 
NR > FNR && ($1 in n) {m++} 
END {print m} ' kddb_feature_count.txt kddb.t_feature_count.txt
```

If I were to build an ML model for this task, 
I would remove features that appear less than 10 times in the training set
as a preprocessing step.
Here's a quick way to generate a feature include-list.

```bash
# features that appear more than min_occurrences times
awk -v min_occurrences=10 '
$2 > min_occurrences {print $1} 
' kddb_feature_count.txt > kddb_feature_include_list.txt
```

That's 3,814,194 eligible features in the training set, 
13% of the original dimensionality. 
This would bring nice speedups to model training and prediction, at no
cost to accuracy.
