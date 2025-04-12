---
title: "The Command Line"
date: 2025-04-10 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  # overlay_image: /assets/images/chris-ried-ieic5Tq8YMk-unsplash.jpg
categories: 
  - Advanced Topics for Data Science
  - Featured
excerpt: The command line is a gateway to advanced data science.
toc: true
toc_sticky: true
author_profile: true
sidebar:
  nav: atds-sidebar
---

*This is an installment in the series 
Advanced Topics for Data Science.*

# Settings things up

## Open Terminal on a Mac and Change Settings

Most of us use Macs so let's jump right into the command line. 
Open the Terminal application by pressing command-space, typing "terminal" (this should highlight the Terminal application and the action should be Open), and hitting enter.

Immediately change the default settings. Click the Terminal menu, then Settings. 
Under Profiles select the Keyboard tab and check "Use Option as Meta key".
This will give you some nice keystroke shortcuts that you would normally expect in a Linux shell.

{% include figure popup=true image_path="/assets/images/atds-the-command-line/terminal-option-as-meta.png" alt="Select Use Option as Meta key" caption="Select Use Option as Meta key." %}

Under Profiles select the Window tab and toggle "Limit number of rows to" and set it to 10,000.
This prevents Terminal from crashing your computer if you accidently `cat` an enormous file, say.

{% include figure popup=true image_path="/assets/images/atds-the-command-line/terminal-limit-rows.png" alt="Set the max number of rows to 10,000" caption="Set the max number of rows to 10,000." %}

Show what directory you are in. "Present working directory".
```bash
pwd
```

Shows the user name you are logged in as.
```bash
whoami
```

List contents of directory. The options show extended information (`l` for long), 
human readable file sizes (`h`), reverse (`r`) time (`t`) ordered.
```bash
ls -lhart
```

Create a nested directory recursively (`p`). In this case `repos` didn't exist yet so it will also get created.
If `repos` did already exist, the command will not error out.
```bash
mkdir -p ~/repos/tmp
```

Change to a new working directory. 
We can work from here to mess around and delete the whole thing later, if needed.
```bash
cd ~/repos/tmp
```

## Install Oh My Zsh

