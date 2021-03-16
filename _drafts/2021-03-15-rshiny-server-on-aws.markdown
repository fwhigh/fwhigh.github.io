---
title: "Stand Up an R Shiny Server on AWS"
date: 2021-03-15 00:00:00 -0700
comments: true
categories: 
  - engineering
tags:
  - aws
  - r
  - docker
  - elasticbeanstalk
excerpt: How to stand up an R Shiny servery using AWS.
---

{% include toc %}

# Glossary

I'll be using a ton of abbreviations below for brevity. Here's what they mean. 

* AWS: Amazon Web Services
* EB: Elastic Beanstalk, an AWS service that serves web applications like web sites and REST APIs
* ECR: Elastic Container Registry, an AWS service that hosts Docker images
* CLI: Command line interface

# Quickstart

Requirements:

* [Docker](https://www.docker.com/)
* An [AWS](https://aws.amazon.com/) account
* The AWS CLI

Steps:

1. Install the free version of [Docker](https://www.docker.com/)
1. Create an AWS account
1. On a Mac: install homebrew then `brew install awscli`
1. Create Dockerfile.base that just inherits `FROM rocker/shiny` on [Dockerhub](https://hub.docker.com/r/rocker/shiny)
1. Create an [ECR repo](https://aws.amazon.com/ecr/) called rshiny-base
1. 

You should end up with a directory structure like this.

```
apps/
|── hello-world/
    |-- server.R
    |-- ui.R
Dockerfile
Dockerfile.base
index.html
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
Take note here of your ECR ID, which I will call `$ecr_id` below. 
It's a bunch of numbers.

## Create a sample Shiny app 

Create a directory structure like this:

```
apps/
|── hello-world/
    |-- server.R
    |-- ui.R
```

Insert a simple exmaple from, say, the [Shiny gallery](https://shiny.rstudio.com/gallery/#demos)
into `server.R` and `ui.R`.

## Create a base Dockerfile

Now make a file called `Dockerfile.base` that only has one line:

```
FROM rocker/shiny
``` 

Now build it and push to your new ECR repo.

```bash
# region="us-west-1"
# ecr_id=123456
aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin ${ecr_id}.dkr.ecr.${region}.amazonaws.comdocker build -t rshiny-base Dockerfile.base
docker tag rshiny-base:latest ${ecr_id}.dkr.ecr.${region}.amazonaws.com/rshiny-base:latest
docker push ${ecr_id}.dkr.ecr.${region}.amazonaws.com/rshiny-base:latest
```

## Create a new Dockerfile

Now create a file called `Dockerfile` with the contents

```
FROM rocker/shiny

COPY apps /srv/shiny-server

EXPOSE 3838
CMD ["/usr/bin/shiny-server.sh"]
```

You can try to build it and run it

```bash
docker build -t rshiny-apps .
docker run --rm -p 3838:3838 rshiny-apps
```

Now open http://127.0.0.1:3838/.


