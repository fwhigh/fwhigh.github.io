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

CHANGE YOU/WE TO I

Situations where you'll want to develop new code are:
1. you're debugging a failed Metaflow run
1. you're inspecting artifacts from a successful Metaflow run
1. you're enhancing a Metaflow flow with, say, new features for your model or new modeling choices

# High Level Sketch

Here's a rundown of my recommended setup.
* Create or reuse a git repository
* Make a directory structure with
  * a directory for Metaflow flows and local common code
  * a pip-installable Python package
  * I've written more directory structures in [Metaflow Best Practices for Machine Learning]({% post_url 2021-05-26-metaflow-best-practices-for-ml %}#develop-a-separate-python-package).
* Use feature branches and pull requests to make changes
* Write unit tests 
* Set up continuous integration and have it run the unit tests
* Fire up an IDE to edit code in a feature branch (I like PyCharm)
* Fire up Jupyter Lab to load Metaflow data and object artifacts, and use [autoreload magic](https://ipython.org/ipython-doc/3/config/extensions/autoreload.html) to test source code edits that you're actively making in your IDE against those artifacts

I use this setup when developing examples in
[https://github.com/fwhigh/metaflow-helper](https://github.com/fwhigh/metaflow-helper).
I'll be referring to those components quite a bit. 
The local Python package there is called metaflow-helper and is used by doing, for example,
`from metaflow_helper.utils import install_dependencies`.
The worklows live in multiple subdiretories of examples/, like examples/model-selection/.

# Pre Adoption

In the early stages of a project,
prior to first adoption,
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

There's nothing else you'll need on the core Metaflow side.

# Editing and Testing Your Own Code

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
I can now make local changes to the metaflow-helper package source code,
and they will also immediately be reflected at the function level without
having to reimport metaflow-helper. 
This is also true if the function I'm editing is a member of a class.

# Putting It All Together

The really killer thing is now I can access Metaflow artifacts
and feed them into common or metaflow-helper functions while
making on-the-fly changes to the code. 
At the cell level I can run and rerun 
to debug code edits.

So if I'm debugging a failed Metaflow run, I can access all of the 
successful upstream steps and even some of the upstream artifacts from
the failed step (!), and make changes to and test just the part that failed.

Once I'm happy with the result I can git-commit and -push and issue a pull request.
Fix any failed unit test, get code reviewer approval, merge to the the target branch
and you're ready to go with the changes.

# Post Adoption

Once you've done this Jupyter-to-source cycle enough times your source code becomes
larger and more battle-hardened over time.
If you've also achieved buy-in and adoption from your users,
you've got production-worthy flow and code.

Runs still fail at these mature stages, and you'll still need to debug.
You can still use the Jupyter notebook debugging pattern from the earlier stages,
but you'll be skewing much more heavily to iterative changes to your 
production code rather than prototyping from scratch in Jupyter and pushing to source code
scripts and packages.



