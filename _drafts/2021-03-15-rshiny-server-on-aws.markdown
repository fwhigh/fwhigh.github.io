---
title: "Stand Up an R Shiny Server on AWS"
date: 2021-03-15 00:00:00 -0700
comments: true
categories: 
  - Engineering
excerpt: How I tricked AWS into serving R Shiny using rocker and Elastic Beanstalk.
---

{% include toc %}

# Overview

This is a fast way to stand up a Shiny server in the cloud with just about 200 total lines of code,
including the example app, thanks to [rocker](https://www.rocker-project.org/)'s Shiny image
and the maturity of cloud services. 
The time consuming parts are Docker image data transfer, server start overheads, and of course
any software installation and account signups that you need. 

It's a "Hello, World!" procedure and more work would need to be done to integrate it properly into a CI/CD
pipeline, but existing AWS services make that a pretty straightforward and pleasant experience. 

## Glossary

I'll be using some abbreviations below for brevity. Here's what they stand for. 

* AWS: Amazon Web Services
* EB: Elastic Beanstalk, an AWS service that serves web applications like web sites and REST APIs
* ECR: Elastic Container Registry, an AWS service that hosts Docker images
* CLI: Command line interface

## Requirements

* [Docker](https://www.docker.com/)
* An [AWS](https://aws.amazon.com/) account
* The AWS CLI and the AWS Elastic Beanstalk CLI

## Quickstart

1. Install the free version of [Docker](https://www.docker.com/)
1. Create an AWS account
1. On a Mac: install homebrew then `brew install awscli`
1. Create Dockerfile.base that just inherits `FROM rocker/shiny` on [Dockerhub](https://hub.docker.com/r/rocker/shiny)
1. Create an [ECR repo](https://aws.amazon.com/ecr/) called `rshiny-base`
1. Build and push the `Dockerfile.base` image to `rshiny-base`
1. Build the `Dockerfile`
1. 

You should end up with a directory structure like this.

```
apps/
|-- index.html  # optional
|── hello-world/
    |-- server.R
    |-- ui.R
Dockerfile
Dockerfile.base
```

Note that I initially tried cloning rocker/shiny, creating a new AWS EB Docker application, 
and deploying the Dockerfile as-is but the build timed out in AWS EB. My solution is to build the image
locally and push that to AWS ECR, then have AWS EB pull the image directly from AWS ECR. 

# Full procedure

## Docker and AWS preliminaries

Install the free version of [Docker](https://www.docker.com/).

Create an AWS account. Take note of your default region. Mine is `us-west-1`. 
Let's call it `$region`.

Install the AWS CLI. I like to use homebrew a la `brew install awscli`.

Create an [ECR repo](https://aws.amazon.com/ecr/) called `rshiny-base`. 
Take note here of your AWS account ID, which I will call `$aws_account_id` below. 
It's a bunch of numbers.

## Create a base Dockerfile

Now make a file called `Dockerfile.base` that only has one line:

```
FROM rocker/shiny
``` 

Now build it and push to your new ECR repo. 


Now build the base image and call it `rshiny-base`.

```bash
docker build -t rshiny-base -f Dockerfile.base .
```

Then push the image to ECR.
Follow the instructions on ECR web site to on how to authenticate and push, but here's what it looked like
at the time of writing.

```bash
# region="us-west-1"
# aws_account_id=123456789
aws ecr get-login-password --region $region | docker login --username AWS --password-stdin ${aws_account_id}.dkr.ecr.${region}.amazonaws.comdocker build -t rshiny-base Dockerfile.base
docker tag rshiny-base:latest ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/rshiny-base:latest
docker push ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/rshiny-base:latest
```

The upload took a while for me. If I had to do this repeatedly I would set up an automated job to build and push using AWS Batch or,
much more likely, as part of a CI/CD pipeline using AWS CodePipeline. 

## Create a Shiny app

Make a directory called `apps/` and put a simple working app spec there, 
copied from, say, the [Shiny gallery](https://shiny.rstudio.com/gallery/#demos).
Here's my final directory structure

```
apps/
|── hello-world/
    |-- server.R
    |-- ui.R
```

## Create a new Dockerfile for your app server

Now create a file called `Dockerfile` with the following contents.

```
FROM <aws_account_id>.dkr.ecr.<region>.amazonaws.com/rshiny-base

COPY apps /srv/shiny-server

EXPOSE 3838
CMD ["/usr/bin/shiny-server.sh"]
```

You can try to build it and run it

```bash
docker build -t rshiny-apps .
docker run --rm -p 3838:3838 rshiny-apps
```

Now open http://127.0.0.1:3838/. You should see a message letting you know the server is running properly. 
You can create and edit a home page at the apps root, `apps/index.html`. 

## Push it to Elastic Beanstalk


I installed the AWS EB CLI using `brew install awsebcli`. I debated whether to try to create the server using the
CLI or the web UI, and decided to try the CLI first. I've done a lot of clicking in the UI in past work 
so I wanted to try something new. 

Here's what I did to create an application called `rshiny`. 

```bash
eb init
eb create rshiny
```







