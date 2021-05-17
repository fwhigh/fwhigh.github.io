---
title: "Parallel Grep and Awk"
date: 2021-03-21 12:00:00 -0700
comments: true
author: "Will High"
categories: 
  - big data
  - streaming algorithms
tags:
  - bash
  - grep
  - awk
  - gnu parallel
  - featured
excerpt: I get a nearly 6x speedup over standard grep by using GNU parallel.
---

{% include toc %}

# tl;dr 

I get a 6x speedup on a vanilla grep task by using 
[GNU parallel](https://www.gnu.org/software/parallel/)
(`brew install parallel`). 
I get 4x speedup on an awk task computing feature cardinality for a machine learning problem.

# Overview

This post is about speeding up grep and awk on a single machine. 
It's a modified repost of a Wordpress article I wrote a few years ago. 
I thought it was cool enough to bring over to my new blog home, 
and in doing so I'm seeing the 3x gain I originally got on the grep task go up to 6x. 
Awesome. 

# Parallel grep

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

# Parallel feature cardinality with awk

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
