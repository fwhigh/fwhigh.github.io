---
title: "Real-time Machine Learning Model Inference on AWS"
date: 2021-03-21 12:00:00 -0700
comments: true
author: "Will High"
categories: 
  - Machine Learning
excerpt: I get a nearly 6x speedup over standard grep by using GNU parallel.
---

{% include toc %}

# tl;dr 

You can do a ton of damage building batch-trained and real-time-scored machine learning
model infrastructure with existing AWS services, and you can get started with it with 
low cost.

Github repo [here](https://github.com/fwhigh/predictive-models-in-prod).

# Introduction

Developing and playing with a machine learning model on your laptop is fun, 
but what do you do when you want others to use it? 
Or, say, when you want to run and rerun training and prediction automatically over time with new data, 
without having to keep your laptop awake? Will your model live on if you leave your job? 
Putting a predictive model into production solves these problems — and many more, 
depending on how sophisticated you can get with it.

In this tutorial I’ll show you what I consider to be a near-minimal production architecture 
for serving models and predictions to your colleagues and potentially to the general public. 
The goal is to cleave the models and their outputs from you and your computer. You will

1. train a predictive NLP machine learning model that pulls data from AWS S3,
1. push the model and metadata back into S3 and version them,
1. create an API service that loads the latest model and makes predictions on the fly, and
1. automate re-training on a regular schedule in an AWS production environment.

Here’s a sketch of the components.

![](/assets/real-time-ml-aws/Predictive-Models-in-Production.png 'Predictive models in production architecture diagram'){:width=""}

Your model should classify comments like “Check it out this free stuff” as spam 
and “I take issue with your characterization” as not spam. 
The predictions will be accessible via a web based API service that can be called like this from the command line:

```bash
curl -X POST "http://pmip-env.v4fisfj3tm.us-west-1.elasticbeanstalk.com/predict?type=class" -H "accept: application/json" -H "Content-Type: application/json" -d "{ \"comments\": [ \"Check it out this free stuff\", \”I take issue with your characterization\" ]}"
```

and give you outputs like this:

```bash
{"result": [{"class": 1}, {"class": 0}]}
```

This solution costs me cents per month in AWS costs.

Please note that this tutorial is not an exhaustive step-by-step recipe 
but a sketch of the overall system with pointers to other tutorials, 
though I do design in customizations as needed and I strive to point out the gotchas. 
My School of Data session is 1.5 hours and it’s not possible to complete all of these steps from scratch in that time. 
More realistically it takes days of solid work, or weeks if you’re learning new skills in the process.


The code is checked into a GitHub repo:

https://github.com/fwhigh/predictive-models-in-prod

I’m calling the Python package there pmip.

If you have questions/comments/suggestions, please reach out on Twitter @WillHighSci or interact on Github. 
Pull requests are encouraged.

# Some big picture things to note

## Your Jupyter notebook is your production code

A necessary decision to make is whether your production code will be a Python package 
or even a simple Python module or script.

Most of us do exploratory analytics and modeling in Jupyter during the learning 
and proof-of-concept phase, 
so if we were to go the Python package or module we would have to refactor our code. 
This compounds the work, and it presents a risk of introducing bugs. 
I take the point of view that the distance between exploratory analysis 
and production should be as small as possible. 
So we’ll be using our exploratory Jupyter notebooks as our production code using the beautifully elegant package Papermill. 
Papermill lets you execute Jupyter notebooks programmatically with arguments in scripts, 
and save the resulting notebook.

We will actually simultaneously be developing a Python package. Sometimes it’s just better to promote a very useful utility function or class to a package so you can reuse it in multiple places. Over time you’ll get a feel for what belongs in a notebook and what belongs in a package.

## Look out for three ✨Magical Things✨

I’ve got three neat productivity tricks that together radically shorten your local development cycle, which is the time between editing code and testing it out. They allow you to edit package code either inside or outside the container, edit the Jupyter notebook inside the container from your browser, have package changes be immediately available in the notebook, and have both package and notebook code changes persist when you shut down the container and ship it to production. Look out for the tips in the model building section below.

# Glossary

*CLI*: Command line interface. These are shell commands like ls , cd , and git. All commands are Unix/Linux and were tested on Mac and Ubuntu. This tutorial assumes you’re comfortable with the command line. If you’re not and you’re serious about productionalizing machine learning models, please consider learning it.

*Productionalize*: To put a design/solution/idea into production, as in a factory. Orwellian tinges aside, it is an extremely useful term for communicating what we’re doing here. Contrast productize, which is to create a useable, useful, possibly delightful product out of an idea for people to consume.

*API*: Application programming interface. In our context this is shorthand for “web API”, most often RESTful web services. We are implementing a RESTful web API service for general use.

*IDE*: Integrated development environment. This is code editing software with additional convenience functionality to make development faster and better, like PyCharm for Python, RStudio for R, and Eclipse for Java. SublimeText, Emacs, and Vim also count as fully fledged IDEs, particularly due to their extensibility via plugins.

*Container/image*: These are like virtual machines, but much more efficient. I’ll use Docker for containerization. The image is the compiled container byte object and the container is a running instance of the image. I am certainly not an expert in modern container technology even though I use them a lot, and I’m likely to misuse language here — I hope container wonks will correct me. Let me just say that I love them and it’s hard to overstate the usefulness and power of containers.

*Batch processing*: Typically long-running computational jobs, like training a model or processing a lot of data on a schedule or based on some external event. AWS Batch is a service to do batch processing.

*Real-time prediction*: This means low-latency predictions on demand. It can either be fast access to precomputed static data or predictions that are computed on the fly against user-supplied inputs that cannot be tabulated in advance. We’re doing the latter, because we can’t precompute all possible comments people will make, spam or otherwise.

There’s lots of other jargon here that I’ll assume you already know or will Google.

# Stuff to do in advance

I suggest installing a package manger. 
For Mac, Homebrew is popular. 
It requires installing Xcode from the app store and agreeing to the terms.

## Install Python

I recommend installing Python via Homebrew; make sure it’s set as your default once you’ve installed it. This can be tricky. At the time of writing my solution was to prepend /usr/local/bin in my .bash_profile like this:

```bash
echo export PATH=\"/usr/local/bin\":\$PATH >> ~/.bash_profile
```

In the past I’ve also had to

```bash
cd /usr/local/bin && sudo ln -s python3 python && sudo ln -s pip3 pip
```

to make Homebrew’s Python 3 my default. These are just some tips — you’ll have to play around and figure it out for your situation.


## Create an AWS account and increase your EC2 limits

Go to AWS and create an account. Enter your credit card information. I’ll do my best to radically minimize costs but it’s your responsibility to monitor and control them, obviously.

Once you create your account, view your AWS EC2 limits by going to the EC2 landing page and selecting “Limits” on the left to see your current limits, which are probably very low or zero. Request from AWS an EC2 limit increase of at least 1 m4.large managed instance type. This enables you to use AWS Batch, which doesn’t use free-tier eligible instances at the time of writing.

## Install and configure the AWS CLI and the AWS EB CLI

Follow the instructions here. It involves installing the CLI software and setting up a credential for your AWS IAM user. The instructions are all there. I prefer the `brew install awscli` approach instead of pip installing — just a personal choice.

You’ll be choosing a default region. Remember it. You might as well make it close to you, so if you’re in Los Angeles, why not choose us-west-1?

Install the AWS ElasticBeanstalk CLI as well. My preferred method is `brew install awsebcli`.

## Create a Github account

Create a free account on Github. I also recommend installing the Github Desktop client instead of using the git CLI, but again this is a personal choice.

## Install Docker

Install Docker community edition (free), then run it on your machine.

## Additional recommended software

Some additional software I would recommend.

* A good Python IDE, like PyCharm or SublimeText.
* wget for downloading stuff in shell scripts.
* jq for manipulating JSON directly from the command line.

## Seed AWS S3 with initial training data

Go to AWS S3, click “Create Bucket”. Choose a bucket name and remember it. Please make yourself aware of whether the public has access to your bucket — you can make it private or public.

Click on the new bucket to navigate inside of it, then do the following.

* “Create folder” and call it training, then save.
* Inside training, again “Create folder” and call it 20190101, then save.

You should now have an S3 path s3://<your-S3-bucket>/training/20190101.

Download this zip file to your local computer, then upload that same file back into your new S3 path. You can use the UI or the AWS CLI.

When you’re done you should have a file at s3://<your-S3-bucket>/training/20190101/YouTube-Spam-Collection-v1.zip.

The reason I’m giving it a date of the beginning of this year is that I want the regularly scheduled model training job to pick up the latest data at the time of training, and I’ll do that by having the job look for the largest YYYYMMDD subfolder in s3://<your-S3-bucket>/training. So if you have new comments labeled as spam or not, you would put all the training data, old and new, in s3://<your-S3-bucket>/training/<YYYYMMDD>/ using today’s date, and your next model training run in production will pick it up. At any time you can also clean up old training data directories, or just keep them for posterity.

There are other ways to add in new training data to the system, subject to your own needs and creativity.




# Build a predictive model

Let’s build a spam predictor using the YouTube comment data set in the UCI ML repository. You’ve already pulled down the training data into your S3 training folder. You’ll now build an NLP model that predicts whether a short comment is spam or not.

## Build the Docker image

I’ll use a basic Python 3 image, but you can use fancier images that are preconfigured for deep learning or other state-of-the-art algorithms. Build the image by running

```bash
bash scripts/build_training_image.sh
```

Go have a look at that script to see what it does. It’s a fairly standard Docker build command that provides the image with some run-time variables at build time. During the build, the training Dockerfile Dockerfile.train installs the local pmip Python package using the scripts/install.sh script; we’re using pip install’s -e option there. This installs in editable mode so that you don’t have to reinstall the package every time you make changes to the pmip source code.

Pip installation in editable mode is ✨Magical Thing Number One✨.

## Train the model interactively in the Jupyter notebook


Have a look at the Jupyter notebook that trains the image. You can run the notebook yourself from your new container by executing

```bash
ENVIRONMENT=dev bash scripts/run_training_container.sh -c "jupyter notebook notebooks/ --allow-root --ip 0.0.0.0 --port 8888 --no-browser"
```

The port 8888 is exposed in your training Dockerfile, Dockerfile.train. The IP 0.0.0.0 lets the container’s web server map to your localhost. In the training script the docker option -it lets you run commands interactively, -p 8888:8888 exposes the port (and maps to 8888) during the session, and --mount with arguments mounts your working directory pwd to /usr/src/app within the container and makes any changes you make inside the container immediately available in your local filesystem outside of it, and any changes you make on your local filesystem in that directory available inside the container.

This cross-mount pattern is very, very wonderful — it’s ✨Magical Thing Number Two✨. It enables you to

*  make changes to the pmip Python package on your laptop, outside of the container, immediately available inside of the container, which is optimal for local development;
* make any data you write to /usr/local/app immediately available locally in pwd as well, and live on after you shut down your container interactive session, which is useful for optionally running pmip code locally outside of the container if you want to and for persisting data across development sessions;

Now that you’ve run the container with the notebook command you can open http://localhost:8888 to see and run the training notebook in your browser. If run the whole notebook, you’ll get a serialized model object in the data directory. You can also make changes and save the notebook, and because of the Docker mount pattern I used, those changes will be reflected in your local machine’s filesystem and be git commit-able back into Github.

Note the use of the autoreload Jupyter magic command at the top of the notebook. This is ✨Magical Thing Number Three✨. In combination with editable install and the Docker mount pattern, now, when you edit pmip package code locally on your machine — say in PyCharm or vim or whatever— it is immediately reflected in the notebook inside the container during your session. You don’t have to reinstall the pmip package nor re-execute all the code from the top of the notebook on every code change you want to test.

These three Magical Things together make your local development loop extremely short. They may well give me the biggest speedup to my own software development productivity out of anything.

## Train the model programmatically on your local machine

Now that you’ve run the notebook interactively and possibly made edits and saved, try running it programmatically like this:

```bash
ENVIRONMENT=dev BUCKET="s3://<your-bucket>" bash scripts/run_training_container.sh scripts/train.sh
```

Please read training script source code. It

* pulls down the latest data from S3,
* runs the notebook programmatically using Papermill, which creates a new model.pkl, and
* makes an HTML file from the executed notebook for your own records.

When it’s done you can look at all the data artifacts in the local data directory.

## Seed AWS S3 with the trained model

Using what you now know about S3, go ahead and manually upload the model that you just created in thedata directory, so that you end up with an S3 file that looks like this.

```
s3://<your-S3-bucket>/models/staging/20190101/model.pkl
```

## Create an AWS ECR repo and push the image to it

We’ll be pushing our Docker image to AWS ECR. Set up a repository called pmip using the UI, and take note of the URI.

Now push the Docker image to it.

```bash
bash scripts/push_training_image.sh <your-ECR-repo-URI>
```

If this is your first or only ECR repo then you can also get your repo URI by running

```bash
aws ecr describe-repositories | jq -r '.repositories[0].repositoryUri'
```

# Create an API to make predictions in real-time

I make heavy use of APIs in my daily work. AWS Lambda is a really exciting way to serve APIs and pay on a per-function-call basis rather than pay for an always-on instance. I have checked in code and notes to deploy to Lambda using the serverless node framework. I ran into surprisingly subtle challenges getting the Lambda function to pick up the latest models from S3 in an elegant and “correct” way, so I’m going to outline a simple solution using ElasticBeanstalk here instead.

I’ve written a Flask application that implements endpoints in pmip.routes. You can run this application locally with

```bash
ENVIRONMENT=dev BUCKET=<your-S3-bucket> bash scripts/run_api_container.sh
```

and then opening http://localhost:8000 in your local browser. I used Flask-RESTplus, which automatically produces Swagger documentation for the API. The landing page at the root of the URL is a lovely rendering of the Swagger object. You can configure what gets displayed there in the pmip.routes submodule, and you can actually run API calls directly from the UI as well. Super convenient for you and your API consumers.

To create the ElasticBeanstalk application go to the EB homepage, select your region and click “Get started”.

* Application name: pmip
* Platform: Docker
* Click “More configuration options”
* Under “Software” click “Modify”
* Under Environment properties create ENVIRONMENT with value staging and BUCKET with value <your-S3-bucket> .
* Click “Apply”
* Click “Create application”

In your local development directory do

```bash
eb init
```

Then

* Choose your region
* Select the application you just created
* Do not use CodeCommit


Now go to AWS IAM and select your region.

* Click aws-elasticbeanstalk-ec2-role
* Click “Attach policies”
* Search for and select AmazonEC2ContainerRegistryReadOnly,
* Search for and select AmazonS3ReadOnlyAccess
* then click “Attach policies”

Commit your latest changes to the code, then run
eb deploy
That’s it. Your application is using your training image in ECR as the base image. That’s where the actual API code lives — we’re not pulling from Github or a PyPI (pip) repository. The API application uses gunicorn with 3 workers.
Navigate to the EB application environment page (make sure you’re using the right region). You’ll see the deployment status and a link to your application’s URL. Once it’s live you can read the Swagger documentation there and play with predictions. Check the /model-info endpoint — it should indicate that you’re using the latest model ID from your staging S3 subdirectory.
This is your minimal production API.










