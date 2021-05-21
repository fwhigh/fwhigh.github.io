---
title: "Tips for a Short and Enjoyable Machine Learning Development Cycle Using Metaflow"
date: 2021-05-28 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - Machine Learning
  - Engineering
  - Featured
  - Metaflow
excerpt: The combination of an IDE, a Jupyter notebook, and some best practices can radically shorten the development and debugging cycle when building machine learning models on Metaflow.
toc: true
toc_sticky: true
author_profile: false
sidebar:
  nav: metaflow-sidebar
---

# Overview

This post addresses feature development and debugging of Metaflow jobs.
I aim to make the entire cycle as short, painless, and accurate as possible. 
Situations where the need to debug or develop a new feature might be:
1. my Metaflow run failed and I need to figure out why and fix it
1. I want to make a Metaflow based model better by adding new features or improving the model
1. I simply want to inspect artifacts of successful Metaflow runs to study them

Let's start with the setup.

# Setup

Here's a rundown of my recommended setup.
* Create or reuse a git repository.
* Make a directory structure with (see [Metaflow Best Practices for Machine Learning]({% post_url /blog/2021-05-26-metaflow-best-practices-for-ml %}#develop-a-separate-python-package) for more specifics on directory structures)
  * a subdirectory for Metaflow flows and local common code
  * and a pip-installable Python package.
* Use feature branches and pull requests to make changes.
* Write unit tests.
* Set up continuous integration and have it run the unit tests.
* Fire up an IDE to edit code in a feature branch (I like PyCharm).
* Fire up Jupyter Lab to load Metaflow data and object artifacts, and use [autoreload magic](https://ipython.org/ipython-doc/3/config/extensions/autoreload.html) to test source code edits that I'm actively making in my IDE against those artifacts.

I use this setup when developing examples in
[https://github.com/fwhigh/metaflow-helper](https://github.com/fwhigh/metaflow-helper).
I'll be referring to those components quite a bit. 
The local Python package there is called metaflow-helper and is used by doing, for example,
`from metaflow_helper.utils import install_dependencies` at the top of flows.
The flows live in multiple subdiretories of `examples/`, like `examples/model-selection/`.

# Pre Adoption

In the early stages of a project,
prior to first adoption by my potential users,
I do most of my prototyping in Jupyter and then slowly begin
to copy-paste working functions and classes into 
Metaflow steps (train.py, predict.py in examples/model-selection/),
my local common Python script (common.py in examples/model-selection/),
and into my local Python package (metaflow-helper at the top level).

The basic structure of the notebook looks like this.
{% gist c6f9c88cf94cedf2e96d6900ac0f1226 debug.ipynb %}

# Accessing Metaflow Artifacts

Metaflow already provides simple artifact access patterns 
like 

```python
from metaflow import Metaflow

print(Metaflow().flows)
``` 
and 

```python
from metaflow import Step

data = list(Step(f'Train/1234/some_step'))[0].data`
```

There's nothing else I'll need on the core Metaflow side.

# Editing and Testing My Own Code

But to make and test edits on my own code I've got autoreload 2 enabled.
In my notebook I'll `import common` at the top
and in a later cell use a fuction from common.py like

```python
result = common.some_function()
```

I can now make changes to the source code of `some_function` directly in common.py
and see those changes reflected immediately.
I don't have to re-import common, I can just
re-execute the cell that calls the function.

The same is true for my local package called metaflow-helper, which I installed using
`pip install -e .` at the top level of the repository. 
That `-e` means "editable mode". 
I can `from metaflow_helper.feature_engineer import FeatureEngineer`
and in later cells instantiate FeatureEngineer.
When I make changes to member functions of FeatureEngineer,
they will also immediately be reflected at the notebook cell level without
having to reimport metaflow-helper.

# Putting It All Together

The really killer thing is now I can access Metaflow artifacts
from successful or even failed Metaflow runs
and feed them into common or metaflow-helper functions in the notebook while
making on-the-fly changes to the code. 
At the cell level I can run and rerun 
to debug code edits.

Once I'm happy with the result I can git-commit and -push and issue a pull request.
Fix any failed unit test, get code reviewer approval, merge to the the target branch
and I'm ready to go with the changes.

# Post Adoption

Once I've done this Jupyter-to-source cycle enough times my source code becomes
larger and more battle-hardened.
If at some stage I've also achieved buy-in and adoption from my users,
I've got production-worthy flow and code.

Runs still fail at these mature stages, and I'll still need to debug.
I can still use the Jupyter notebook debugging pattern from the earlier stages,
but I'll be skewing much more heavily to iterative changes to my 
production code rather than prototyping from scratch in Jupyter and pushing to source code
scripts and packages.



