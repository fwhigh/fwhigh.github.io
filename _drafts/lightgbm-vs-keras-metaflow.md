---
title: "LightGBM vs Keras Model Selection At Scale Using Metaflow"
date: 2021-05-24 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - machine learning
  - engineering
tags:
  - metaflow
  - lightgbm
  - keras
excerpt: Demo of configurable, parallel LightGBM vs Keras model selection using Metaflow, including hyperparameter tuning, cross-validation, and early stopping. 
toc: true
toc_sticky: true
classes: wide
---

<!-- 
NOTES

examples/model-selection/results/1621113810652298
- 10_000 samples, noise 10, 1 category
examples/model-selection/results/1621115192490006
- 10_000 samples, noise 100, 1 category
examples/model-selection/results/1621116705680653
- 10_000 samples, noise 10, 2 categories
examples/model-selection/results/1621117548416977
- 10_000 samples, noise 100, 2 categories
-->

# Overview

I'm proud to be one of the earliest users of 
[Metaflow](https://metaflow.org/)
when I was at Netflix, 
months after creation and well before it was open sourced. 
I had the privilege of working alongside 
its creators and a lot of talented developers who
built some spectacular ML based applications 
with Metaflow. 
Now that I've left Netflix I look forward to continuing to use it, and
helping others get the most out of it. 

This post demonstrates one of the ways I like to use it: 
machine learning model selection at scale. 
I'll compare 7 total different hyperparameter settings for
LightGBM and Keras regressors,
each with 5 fold cross validation and early stopping,
trained and scored on a mock data set.
All 35 of these instances are executed in parallel.
The following box plots show the min and max
and the 25th, 50th (median), and 75th percentiles
of r-squared score.

<figure class="align-center" style="display: table;">
    <a href="/assets/lightgbm-vs-keras-metaflow/1621115192490006/all-scores.png"><img width="100%" src="/assets/lightgbm-vs-keras-metaflow/1621115192490006/all-scores.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Noisy regression, one category: any of the tested Keras architectures wins on r-squared score.</figcaption>
</figure>
<figure class="align-center" style="display: table;">
    <a href="/assets/lightgbm-vs-keras-metaflow/1621117548416977/all-scores.png"><img width="100%" src="/assets/lightgbm-vs-keras-metaflow/1621117548416977/all-scores.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Noisy regression, two categories: LightGBM with interactions wins on r-squared score.</figcaption>
</figure>

Predictions from the best model settings
on the held out test set look like this 
for the noisy one-category data set.

<figure class="align-center" style="display: table; ">
    <a href="/assets/lightgbm-vs-keras-metaflow/1621115192490006/predicted-vs-true.png"><img width="100%" src="/assets/lightgbm-vs-keras-metaflow/1621115192490006/predicted-vs-true.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Predicted versus true for the noisy regression, one category.</figcaption>
</figure>

For just 2 models each on a hyperparameter grid of size 10 to 100,
and using 5 fold cross validation, 
cardinality can reach between about 100 and 1000 jobs. 
Running Metaflow in the cloud (e.g. AWS) 
lets you execute each one of them
concurrently in isolated containers.
I've seen the cardinality blow up yet another order of magnitude or two
and things still work just fine, 
as long as you've got the time,
your settings are reasonable, 
and your account with your cloud provider is big enough.

The code is available at
[https://github.com/fwhigh/metaflow-helper](https://github.com/fwhigh/metaflow-helper).
Comments, issues, and pull requests are welcome.

This post is *not* meant to conclude whether 
LightGBM is better than Keras or vice versa -- 
I chose them for illustration purposes only.
What model to choose, and which will win a tournament, 
are application-dependent. 
And that's sort of the point!
This procedure outlines how you would productionalize model tournaments
that you can run on many different data sets, 
and repeat the tournament over time as well.

# Quickstart

You can run the model selection tournament immediately like this. 
Install a convenience package called metaflow-helper.

{% include gist_embed.html data_gist_id="fwhigh/6dba6e364210b79b8da126f9019dde41" data_gist_file="model_selection_quickstart_install.sh" %}

Then run the Metaflow tournament job. 
This one needs a few more packages, including Metaflow itself, 
which metaflow-helper doesn't currently require.

{% include gist_embed.html data_gist_id="fwhigh/6dba6e364210b79b8da126f9019dde41" data_gist_file="model_selection_quickstart_train_run.sh" %}

Results are printed to the screen, 
but they are also summarized in a local file `results/<run-id>/summary.txt`
along with some plots. 

This is the flow you are running. 
The mock data is generated in the start step.
The next step splits across all hyperparameter grid points
for all contenders -- 7 total for 2 models in the case of this example.
Then there are 5 tasks for each of 5 cross validation folds, 
for a total of 35 tasks.
Models are trained in these tasks directly.
The next step joins the folds and summarizes the results by
model and hyperparameter grid point.
Then there's a join over all models and grid points, 
whereupon a final model with a held out test set is trained and evaluated/
Finally a model on all of the data is trained.
The end step produces summary data and figures.

<figure class="align-center" style="display: table; ">
    <a href="/assets/lightgbm-vs-keras-metaflow/model-selection-flow.png"><img width="100%" src="/assets/lightgbm-vs-keras-metaflow/model-selection-flow.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Model selection flow.</figcaption>
</figure>


# Key Features and Functionality

## Mocking A Data Set

The mock regression data is generated using 
[sklearn.datasets.make_regression](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.make_regression.html).
Keyword parameter settings are controlled entirely in
[examples/model-selection/config.py](https://github.com/fwhigh/metaflow-helper/blob/main/examples/model-selection/config.py) 
in an object called 
`make_regression_init_kwargs`.
If you set `n_categorical_features = 1` you'll get a single data set with
`n_numeric_features` continuous features, 
`n_informative_numeric_features` of which are "informative" to the target `y`,
with noise given by `noise`,
through the relationship `y = beta * X + noise`.
`beta` are the coefficients,
`n_numeric_features - n_informative_numeric_features` 
of which will be zero. 
You can add any other parameters 
`make_regression` accepts directly to `make_regression_init_kwargs`.

If you set `n_categorical_features = 2` or more, you'll get
`n_categorical_features` independent regression sets concatenated together 
into a single data set. 
Each category corresponds to a totally independent set of coefficients. 
Which features are uninformative for each of the categories
is entirely random. 
This is a silly construction but it allows for validation of the flow against 
at least one categorical variable.

## Specifying Contenders

All ML model contenders, including their hyperparameter grids, 
are specified in 
[examples/model-selection/config.py](https://github.com/fwhigh/metaflow-helper/blob/main/examples/model-selection/config.py) 
with the `contenders_spec` object. 
Implement this spec object like you would any hyperparameter grid that you would
pass to 
[GridSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html) 
or
[ParameterGrid](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.ParameterGrid.html). 

Here's an illustration of tuning two models.
The LightGBM model is being tuned over 3 different `max_depth` settings.
The Keras model is being tuned over 4 different layer architectures:
* no hidden layers
* one hidden layer of size 15
* two hidden layers each of size 15
* one wide hidden layer of size 225

{% include gist_embed.html data_gist_id="fwhigh/6dba6e364210b79b8da126f9019dde41" data_gist_file="model_selection_contenders_spec.py" %}

The model is specified in a reserved key, `'__model'`.
The value of `'__model'` is a fully qualified Python object path string.
In this case I'm using metaflow-helper convenience objects I'm calling model helpers,
which reimplement init, fit, and predict 
with a small number of required keyword arguments. 

Anything prepended with `'__init_kwargs__model'` gets passed to the model initializers
and `'__fit_kwargs__model'` keys get passed to the fitters.
I'm 
wrapping the model in a Scikit-learn [Pipeline](sklearn.pipeline.Pipeline)
with step-name `'model'`.

I implemented two model handlers, 
a LightGBM regressor
and a Keras regressor.
Sources for these are in 
[metaflow_helper/model_helpers](https://github.com/fwhigh/metaflow-helper/tree/main/metaflow_helper/model_handlers).
They're straightforward, and you can implement additional ones
for any other algo. 

# Further Ideas and Extensions

There are a number of ways to extend this idea.

**Idea 1:** 
It was interesting to do model selection on a continuous target variable,
but it's possible to do the same type of optimization for a classification task using
[sklearn.datasets.make_classification](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.make_classification.html)
and
[sklearn.datasets.make_multilabel_classification](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.make_multilabel_classification.html)
to mock data.

**Idea 2:** 
You can add more model handlers 
for ever larger model selection searches. 

**Idea 3:** 
It'd be especially interesting to try to use 
*all* models in the grid in an ensemble,
which is definitely also possible with Metaflow
by joining each model from parallel grid tasks
and applying another model of models.

**Idea 4:** 
Random grid search performs better than searching on evenly spaced grid points.
I don't know off the top of my head
how complicated it would be to support 
randomized hyperparameter search in the `contender_spec`,
one issue being that the values have to be random number generators 
with args
rather than lists of scalars and iterables. 

**Idea 5:**
I do wish I could simply access each task in Scikit-learn's cross-validation search
(e.g. GridSearchCV)
tasks and distribute those directly into Metaflow steps. 
Then I could recycle all of its Pipeline and CV search machinery and patterns, 
which I like.
I poked around the Scikit-learn source code just a bit 
but it didn't seem straightforward to implement things this way. 
I had to break some Scikit-learn patterns 
to make things work but it wasn't too painful.

I'm interested in hearing your ideas, too. 
Are you using Metaflow? How do you use it?
