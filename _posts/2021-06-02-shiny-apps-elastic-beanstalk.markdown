---
title: "Deploy Custom Shiny Apps to AWS Elastic Beanstalk"
date: 2021-06-02 13:00:00 -0700
comments: true
categories: 
  - Engineering
excerpt: How I tricked AWS into serving R Shiny with my local custom applications using rocker and Elastic Beanstalk.
toc: true
toc_sticky: true
---

# Overview

This is a fast way to stand up a Shiny server in the cloud 
that serves your own set of custom Shiny apps
with very few lines of code,
including the example app, 
thanks to [rocker](https://www.rocker-project.org/)'s Shiny images
and AWS.
The time consuming parts are Docker image data transfer, server start overheads, and of course
any software installation and account signups that you need. 

Note that I could not get this to work when pulling rocker's image from
Dockerhub directly within Elastic Beanstalk. 
EB timed out. 
My solution, which appears to be pretty stable,
is to rebuild the rocker image locally,
push it to AWS's own Docker image repository called ECR,
and ask EB to pull that instead.
The idea is the in-region data transfer across AWS services 
should generally be faster.

You can also automate the rocker image build with AWS CI/CD services,
which I have done successfully in the past using CodePipelines.
But this post is just a "Hello, World!" and I'll leave that part to you.

# Glossary

* AWS: [Amazon Web Services](https://aws.amazon.com/)
* EB: [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/), an AWS service that serves web applications like web sites and REST APIs
* ECR: [Elastic Container Registry](https://aws.amazon.com/ecr/), an AWS service that hosts Docker images
* CLI: Command line interface

# Requirements

* [Docker Desktop](https://www.docker.com/)
* An [AWS](https://aws.amazon.com/) account
* The [AWS CLI](https://aws.amazon.com/cli/)] (`brew install awscli`)
* The [AWS Elastic Beanstalk CLI](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html) (`brew install awsebcli`)

# Quickstart

1. `mkdir new-shiny-app-repo && cd new-shiny-app-repo`
1. `mkdir apps` and then put a ["Hello, World!" Shiny app](https://shiny.rstudio.com/gallery/) in there
1. Create Dockerfile.base that just pulls `FROM rocker/shiny` on [Docker Hub](https://hub.docker.com/r/rocker/shiny) (or rocker/shiny-verse to also make the tidyverse available) and installs any additional R packages your apps need
1. Create an [ECR repo](https://aws.amazon.com/ecr/) called `rshiny-base` on ECR
1. Build the `rshiny-base` image locally from Dockerfile.base and push it to ECR
1. Create the `Dockerfile` specified below
1. On a Mac: install the EB CLI with `brew install awsebcli`
1. Git-commit your changes
1. `eb init shiny`
1. `eb create shiny`

You should end up with a directory structure like this.

```
new-shiny-app-repo
├── apps/
|  ├── index.html  # optional
|  └── hello-world/
|     ├── server.R
|     └── ui.R
├── Dockerfile
├── Dockerfile.base
└── .gitignore
```

# More Details

## Docker and AWS Preliminaries

Install the free version of [Docker Desktop](https://www.docker.com/).

Create an [AWS account](https://aws.amazon.com/). 
Take note of your default region. 
Mine region is `us-west-1`. 
Let's call it `$region`.

Install the AWS CLI. I like to use homebrew a la `brew install awscli`.

Install the AWS EB CLI. I like to use homebrew a la `brew install awsebcli`.

Create an [ECR repo](https://aws.amazon.com/ecr/) called `rshiny-base`. 
Take note here of your AWS account ID, which I will call `$aws_account_id` below. 
It's a bunch of numbers.

## Create a Base Dockerfile

Now make a file called `Dockerfile.base`.
You'll be pulling a base Shiny image
and then this is where you're going to want to 
install additional R packages.
You're installing them here because it could take a while,
and Elastic Beanstalk would time out if you did it downstream of this. 
Here I'm installing ROCR and gbm.

```docker
FROM rocker/shiny
# Install more R packages like this:
RUN . /etc/environment && R -e "install.packages(c('ROCR', 'gbm'), repos='$MRAN')"
``` 

For added stability you can pin to a specific `rocker/shiny` version, 
e.g. 3.4.4, with `FROM rocker/shiny:3.4.4`. 
I'm sure there's a way to pin R packages as well
but it's not on my fingertips and so I'll add that in later.

Now build the base image locally and call it `rshiny-base`.

```bash
docker build -t rshiny-base -f Dockerfile.base .
```

Then push the image to ECR.
Follow the instructions on ECR web site on how to authenticate and push, 
but here's what it looked like
at the time of writing.
You're logging into AWS, 
building the image locally (can skip this if you already did it), 
tagging the locally built image as latest,
then pushing the local image to ECR.

```bash
# region="us-west-1"
# aws_account_id=123456789
aws ecr get-login-password --region $region | docker login --username AWS --password-stdin ${aws_account_id}.dkr.ecr.${region}.amazonaws.com
docker build -t rshiny-base Dockerfile.base
docker tag rshiny-base:latest ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/rshiny-base:latest
docker push ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/rshiny-base:latest
```

The upload took a while for me. 
If I had to do this repeatedly I would set up an automated job 
to build and push using AWS Batch or,
much more likely, 
as part of a CI/CD pipeline using AWS CodePipeline. 

## Create Any New Shiny Apps

Make a directory called `apps/` and put a simple working app spec there, 
copied from, say, the [Shiny gallery](https://shiny.rstudio.com/gallery/).
Here's the subdirectory structure
for a bunch of custom apps.

```
apps/
├── hello-world/
|   ├── server.R
|   └── ui.R
├── app1/
|   ├── server.R
|   └── ui.R
└── app2/
    ├── server.R
    └── ui.R
```

You will access them after EB deployment at `http://<url>/hello-world/`,
`http://<url>/app1/`, and so forth.

## Create a New Dockerfile for Your App Server

Now create a file called `Dockerfile` with the following contents.

```docker
FROM <aws_account_id>.dkr.ecr.<region>.amazonaws.com/rshiny-base
USER shiny
COPY apps /srv/shiny-server
EXPOSE 3838
CMD ["/usr/bin/shiny-server.sh"]
```

The key thing we are doing here is copying your custom apps
into the Docker image itself. 
You can try to build it and run it like this.

```bash
docker build -t rshiny-apps .
docker run --rm -p 3838:3838 rshiny-apps
```

Now open http://127.0.0.1:3838/. 
You should see a message letting you know the server is running properly. 
You can create and edit a custom home page at the apps root, 
`apps/index.html`. 

## Commit to Git

The EB CLI zips your latest git commit on your configured default branch.
**It does not zip your latest changes if you have not git committed them.**
Can't tell you how many times I've forgotten to commit.

## Push It to Elastic Beanstalk for the First Time

Here's what I did to create an application called `shiny`,
from the root directory of the git repository.

```bash
eb init
eb create shiny
```

And you're done! Go to the 
Elastic Beanstalk
to find your (obscure) URL.
You should see your index and you can visit the 
`http://<url>//example-app/` path from there.

## Make Changes and Push Again

Make your changes, git-commit, then

```bash
eb deploy
```

So easy.

# Next Steps

* Build Dockerfile.base automatically in a CI/CD pipeline.
* Find a way to install packages in Dockerfile directly during app development to shorten the loop.

Enjoy!
