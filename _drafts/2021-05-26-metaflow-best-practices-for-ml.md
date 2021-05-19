---
title: "Metaflow Best Practices for Machine Learning"
date: 2021-05-26 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - Machine Learning
  - Engineering
  - Featured
  - Metaflow
excerpt: I've compiled a list of best practices for using Metaflow for machine learning from my years of using it at Netflix.
toc: true
toc_sticky: true
author_profile: false
sidebar:
  nav: metaflow-sidebar
---

# Overview

Here's a list of practices and patterns 
tested over roughly four or so years of experience with Metaflow
that result in 
a short development loop;
reusable, maintainable, and reliable code;
and just an overall fun and rewarding developer experience. 

# Best practices for ML

## Minimal directory structure

I'll start with a suggested minimal directory structure.
This example has a training flow and a prediction flow,
with common code across the two 
plus a Jupyter notebook for debugging.

```
<your-repo>
├── flows
|  ├── train.py
|  ├── predict.py
|  ├── debug.ipynb
|  └── common.py
└── .gitignore
```

The training and prediction flow specs import common code and do... you know, machine learning.
They are separated out to support ongoing predictions on a model
that is potentially re-trained on a different schedule.
So you might run train.py quarterly,
and run predict.py weekly on fresh incoming data.

Check out this minimal flow example
and 

{% gist c6f9c88cf94cedf2e96d6900ac0f1226 train.py %}

Note `import common`.

## Put common code into a separate script

In the minimal code structure and flow example I've got a script called
common.py. That contains reusable functions, classes,
and variables that both train.py and predict.py can use. 
Pulling your code into a separate script makes it more easily 
reusable and testable 
and shortens your Metaflow steps and overall flow spec. 

## Git-ignore .metaflow

Don't forget to add `.metaflow` to your .gitignore 
because those directories contain the local data artifacts.

## Debug in Jupyter a notebook

You can debug common code and access Metaflow data artifacts
in Jupyter notebooks.
Here's a minimal example of all of that.

{% gist c6f9c88cf94cedf2e96d6900ac0f1226 debug.ipynb %}

I'm using autoreload magic so that I can make changes to common.py 
and have those changes immediately reflected at the cell level 
without having to re-import common or otherwise run a bunch of other cells.
Your working directory in the notebook should be 
`<your-repo>/flows/` in this case.

(Note: that debug snippet shows artifacts from my previous post,
{% comment %} 
<!-- TODO Uncomment the following after the model selection post goes live-->
[LightGBM vs Keras Model Selection At Scale Using Metaflow]({% post_url 2010-07-21-lightgbm-vs-keras-metaflow %}).)
{% endcomment %}

## Develop a separate Python package

When it makes sense to (and not earlier) 
I try to separate out my 
more broadly reusable code into a separate, pip-installable Python package.
You can put the code in the same repo, or break it out into another one.
Here's a minimal example of putting it in the same repo.

```
<your-repo>
├── flows
|  ├── train.py
|  ├── predict.py
|  ├── debug.ipynb
|  └── common.py
├── .gitignore
├── your_package
|  ├── __init__.py
|  └── models.py
└── setup.py
```

Adding the setup.py makes `your_package` pip installable.
I won't wade into suggesting what to put into setup.py, 
many others have already covered that.
During development I'll 
fire up a Python venv with `python -m venv venv && . venv/bin/activate`
and then install the package in editable mode
with `pip install -e .`,
all from the top level of the repo.

In each Metaflow step I'll pip install from git if the package is not already locally available.
Here are the functions that do that, which I put into common.py.

{% gist c6f9c88cf94cedf2e96d6900ac0f1226 common.py %}

Now at the top of each step you would do something like:

```python
common.install_dependencies(
	{'your_package': 'git+ssh://git@github.com/<github-username>/<your-repo>.git'}
)
```

This will try to `import your_package` (the dictionary key)
and if it fails, pip-install from Github.
Doing this will seem like nonsense during development, but when you 
deploy to a production environment this will become necessary.
Installing via pip lets you get athe code from Github or PyPI,
and will let you pin in both cases.
Here are some different ways to pin.

```python
# Install the latest commit from the default branch
{'your_package': 'git+ssh://git@github.com/<github-username>/<your-repo>.git'}
# Pin by installing a tagged commit
{'your_package': 'git+ssh://git@github.com/<github-username>/<your-repo>.git@v0.0.1'}
# Pin by installing a commit hash
{'your_package': 'git+ssh://git@github.com/<github-username>/<your-repo>.git@00db203'}
# Install from PyPI
{'your_package': 'your_package'}
# Pin by install from a PyPI version
{'your_package': 'your_package==0.0.1'}
# etc etc
```

You can call `install_dependencies` in your debugging Jupyter notebook, too.
If `your_package` is already available to it, nothing will happen. 
*This means you can test your Metaflow artifacts, common flow code,
and your external package code all in the same notebook.*

And speaking of pinning...

