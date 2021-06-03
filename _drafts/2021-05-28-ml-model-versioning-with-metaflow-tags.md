---
title: "Versioning Machine Learning Models with Metaflow Tags"
date: 2021-05-28 12:00:00 -0700
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
excerpt: Metaflow tags provide a remarkably simple and powerful way to version ML models.
toc: true
toc_sticky: true
author_profile: false
sidebar:
  nav: metaflow-sidebar
---

# Intro

Metaflow tags provide a remarkably simple and powerful way to version ML models.
I like to think of Metaflow run tags for data 
almost the way I think about git commit tags for code (key word [almost](#gotcha)).
You can tag any run for any purpose -- it's just a string you apply to a run,
which you can later use to filter runs.

Here are two functions that retrieve runs. 
The first function just gets the latest successful run of a flow called GenerateData.
The second function gets the last successful run of a flow called Train
after filtering by a user-supply list of tags.
These are the functions I'll use in the example in the post.
I'll be filtering by `tags=['prod:v1']` or `tags=['test:v1']` 
to demonstrate model versioning.

{% include gist_embed.html data_gist_id="fwhigh/3d4eecad23dc77b745016e9e987c4e58" data_gist_file="filter_runs.py" %}

You can reserve specific tags or tag patterns for specific purposes, 
like production releases of Metaflow data artifacts, 
exactly like you would tag a git commit with something like "v0.5.2" 
for a software package artifact release.

The use-case for this is a fairly routine ML model in a production setting.
Your organization needs fresh predictions reliably.
How fresh and how reliably are subject to implicit or explicit
[SLA](https://en.wikipedia.org/wiki/Service-level_agreement).

In cases like these you, the owner of the ML part of the system,
might train a model and persist it using Metaflow train.py,
then load that model on a schedule that matches that time frame from the SLA.

You can start with the basic practices I talked about in my other post on
[Metaflow Best Practices for Machine Learning]({% post_url 2021-05-25-metaflow-best-practices-for-ml %}).
This is how I structured the example for this post at
[examples/model-versioning](https://github.com/fwhigh/metaflow-helper/tree/main/examples/model-versioning),
but with some interesting variation.
I've got
* **generate_data.py** creates mock train and predict data in a separate step,
* **train.py** engineers features and trains the model,
* **predict.py** makes predictions on a production model against new data,
* **common.py** holds common code across all these flows,
* **config.py** holds common configuration values,
* **test_config.py** is for small scale test-mode runs,
* and a separate, pip-installable Python package called [**metaflow-helper**](https://github.com/fwhigh/metaflow-helper/) holds some of the more reusable common code across multiple of my examples.

Let's just dive in with a small scale example to get a sense for it.
All of the code is available at
[https://github.com/fwhigh/metaflow-helper](https://github.com/fwhigh/metaflow-helper).

# Demo

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

Beware, you can tag
multiple runs with the same tag. 
Usually at prediction time you'd want to get the latest successful
train run filtered by a given tag. 
But it's also possible to simply tag a run accidentally 
and you won't have a way to fix it and roll back at this time.
Thankfully there's 
[a ticket out](https://github.com/Netflix/metaflow/issues/159) 
to add tag editing capabilities to Metaflow.
Once that happens we'll have a way to promote models from test to prod without having to
rerun -- this will be a huge time and resources saver. Stay tuned.
