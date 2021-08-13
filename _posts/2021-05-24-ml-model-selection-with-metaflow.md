---
title: "Machine Learning Model Selection with Metaflow"
date: 2021-05-24 12:00:00 -0700
last_modified_at: 2021-08-12 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/chris-ried-ieic5Tq8YMk-unsplash.jpg
categories:
  - Machine Learning
  - Engineering
  - Featured
  - Metaflow
excerpt: Configurable, repeatable, parallel model selection using Metaflow, including randomized hyperparameter tuning, cross-validation, and early stopping.
toc: true
toc_sticky: true
author_profile: false
sidebar:
  nav: metaflow-sidebar
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
examples/model-selection/results/1621302832648370
- 10_000 samples, noise 100, 1 categories, randomized search
examples/model-selection/results/1621303262250162
- 10_000 samples, noise 100, 2 categories, randomized search
-->

* [AICamp recipe Github gist](https://gist.github.com/fwhigh/c6f9c88cf94cedf2e96d6900ac0f1226#file-aicamp_recipe-sh)
* [fwhigh/metaflow-helper Github project](https://github.com/fwhigh/metaflow-helper)
* [AICamp talk on YouTube](https://www.youtube.com/watch?v=kDIBTWDJGgY)

# Overview

I worked with the
[Metaflow](https://metaflow.org/)
creators at Netflix from
the time they built their first proofs of concept.
About six months later I built my first flows as
one of the earliest adopters.
I had been rolling my own Flask API services
to serve machine learning model predictions
but Metaflow provided a much more accessible, lower
complexity path to keep the models and services up to date.

I also had the privilege of working next to
a lot of other talented developers who
built some of their own spectacular ML based applications
with Metaflow over the following years.
Now that I've left Netflix I look forward to continuing to use it
and helping others get the most out of it.

What is Metaflow?
It's a framework that lets you write data pipelines
in pure Python, and it's particularly suited to scaling up machine learning applications.
Pipelines are specified as multiple *steps* in a *flow*,
and steps can consist of potentially many *tasks* executed
in parallel in their own isolated containers in the cloud.
Tasks are stateless and reproducible. Metaflow persists objects and data in
a data store like S3 for easy retrieval, inspection, and further processing
by downstream systems.
Read more at [https://metaflow.org/](https://metaflow.org/).

In this post I'll demonstrate one of the ways I like to use it:
doing repeatable machine learning model selection at scale.
(This post does not address the ML model reproducibility crisis.
Repeatable here means easily re-runnable.)
I'll compare 5 different hyperparameter settings for each of
LightGBM and Keras regressors,
with 5 fold cross validation and early stopping,
for a total of 50 parallel model candidates.
All of these instances are executed in parallel.
The following box plots show the min and max
and the 25th, 50th (median), and 75th percentiles
of r-squared score from a mock regression data set.

<figure class="align-center" style="display: table;">
    <a href="/assets/ml-model-selection-with-metaflow/1621302832648370/all-scores.png"><img width="100%" src="/assets/ml-model-selection-with-metaflow/1621302832648370/all-scores.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Noisy regression, one category: any of the tested Keras architectures wins on out-of-sample r-squared score.
    The narrow single-hidden-layer Keras model happened to be best overall, with l1 factor 2.4e-7 and l2 factor 7.2e-6.</figcaption>
</figure>
<figure class="align-center" style="display: table;">
    <a href="/assets/ml-model-selection-with-metaflow/1621303262250162/all-scores.png"><img width="100%" src="/assets/ml-model-selection-with-metaflow/1621303262250162/all-scores.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Noisy regression, two categories: LightGBM with depth 3 interactions and learning rate 0.03 wins on out-of-sample r-squared score.
    The LightGBM model with depth 1 performed the worst.</figcaption>
</figure>

Predictions from the best model settings
on the held out test set look like this
for the noisy one-category data set.

<figure class="align-center" style="display: table; ">
    <a href="/assets/ml-model-selection-with-metaflow/1621302832648370/predicted-vs-true.png"><img width="100%" src="/assets/ml-model-selection-with-metaflow/1621302832648370/predicted-vs-true.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Predicted versus true for the noisy regression, one category.</figcaption>
</figure>

For just 2 models each on a hyperparameter grid of size 10 to 100,
and using 5 fold cross validation,
cardinality can reach between of order 100 to 1000 jobs.
It's easy to imagine making that even bigger with more models
or hyperparameter combinations.
Running Metaflow in the cloud (e.g. AWS)
lets you execute each one of them
concurrently in isolated containers.
I've seen the cardinality blow up to of order 10,000 or more
and things still work just fine,
as long as you've got the time,
your settings are reasonable,
and your account with your cloud provider is big enough.
With the

The code is available at
[https://github.com/fwhigh/metaflow-helper](https://github.com/fwhigh/metaflow-helper).
The examples in this article are reproducible from the commit tagged **v0.0.1**.
You can also install the tagged package from PyPI with
`pip install metaflow-helper==0.0.1`.
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
Install a convenience package called metaflow-helper
at the commit tagged v0.0.1.

{% include gist_embed.html data_gist_id="fwhigh/c6f9c88cf94cedf2e96d6900ac0f1226" data_gist_file="model_selection_quickstart_install.sh" %}

Then run the Metaflow tournament job at a small scale just to test it out.
This one needs a few more packages, including Metaflow itself,
which metaflow-helper doesn't currently require.

{% include gist_embed.html data_gist_id="fwhigh/c6f9c88cf94cedf2e96d6900ac0f1226" data_gist_file="model_selection_quickstart_train_run.sh" %}

Results are printed to the screen,
but they are also summarized in a local file `results/<run-id>/summary.txt`
along with some plots.
There are full scale model selection configurations available in there as well.

Following figure shows the flow you are running.
The mock data is generated in the start step.
The next step splits across all hyperparameter grid points
for all contenders -- 10 total for 2 models in the case of this example.
Then there are 5 tasks for each cross validation fold,
for a total of 50 tasks.
Models are trained in these tasks directly.
The next step joins the folds and summarizes the results by
model and hyperparameter grid point.
Then there's a join over all models and grid points,
whereupon a final model with a held out test set is trained and evaluated/
Finally a model on all of the data is trained.
The end step produces summary data and figures.

<figure class="align-center" style="display: table; ">
    <a href="/assets/ml-model-selection-with-metaflow/model-selection-flow.png"><img width="100%" src="/assets/ml-model-selection-with-metaflow/model-selection-flow.png" /></a>
    <figcaption style="display: table-caption; caption-side: bottom; font-style: italic;" width="100%">Model selection flow.</figcaption>
</figure>

# Mocking A Data Set

The mock regression data is generated using
Scikit-learn
[make_regression](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.make_regression.html).
Keyword parameter settings are controlled entirely in configuration files like
[randomized_config.py](https://github.com/fwhigh/metaflow-helper/blob/v0.0.1/examples/model-selection/randomized_config.py)
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

# Specifying Contenders

All ML model contenders, including their hyperparameter grids,
are also specified in
[randomized_config.py](https://github.com/fwhigh/metaflow-helper/blob/v0.0.1/examples/model-selection/randomized_config.py)
using the `contenders_spec` object.
Implement this spec object like you would any hyperparameter grid that you would
pass to Scikit-learn
[GridSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html)
or
[RandomizedSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.RandomizedSearchCV.html),
or equivalently
[ParameterGrid](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.ParameterGrid.html)
or
[ParameterSampler](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.ParameterSampler.html).
Randomized search is automaticallly used if the `'__n_iter'` key is present in the contender spec,
otherwise the flow will fall back to grid search.

Here's an illustration of tuning two models.
The LightGBM model is being tuned over 5 random `max_depth` and `learning_rate` settings.
The Keras model is being tuned over 5 different combinations of layer architectures
and regularizers. The layer architectures are
* no hidden layers,
* one hidden layer of size 15,
* two hidden layers each of size 15, and
* one wide hidden layer of size 225.
The regularizers are l1 and l2 factors, log-uniformly sampled
and applied globally to all biases, kernels, and activations.
This specific example may well be a naive search,
but the main purpose right now is to demonstrate what is possible.
The spec can be extended arbitrarily for real-world applications.

{% include gist_embed.html data_gist_id="fwhigh/c6f9c88cf94cedf2e96d6900ac0f1226" data_gist_file="model_selection_contenders_spec.py" %}

The model is specified in a reserved key, `'__model'`.
The value of `'__model'` is a fully qualified Python object path string.
In this case I'm using metaflow-helper convenience objects I'm calling model helpers,
which reimplement init, fit, and predict
with a small number of required keyword arguments.

Anything prepended with `'__init_kwargs__model'` gets passed to the model initializers
and `'__fit_kwargs__model'` keys get passed to the fitters.
I'm
wrapping the model in a Scikit-learn
[Pipeline](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html)
with step-name `'model'`.

I implemented two model wrappers,
a LightGBM regressor
and a Keras regressor.
Sources for these are in
[metaflow_helper/models](https://github.com/fwhigh/metaflow-helper/tree/v0.0.1/metaflow_helper/models).
They're straightforward, and you can implement additional ones
for any other algo.

# Further Ideas and Extensions

There are a number of ways to extend this idea.

**Idea 1:**
It was interesting to do model selection on a continuous target variable,
but it's possible to do the same type of optimization for a classification task using
Scikit-learn
[make_classification](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.make_classification.html)
and
[make_multilabel_classification](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.make_multilabel_classification.html)
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
I do wish I could simply access each task in Scikit-learn's cross-validation search
(e.g. GridSearchCV)
tasks and distribute those directly into Metaflow steps.
Then I could recycle all of its Pipeline and CV search machinery and patterns,
which I like.
I poked around the Scikit-learn source code just a bit
but it didn't seem straightforward to implement things this way.
I had to break some Scikit-learn patterns
to make things work but it wasn't too painful.

I'm interested in any other ideas you might have. Enjoy!
