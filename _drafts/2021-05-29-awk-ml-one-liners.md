---
title: "Awk Machine Learning One Liners"
date: 2021-05-29 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - Engineering
excerpt: Here's a cheat sheet of awk one liners for machine learning metrics you might want during fast ad hoc analysis.
toc: true
toc_sticky: true
author_profile: true
classes: wide
---

# Intro

Here's a cheat sheet for awk one liners for machine learning metrics,
like precision, recall, AUC, RMSE, and so forth.

You can skip to the 
<a href="#cheat-sheet">Cheat Sheet</a>
now.
The cheat sheet uses a vowpal wabbit model on the RCV1 benchmark data set,
and I outline the procedure on producing in 
<a href="#setup">Setup</a>.
This post is built backwards for ease of repeated reference. 

# Cheat Sheet

Full gist available 
[here](https://gist.github.com/fwhigh/7745ea6f278f31f2f4e525b7a00df273). 

```bash
# 500k-1 random predictions
awk -v n_lines=499999 'BEGIN {for (i=0; i<n_lines; i++) {print int(rand()>0.5),rand()}}' > label_pred
```

First 10 lines: 

```
1 0.394383
1 0.79844
1 0.197551
0 0.76823
0 0.55397
0 0.628871
0 0.513401
1 0.916195
1 0.717297
0 0.606969
```

## Accuracy

{% include gist_embed.html data_gist_id="fwhigh/7745ea6f278f31f2f4e525b7a00df273" data_gist_hide_footer="true" data_gist_file="ml_one_liners.sh" data_gist_line="1-2" %}

## Precision

{% include gist_embed.html data_gist_id="fwhigh/7745ea6f278f31f2f4e525b7a00df273" data_gist_hide_footer="true" data_gist_file="ml_one_liners.sh" data_gist_line="4-5" %}

## Recall

{% include gist_embed.html data_gist_id="fwhigh/7745ea6f278f31f2f4e525b7a00df273" data_gist_hide_footer="true" data_gist_file="ml_one_liners.sh" data_gist_line="7-8" %}

## Mean Squared Error

{% include gist_embed.html data_gist_id="fwhigh/7745ea6f278f31f2f4e525b7a00df273" data_gist_hide_footer="true" data_gist_file="ml_one_liners.sh" data_gist_line="10-11" %}

## AUC with sort

{% include gist_embed.html data_gist_id="fwhigh/7745ea6f278f31f2f4e525b7a00df273" data_gist_hide_footer="true" data_gist_file="ml_one_liners.sh" data_gist_line="13-14" %}

## AUC without sort

{% include gist_embed.html data_gist_id="fwhigh/7745ea6f278f31f2f4e525b7a00df273" data_gist_hide_footer="true" data_gist_file="ml_one_liners.sh" data_gist_line="16-17" %}

## Cross entropy

# Setup

Let's set up one model for binary logistic regression problem
and one for a continuous regression problem. 

We'll get set up with the world's greatest ML command line utility, 
[Vowpal Wabbit](https://vowpalwabbit.org/start.html).

```bash
brew install wget
```

```bash
mkdir vw_example
cd vw_example
```

## Build vw and perf from source

You can 

```bash
brew install vowpal-wabbit
```

but you won't get some of vw's utility commands that are useful for looking at the models, for example.
Here's how I installed from source.

Install from source on MacOS. My procedure involved variants of the recommended procedures
* [vw pythoon osx](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Python#osx)
* [vw macos deps](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies#macos)
* [vw building on linux](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#linux)
* [perf](http://osmot.cs.cornell.edu/kddcup/software.html)

Summary of my tips:
* `alias nproc="sysctl -n hw.logicalcpu"`
* `alias vw='<path-to>/vowpal_wabbit/build/vowpalwabbit/vw'`
* use `gzcat` instead of `zcat`
* install perf from source at http://osmot.cs.cornell.edu/kddcup/software.html
* `alias perf='/Users/fwhigh/repos/perf.src/perf'`

```bash
# vw prereqs
brew install cmake
brew install boost
brew install flatbuffers
brew install zlib
brew install boost-python3

# build and alias vw
git clone https://github.com/VowpalWabbit/vowpal_wabbit.git
cd vowpal_wabbit
mkdir build
cd build
cmake ..
alias nproc="sysctl -n hw.logicalcpu"
make -j $(nproc) vw-bin
alias vw=$(pwd)/vw

# build and alias perf
cd ..
wget http://osmot.cs.cornell.edu/kddcup/perf/perf.src.tar.gz
tar xzvf perf.src.tar.gz
cd perf.src
rm perf
make 
alias perf=$(pwd)/per
```

## Train a model

* [Rcv1-example](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Rcv1-example)
* [leon bottou sgd with rcv1 benchmark](https://leon.bottou.org/projects/sgd)

```bash
# get the data
wget http://hunch.net/~vw/rcv1.tar.gz
tar xzvf rcv1.tar.gz
cd rcv1

# train
vw rcv1.train.vw.gz --cache_file cache_train -f r_temp
```

Output:

```
final_regressor = r_temp
Num weight bits = 18
learning rate = 0.5
initial_t = 0
power_t = 0.5
creating cache_file = cache_train
Reading datafile = rcv1.train.vw.gz
num sources = 1
Enabled reductions: gd, scorer
average  since         example        example  current  current  current
loss     last          counter         weight    label  predict features
1.000000 1.000000            1            1.0   1.0000   0.0000       50
0.511062 0.022124            2            2.0   0.0000   0.1487      103
0.260481 0.009900            4            4.0   0.0000   0.0418      134
0.237231 0.213981            8            8.0   0.0000   0.1846      145
0.247655 0.258079           16           16.0   1.0000   0.2879       23
0.242766 0.237877           32           32.0   0.0000   0.1818       31
0.236583 0.230399           64           64.0   0.0000   0.1566       60
0.229860 0.223137          128          128.0   1.0000   0.8289      105
0.186476 0.143092          256          256.0   1.0000   0.8526      122
0.156768 0.127060          512          512.0   0.0000   0.4880       74
0.128087 0.099406         1024         1024.0   0.0000   0.3153       99
0.102213 0.076338         2048         2048.0   1.0000   0.6569       31
0.088079 0.073944         4096         4096.0   0.0000   0.2314      114
0.077332 0.066586         8192         8192.0   0.0000   0.5016      103
0.067194 0.057056        16384        16384.0   1.0000   1.0000       49
0.059360 0.051526        32768        32768.0   1.0000   0.9464      103
0.053951 0.048541        65536        65536.0   1.0000   1.0000       33
0.049552 0.045153       131072       131072.0   1.0000   1.0000       50
0.046298 0.043045       262144       262144.0   0.0000   0.0000       49
0.043206 0.040114       524288       524288.0   0.0000   0.4752      106

finished run
number of examples = 781265
weighted example sum = 781265.000000
weighted label sum = 370541.000000
average loss = 0.041728
best constant = 0.474283
best constant's loss = 0.249339
total feature number = 59936409
```

```bash
# predict
vw -t --cache_file cache_test -i r_temp -p predictions rcv1.test.vw.gz
```

Output:

```
only testing
predictions = predictions
Num weight bits = 18
learning rate = 0.5
initial_t = 0
power_t = 0.5
creating cache_file = cache_test
Reading datafile = rcv1.test.vw.gz
num sources = 1
Enabled reductions: gd, scorer
average  since         example        example  current  current  current
loss     last          counter         weight    label  predict features
0.000000 0.000000            1            1.0   0.0000   0.0000       40
0.000000 0.000000            2            2.0   1.0000   1.0000       74
0.000000 0.000000            4            4.0   0.0000   0.0000       43
0.000000 0.000000            8            8.0   0.0000   0.0000       41
0.007351 0.014703           16           16.0   1.0000   1.0000       33
0.006755 0.006158           32           32.0   0.0000   0.0000      209
0.017643 0.028532           64           64.0   1.0000   1.0000       24
0.019878 0.022112          128          128.0   1.0000   1.0000       44
0.066147 0.112416          256          256.0   0.0000   0.0000       48
0.056477 0.046807          512          512.0   1.0000   1.0000       32
0.046589 0.036701         1024         1024.0   0.0000   0.0000      123
0.041715 0.036840         2048         2048.0   1.0000   0.8463       63
0.041956 0.042198         4096         4096.0   0.0000   0.0000       98
0.042282 0.042607         8192         8192.0   0.0000   0.0000       63
0.042623 0.042964        16384        16384.0   0.0000   0.0000      226

finished run
number of examples = 23149
weighted example sum = 23149.000000
weighted label sum = 10786.000000
average loss = 0.042671
best constant = 0.465938
best constant's loss = 0.248840
total feature number = 1775682
```

```bash
gzcat rcv1.test.vw.gz | cut -d ' ' -f 1 | sed -e 's/^-1/0/' > labels
perf -files labels predictions -t 0.5
```

Output:

```
ACC    0.94609   pred_thresh  0.500000
PPV    0.94280   pred_thresh  0.500000
NPV    0.94895   pred_thresh  0.500000
SEN    0.94141   pred_thresh  0.500000
SPC    0.95017   pred_thresh  0.500000
PRE    0.94280   pred_thresh  0.500000
REC    0.94141   pred_thresh  0.500000
PRF    0.94210   pred_thresh  0.500000
LFT    2.02345   pred_thresh  0.500000
SAR    0.90696   pred_thresh  0.500000 wacc  1.000000 wroc  1.000000 wrms  1.000000 

ACC    0.94635   freq_thresh  0.495708
PPV    0.94243   freq_thresh  0.495708
NPV    0.94977   freq_thresh  0.495708
SEN    0.94243   freq_thresh  0.495708
SPC    0.94977   freq_thresh  0.495708
PRE    0.94243   freq_thresh  0.495708
REC    0.94243   freq_thresh  0.495708
PRF    0.94243   freq_thresh  0.495708
LFT    2.02264   freq_thresh  0.495708
SAR    0.90705   freq_thresh  0.495708 wacc  1.000000 wroc  1.000000 wrms  1.000000 

ACC    0.94643   max_acc_thresh  0.495069
PPV    0.94235   max_acc_thresh  0.495069
NPV    0.95000   max_acc_thresh  0.495069
SEN    0.94270   max_acc_thresh  0.495069
SPC    0.94969   max_acc_thresh  0.495069
PRE    0.94235   max_acc_thresh  0.495069
REC    0.94270   max_acc_thresh  0.495069
PRF    0.94253   max_acc_thresh  0.495069
LFT    2.02249   max_acc_thresh  0.495069
SAR    0.90708   max_acc_thresh  0.495069 wacc  1.000000 wroc  1.000000 wrms  1.000000 

PRB    0.94243
APR    0.97666
ROC    0.98137
R50    0.19184
RKL    23149
TOP1   0.00000
TOP10  0.00000
SLQ    0.83526 Bin_Width  0.010000
CXE   82422566849539793746477389187308009598421579761088416023344820680303518007160822351570676024868864.00000
RMS    0.20657
CA1    0.04811 19_0.05_bins
CA2    0.02204 Bin_Size 100
```

Stats for the data set:
* 47,152 features
* 781,265 training instances with XXX features
* 23,149 test instances

# Awk Basics

## Line By Line Processing

Awk in its simplest for runs on a text file line by line. 
You can create a text file with `seq 10 > seq10.txt` and add 5 to each line like this:

```bash
awk '{print $1 + 5}' seq10.txt
```

I'll make heavy use of 
[bash process substutition](https://tldp.org/LDP/abs/html/process-sub.html)
to short circuit the file-writing part. 

```bash
awk '{print $1 + 5}' <(seq 10)
```

The process substitution syntax produces a named pipe that works basically like a file.

```bash
file <(seq 10)
# Output: /dev/fd/63: fifo (named pipe)
```

## BEGIN and END

