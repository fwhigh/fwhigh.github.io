---
title: "Versioning Machine Learning Models with Metaflow Tags"
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
excerpt: Metaflow tags give you a remarkably simple and powerful way to version ML models.
toc: true
toc_sticky: true
author_profile: false
sidebar:
  nav: metaflow-sidebar
---

# Intro

Metaflow tags give you a remarkably simple and powerful way to version ML models.
I like to think of Metaflow run tags for data the same way I think about git commit tags for code.
You can tag any run for any purpose.
You can also reserve specific tags or tag patterns for specific purposes, 
like production releases of Metaflow data artifacts, 
exactly like you would tag a git commit with something like "v0.5.2" 
for a software package artifact release.

The use-case is a fairly routine ML prediction model.
Your organization needs fresh predictions.
How fresh and how quickly are subject to implicit or explicit
[SLA](https://en.wikipedia.org/wiki/Service-level_agreement).

In cases like these you, the owner of the ML modeling part of the system,
might train a model and persist it using Metaflow train.py,
then load that model on a schedule that matches that time frame from the SLA.

You can start with the basic practices I talked about in my other post on
[Metaflow Best Practices for Machine Learning]({% post_url 2021-05-26-metaflow-best-practices-for-ml %}).
This is how I structured the example for this post at
[examples/model-versioning](https://github.com/fwhigh/metaflow-helper/tree/main/examples/model-versioning).
I've got
* generate_data.py to create mock train and predict data in a separate step,
* train.py for model training,
* predict.py for ongoing predictions on a production model against new data,
* common.py for common code across all the flows,
* config.py to hold common configuration values,
* test_config.py to do small scale test-mode runs,
* and a separate, pip-installable Python package called [metaflow-helper](https://github.com/fwhigh/metaflow-helper/) for some of the more reusable common code, the source of which happens to live in the same repo at the top level.

Let's just dive in with a small scale example to get a sense for it.
All of the code is available at
[https://github.com/fwhigh/metaflow-helper](https://github.com/fwhigh/metaflow-helper).

# Quickstart

First, get the code, create and activate a virtual environment,
and install the metaflow-helper package locally in editable model.

{% include gist_embed.html data_gist_id="fwhigh/c6f9c88cf94cedf2e96d6900ac0f1226" data_gist_file="model_selection_quickstart_install.sh" %}

Now generate a small mock data set 
and train two different small models a fraction of it. 
Tag one train run with "prod:v1" to emulate a production model
and tag another train run with "test:v1" to emulate a test model, 
maybe one that you're messing around with in a research sprint.
Then predict on each by asking for specific train run artifacts by tag.
Finally, validate that the train run artifacts match between the tagged train run and the
corresponding predict run.

{% include gist_embed.html data_gist_id="fwhigh/3d4eecad23dc77b745016e9e987c4e58" data_gist_file="train_predict_quickstart.sh" %}

You can use `--configuration config` or just remove that argument
if you're ready to train a full model on a bigger data set.

# Gotcha

There's one gotcha that I'm aware of, which is that you can tag
multiple runs with the same tag. 
Usually at prediction time you'd want to get the latest successful
train run filtered by a given tag. 
But it's also possible to simply tag a run accidentally 
and you won't have a way to fix it at this time.
Thankfully there's 
[a ticket out](https://github.com/Netflix/metaflow/issues/159) 
to add tag editing capabilities to Metaflow.
Once that happens we'll have a way to promote models from test to prod without having to
rerun. Stay tuned.
