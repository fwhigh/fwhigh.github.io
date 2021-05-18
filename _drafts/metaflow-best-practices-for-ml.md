---
title: "Metaflow Best Practices for Machine Learning"
date: 2021-05-24 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - machine learning
  - engineering
  - featured
tags:
  - metaflow
excerpt: I've compiled a list of best practices for using Metaflow for machine learning from my years of using it at Netflix.
toc: true
toc_sticky: true
<!-- classes: wide -->
---

# Overview

Here's a list of ideas 
tested over roughly four or so years of experience with Metaflow
that result in 
an extremely short development loop;
reusable, maintainable, and reliable code;
and just an overall fun and rewarding developer experience. 

# Best Practices


## Use an IDE 

I prefer PyCharm.
Debugging seems to work fine in PyCharm, but it can be a bit tricky
to debug parallel tasks in foreach steps. 
Using test-mode 
(<a href="#implement-a-test-mode">Implement a test mode</a>)

## Pull code into reusable functions and classes


## Keep the scope of each step small


## Keep each step short 

I like to
1. Pull reusable code into common scripts in the flow spec directory structure.
1. Make a separate Python package and install it on the fly in every step
where it makes sense.

1 is easiest. If your flow is flow.py, just pull big chunks of code into functions 
and put them into another python file like common.py. 
You can brute-force test the code in a Jupyter notebook with autoreload enabled.
This makes for a super short development loop.

2 is useful when you can reuse code across multiple flows or projects, 
and you want to harden that code even further with unit tests,
integration tests, and open sourcing, for example.
If you make a package and put it in Github, for example,
then in each step you can install from source directly with
`pip install git+ssh://git@github.com/fwhigh/metaflow-helper.git`.
You can pin by pointing to a specific commit or tag, e.g.
`pip install git+ssh://git@github.com/fwhigh/metaflow-helper.git@v0.0.1`
or
`pip install git+ssh://git@github.com/fwhigh/metaflow-helper.git@00db203`.
Once the package matures you might push it to PyPI or conda
and install with
`pip install metaflow-helper==0.0.1` (similarly for conda).

## Use Jupyter notebooks for testing and debugging

Keep a Jupyter sandbox notebook running with autoreload enabled to debug code and data artifacts.

Put the following in a cell before importing any libraries. 
This will let you re-execute functions from your libraries or modules *at the cell level*
without having to re-import.

```python
%load_ext autoreload
%autoreload 2
```

Then change your working directory to the one containing your Metaflow flow spec.

```python
import os
os.chdir(os.path.join('examples', 'model-selection'))
```

Then import your common code 
(suggestion 1 from <a href="#keep-each-step-short">Keep each step short</a>)
with 

```python
import common
```

Now when you use `result = common.some_function()` in a cell and make changes to 
`some_function` in commony.py, 
you can just rerun the cell and it'll reflect the changes.

## Use Jupyter notebooks for inspecting data artifacts

Where these practices get really powerful is
when you combin this step with 
<a href="#use-jupyter-notebooks-for-testing-and-debugging">Use Jupyter notebooks for testing and debugging</a>.
Add an import cell like this.

```python
from metaflow import Flow, Run, Step
```

Now you can load data artifacts from a given run ID and 
*test new common.py code directly on the data, on the fly*.
If my flow spec is called `Train` and I'm testing
a step (can be a foreach parallel step) called 
`foreach_fold`, I can acces the first
task's data pointer like this.

```python
run_id = '1621297047781971'
data = list(Step(f'Train/{run_id}/foreach_fold'))[0].data
```

If I want to look at the dataframe I've persisted there I do

```python
df = data.df
```

This loads the data on the fly from the Metaflow datastore, 
which may be in the cloud on, say, AWS S3.
Loading the data once into a local object in memory
is a good idea. Now you can use the data 
in common.py functions.

```python
result = common.some_function(df)
```

Make a change to `some_function` in common.py and 
you can rerun just the above line in a cell 
without having to reimport the code
or reload the data.

## Implement a test mode

Use a flag parameter that you can use to 
subset the data and reduce parallelism to 1 concurrent task,
if at all possible. 
If you're training a model,
reduce the number of maximum possible optimization iterations to
something small like 10 epochs.

```python
from metaflow import FlowSpec, Parameter, step
import commmon


class Train(FlowSpec):

    test_mode = Parameter(
        'test_mode',
        help="Run in test mode?",
        type=bool,
        default=False,
    )

    @step
    def start(self):
    	if self.test_mode:
    		# Get a subset of data and reduce parallelism here
 			self.df = common.get_dataframe(max_rows=100)
 			self.max_epochs = 10
 			self.patience = 1
 		else:
 			self.df = common.get_dataframe()
 			self.epochs = 10_000
 			self.patience = 50
```

I did a variant of this in my 
[model selection example](https://github.com/fwhigh/metaflow-helper/tree/main/examples/model-selection)
from 
[LightGBM vs Keras Model Selection At Scale Using Metaflow]({% post_url 2010-07-21-lightgbm-vs-keras-metaflow %}).
Instead of using a boolean flag I point to different configuration files by string,
some of which perform the same tasks of 
subsetting the data down and shortening the model training times dramatically.

## Run flows in test mode in CI/CD pipeline