Follow the instructions at [https://ohmyz.sh/](https://ohmyz.sh/) to install Oh My Zsh,
a user friendly zsh management framework. Zsh is the default Mac shell.

## Setting up Xcode and Homebrew

Install Mac "Xcode" developer utilities.
```bash
xcode-select --install
```

Follow the instructions at [https://brew.sh/](https://brew.sh/) to Install homebrew.
You'll get a ton of command utilities from Homebrew. Requires Xcode developer utilities to be installed.
Install Homebrew using the command provided at their website.

## Setting up Python

It is very easy to clutter your Python environment during development.
The goal of this setup is to give you full control over your Python development environment, 
including letting you set the Python distribution version easily,
select between multiple of them,
and blow away entire installations to then be reproduced from scratch using virtual environments.

I'm giving two options below, first pyenv based then uv based. 
They are mutually exclusive so you must pick just one.

### Using pyenv

Install `pyenv` immediately to keep your Python installations clean and isolated.
```bash
brew install pyenv
```

Install your favorite Python release. 
Here I'm setting it to the latest Python 3 at the time of writing.
```
pyenv install 3.12.3
````

Set the Python release you just installed as your default. Validate with `which python` (should be ~/.pyenv/shims/python)
and `python -V`.
```bash
pyenv local 3.12.3
```

Create a Python virtual environment.
```bash
python -m venv venv
```

Or `source venv/bin/activate` to activate your new Python virtual environment. Pip or uv install away! 
You have a fully isolated Python environment that you can clutter up as needed.
```bash
. venv/bin/activate
```

Deactivate your Python
```bash
deactivate
```

Blow away your Python virtual environment. Needs to be recursive (`r`) and ensure that you will not get an interactive prompt for every deletion `f`, even if you or someone else has aliased `rm` to `rm -i`.
This is your nuclear option when you mess up the environment, which happens to everybody.
After blowing it away you can re-create the virtual environment `venv ` from scratch again.
This is your full Python virtual environment lifecycle for local development.
```bash
rm -rf venv
```

### [Not yet tested] Using uv

Great guidance at [https://bluesock.org/~willkg/blog/dev/switch_pyenv_to_uv.html](https://bluesock.org/~willkg/blog/dev/switch_pyenv_to_uv.html) in case you already have a pyenv installation, which you'll need to remove.

Install uv following [these instructions](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer).
Validate the installation with `uv --version`.

Install Python versions.
```bash
uv python install 3.10 3.11 3.12
```

## Setting up git

Git comes with Xcode developer tools.

Set your global default user name for git.
```bash
git config --global user.name "Your Name"
```

Set your global deafult email for git. You can always override these for any local git repo.
```bash
git config --global user.email "your_email@example.com"
```

# Analysis at the command line

Having lots of processing power and memory can make us lazy. 
This comes with advantages and disadvantages. 
It's great because we can spend our time on higher level concerns in some analytical or scientific area of expertise and we can use higher level languages that more people know and are therefore more maintainable by a larger workforce.
But it also means we may not learn some computer science and algorithm basics that we would otherwise really benefit from.
Learning about machine learning, deep neural networks, or causal inference, or other areas of advanced analysis are highly valuable skills.
Learning about efficient data processing on top of that is one thing that distiguishes a junior from a senior level professional.
It does make you a better data scientist.

The command line is one way to learn about efficient data processing because many of the standard utilities were developed at a time when processing and memory were small and expensive.
The environment forced programmers to write efficient code so that they weren't sitting around waiting for commands to complete all the time.

One key concept we can learn from using the command line is stream processing.
Stream processing means loading just one or a small number of rows at a time and running sequentially through a file or other stream of data.
Another name for it that data scientists will be more familiar with is batching or mini-batching.
In other contexts you may see it referrred to as out-of-core processing or online processing.
You can practice stream processing on the command line and then try to re-implement the same algorithms in higher level languages like Python.

How would you process a one billion line CSV file on your local machine?
A float is 4 bytes so a billion floats is 4 GB. 
Most of us have at least 8 GB of RAM so maybe we'd probably just try to load it in Python via Jupyter.
If it's a CSV with 10 columns this becomes 40 GB and loading the entire file is not possible.

Let's try it out. I'll start by setting a variable for the number of rows to generate.
Let's start with one million to feel our way around.
```bash
N_ROWS=10000000
time yes | head -n $N_ROWS | awk -v OFS=, '{a=rand(); for (i=1; i<10; i++) a = a OFS rand(); print a }' > /tmp/unreasonable.csv
```
We're writing this to `/tmp` because we don't want to have to remember to delete it.
At the time of writing, MacOS deletes `/tmp` contents on startup.
I can validate by counting rows with `wc -l /tmp/unreasonable.csv` and inspecting the file with `less /tmp/unreasonable.csv`.
The size of the file generated with `du -sh /tmp/unreasonable.csv` is 96 MB. 
This is twice the size I expected from a 4 bytes $\times$ 1000000 calculation, which I attribute to this being an ASCII file representing float data rather than a more efficiently serialized object containing native float types.
Bumping `N_ROWS` to 10 million pushes the file to about 1 GB and 100 million rows to 10 GB. Pretty unreasonable territory. Do try this exercise with larger `N_ROWS` but be aware of your available storage space because if you exceed it you will crash your computer.

How many columns are there?
```bash
head -n 1 /tmp/unreasonable.csv | awk -F, '{print NF}'
```
How many rows?
```bash
wc -l /tmp/unreasonable.csv
```
What's the mean of each column? Now it's starting to get interesting.
```bash
time awk -F, '
{
  for (i=1; i<=NF; i++) {
    a[i] += $i
  }
} 
END {
  for (i=1; i<=NF; i++) {
    print i, a[i]/NF
  }
}' /tmp/unreasonable.csv
```
How might I do this quickly on the command line with Python?
```bash
time python - <<EOF /tmp/unreasonable.csv
import sys
a = {}
n = 0
with open(sys.argv[1], 'r') as file: 
    while line := file.readline():
        for i, item in enumerate(line.split(',')):
            if n == 0:
                a[i] = 0.0
            a[i] += float(item)
        n += 1
print({i+1: a[i]/n for i in a})
EOF
```

What's the mean of each column?

Command line utilities are a marvel and a treasure. 
Let me start by generating 100 million random numbers, then computing their average. 
First in Python, which I will write as a command one-liner.
```bash
time python - <<EOF > random_numbers_python
import random
for i in range(100_000_000): print(random.random())
EOF
```
This took me 68.63 seconds. 
The equivalent using old command line utilities in one line.
```bash
time yes | head -n 100000000 | awk '{print rand()}' > random_numbers_cl
```
This took 41.28 seconds. Not a big difference.
Now measuring the average.
```bash
time python - <<EOF <(yes | head -n 100000000 | awk '{print rand()}')
import sys
a = float(0)
n = 0
with open(sys.argv[1], 'r') as file: 
    while line := file.readline():
        a += float(line)
        n += 1
print(a/n, a, n)
EOF
```
This took 32.59 seconds and I got the mean value of 0.5000270049640401. 
Now with awk.
I'll use the same Python generated numbers just to compare answers and level the playing field.
```bash
time yes | head -n 100000000 | awk '{print rand()}' | awk '{a += $1} END {print a/NR " " a " " NR}'
```
This completed in 51.75 seconds and gave me a value of 0.500027.

```bash
time python - <<EOF 
import random
a = float(0)
n = 0
for i in range(100_000_000):
    a += random.random()
    n += 1
print(a/n, a, n)
EOF
```
11.17s

```bash
time awk 'BEGIN {for (i=0; i<1000000000; i++) {a += rand(); n++} print a/n " " a " " n}'
```
8.90s



The command line is an extremely useful straight jacket because it forces you to think and write in a style that is condusive to fast and efficient processing.
These programs are fast because they were written for Unix at a time when memory and compute speed were a fraction of what we now take for granted.
They had to be fast because otherwise they'd be spending much of their lives waiting for commands to complete.
Even today they can beat almost any other kind of program that you would otherwise build in Python or other high level languages on the most basic processing and analysis tasks.

Learning why exactly they are efficient teaches you about practical computer science.


## German Credit Risk UCI data set

Let's fetch, analyze, and build an ML model around the public UCI data set.
This one is easier because it's a binary class prediction task against 1000 instances,
about 20 features, and no missing values.

Download from [openml](https://www.openml.org/search?type=data&sort=runs&status=active&id=31) and move it to your local working directory.
```bash
mv ~/Downloads/dataset_31_credit-g.arff .
```

Look at the data with `less`. Can also do `head less german_credit_data.csv` to just print the first 10 lines.
```bash
less german_credit_data.csv
```

Count the number of lines. I get 1001.
```bash
wc -l german_credit_data.csv
```

Install [Vowpal Wabbit](https://vowpalwabbit.org/start.html) for command line ML.
Validate with `which vw`.
See their [command line tutorial](https://vowpalwabbit.org/start.html).
```bash
brew install vowpal-wabbit
```

```bash
/^f/ {print f[NR]=$2; print f[NR]}
```

```bash
awk -F, '
/^f/
'
<(grep '^@attribute' dataset_31_credit-g.arff | cut -d\' -f 2 | awk '{print "f,"$1}')

```

abc
```bash
awk -F, '
/^@attribute/ {
  print
} 
/^[^%@]/ {
  print
}
' dataset_31_credit-g.arff | head
```

This one liner formats the CSV file
```bash
awk -F, 'NR > 1 {print $2}' < german_credit_data.csv | head
```
