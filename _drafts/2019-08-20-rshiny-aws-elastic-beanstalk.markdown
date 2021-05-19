---
title: "Stand Up a Rocker Shiny Server Using AWS Elastic Beanstalk"
date: 2019-08-20 14:05:20 -0700
comments: true
categories: 
---

Here's how I set up my own R Shiny server using AWS Elastic Beanstalk.

I first tried pull `FROM rocker/shiny` directly in a Dockerfile and deploying to an Elastic Beanstalk single-container Docker instance, but the EB deployment timed out.

I increased the instance to a 

The second thing I tried was something I've done successfully in the past, which is to create a new AWS Elastic Container Registry (ECR) repo and have EB pull from that whenever I deploy a new application. Here are the steps.

# Create a base Dockerfile

Create a file called `Dockerfile.base` with a single line:

```
FROM rocker/shiny
```

# Create an ECR repo 

Call it `rshiny-base`.