## Pin your packages

If you plan on running your flows on a cron schedule or against
triggers over long periods of time,
do yourself a favor and pin your packages.
This increases stability of repeated
flow runs that use artifacts from other flows that ran earlier.
For example, predict.py needs to load the model artifact persisted in train.py,
potentially days, weeks, or months later, depending on your design.

It's useful to think of your Metaflow jobs like you would any long-running
application, for instance a web app. 
Pin for reproducibility and to minimize maintenance over the long term.

You can take this thinking one step further with Metaflow:
*think of each Metaflow step as an independent, long-running application*
and pin potentially different packages at the top of every step. 
One example where I've seen this come up is in 
using Tensorflow. 
Tensorflow requires a specific version range of numpy, 
but otherwise I want access to a more recent numpy release elsewhere.
If I isolate my Tensorflow modeling code to a single step or set of steps,
and do pre- and post-processing in separate steps, I can pin Tensorflow with
a floating numpy version and in the other steps I'll in general get a different numpy version.
The `install_dependencies` function pattern I mentioned above 
in <a href="#develop-a-separate-python-package">Develop a separate Python package</a>
will let me do this.

## Keep flows and flow steps short

If you pull common code into adjacent Python scripts 
or into a separate package as suggested above,
you'll be in a good position to make your flow spec
and each of its steps as short as possible.

Keeping them short is useful for readability and maintainability.
You'll also invariably have to do
other high-level stuff at the step level without the option of 
pulling that code into common functions, 
for example 
[Metaflow step-level exception handling](https://docs.metaflow.org/metaflow/failures#catching-exceptions-with-the-catch-decorator). 
Do flow-control-level operations in steps 
and otherwise call just a few functions per step if you can.

Keeping the scope of steps small is also useful for
debugging different logical chunks of your pipeline without having to
rerun upstream code
and for
resuming execution after a failed run. 
Often times in production
you'll get failures due to platform failures,
and it's useful to have completed 
as much upstream processing successsfully as possible.
Then you can resume from the failed steps forward.
Or you'll get a runtime failure from an unhandled edge case.
It's helpful when upstream, smaller scope steps have
completed successfully and the runtime failure is isolated to a 
small step.
Small steps make the whole debugging and maintenance experience more enjoyable. 

## Use an IDE 

I prefer PyCharm. It plays nice with Metaflow.
Debugging seems to work fine, but it can be a bit tricky
to debug parallel tasks in foreach steps. 
Using test-mode 
(<a href="#implement-a-test-mode">Implement a test mode</a>)
and eliminating parallel tasks helps. 
Make sure to reuse your virtual environment interpreter if you set one up already.
I've also set PyCharm up for Metaflow development against a remote server --
that works pretty well, though there are a lot of configuration options to set.

VS Code works as well. It's faster but has reduced functionality.
I especially miss refactoring and fully-featured inspection when I
use VS Code. 
There's a time and a place for both and as always it boils down to personal preference.

I haven't tested other IDEs like Spyder. 
I'd like to hear if others have and what
the good and the bad are about each one.

## Implement a test mode

Implement a test mode that will run your flow as-is
but on as small a data set as possible and with hyperparameter settings
that make the ML training optimization as quick as possible.
I like to do this by creating
a flag parameter that I can use to 
subset the data and reduce parallelism to one concurrent task for any given step. 
If you're training a model,
reduce the number of maximum possible optimization iterations to
something small like 10 epochs.
Here's one way to create a test-mode using a Metaflow Parameter.

{% gist c6f9c88cf94cedf2e96d6900ac0f1226 test_mode.py %}

Now I can run the flow normally with

```bash
python train.py
```

or in test mode with

```bash
python train.py --test_mode 1
```

I did a variant of this in my 
[model selection example](https://github.com/fwhigh/metaflow-helper/tree/main/examples/model-selection)
from 
{% comment %} 
<!-- TODO Uncomment the following after the model selection post goes live-->
[LightGBM vs Keras Model Selection At Scale Using Metaflow]({% post_url 2010-07-21-lightgbm-vs-keras-metaflow %}).
{% endcomment %}
Instead of using a boolean flag I point to different configuration files by string,
some of which perform the same tasks of 
subsetting the data down and shortening the model training times dramatically.

## Run flows in test mode in a CI/CD pipeline

If you've got a nice and short test mode working you can run it as part of
continuous integration/continuous delivery & deployment. 
You'll see working examples of this in
[metaflow-helper](https://github.com/fwhigh/metaflow-helper).
I've got separate jobs and badges set up for unit testing and for running
the Metaflow examples in test mode.

# What did I miss?

I didn't talk about more advanced practices I like, 
such as 
[Metaflow run tagging](https://docs.metaflow.org/metaflow/tagging#tagging)
and setting up isolated test and development environments
that operate without affecting the production environment. 
I'll cover those in another post on more advanced best practices.

I'd like to hear from you on what I may have missed or how you do things differently!
