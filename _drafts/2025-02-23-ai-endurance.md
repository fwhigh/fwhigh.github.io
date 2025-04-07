---
title: "What Is Enduring in AI"
date: 2025-02-23 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - AI
  - Machine Learning
excerpt: Perspectives on the different ways to build and use AI -- and which have staying power
toc: true
toc_sticky: true
author_profile: true
---

AI is in another hype bubble. 
Some of AI's value today is very real and some not. 
My goal is to evaluate what AI use-cases will endure and what will die with this new bubble. 

# Nobel Prize worthy AI: discovery engines 

Let's start with the Nobel Prizes. 
AlphaFold2 shared the [2024 Nobel Prize in Chemistry](https://www.nobelprize.org/prizes/chemistry/2024/press-release/)
with an early protein design scientist.
The [2024 Nobel Prize in Physics](https://www.nobelprize.org/prizes/physics/2024/press-release/) went to Hopfield and Hinton for inventing the very neural networks that eventually enabled AlphaFold2 to be so effective.
While the chemistry prize makes sense, the physics one is a head scratcher. 
There are connections with statistical physics via Boltzmann machines and with discoveries in the physical world such as AlphaFold2 and new materials and designs in general, but at the end of the day this work was not physics as we previously knew it. 
What is undeniable is the Nobel Prize committee was willing to give essentially two prizes to AI in the same year and kind of bend over backward to do so.
Were they right?

Yes, extremely. 
The prizes highlight what is most valuable and enduring about AI: it is a platform and an engine for novel designs and discoveries that go well beyond human capabilities.
Besides designing new proteins, this also means 
* designing genes [1](https://engineering.berkeley.edu/news/2025/02/new-ai-breakthrough-can-model-and-design-genetic-code-across-all-domains-of-life/)
* designing new alloys, polymers, and other physical materials from molecular building blocks [1](https://news.mit.edu/2023/learning-language-molecules-predict-properties-0707)
* designing new computer chips [1](https://www.livescience.com/technology/computing/humans-cannot-really-understand-them-weird-ai-designed-chip-is-unlike-any-other-made-by-humans-and-performs-much-better) [2](https://ece.engin.umich.edu/stories/ece-faculty-design-chips-for-efficient-and-accessible-ai) [3](https://www.theengineer.co.uk/content/news/ai-confounds-humans-with-improved-chip-designs/)
* discovering how the brain works [1](https://www.cmu.edu/news/stories/archives/2022/february/cmu-neuroscientists-use-deep-learning-model-to-simulate-brain-topography)
* solving math problems and discovering novel solutions to old ones [1](https://deepmind.google/discover/blog/alphageometry-an-olympiad-level-ai-system-for-geometry/) [2](https://www.livescience.com/technology/artificial-intelligence/math-olympics-has-a-new-contender-googles-ai-now-better-than-human-gold-medalists-at-solving-geometry-problems) [3](https://thenextweb.com/news/deepminds-ai-finds-solution-to-decades-old-math-problem) [4](https://www.sydney.edu.au/news-opinion/news/2021/12/02/mathematicians-use-deepmind-ai-develop-new-methods-problem-solving-proofs-conjectures.html)
* creating novel console/PC games or gameplay [1](https://www.techradar.com/gaming/gaming-industry/microsofts-new-breakthrough-generative-ai-model-is-designed-to-create-consistent-and-diverse-gameplay-and-could-be-used-to-preserve-classic-games)
* design more efficient power grids [1](https://arxiv.org/html/2407.04522v1)
* discover optimal supply chain and logistics solutions [1](https://www.mdpi.com/journal/applsci/special_issues/5XYQ47LRTZ)

and more, all at varying stages of maturity.

# The AI being sold to us: the smartest friend you've ever had

The biggest AI product being hawked to everyday consumers and businesses is conversational and "agentic" AI,
including chatGPT, Gemini, Claude, Perplexity.
Conversational AI just means you can ask a question and the AI will give you a very well written and often accurate response, as if there was an incredibly competent person on the other end.
Agentic AI means it can also take actions itself, including contuct a live search, retrieve text or data, or make API calls to other systems connected to it over the internet.
Think "Alexa on steroids".
Not all consumer AI products are created equal, and they also evolve and improve dramatically over time.

Is this even the same AI as the Nobel Prizes were calling out? 
Yes and no.
Yes, these AIs use some of the same deep learning components that AIs used to make new designs and discoveries use.
Yes, their outputs are stunningly uncanny. They even take standardized tests better than people. 
No, their outputs are not otherwise groundbreaking. 
These AIs are undeniably generating high quality text and actions but 
their outputs do not consitute fundamentally new discoveries. 

How do we make sense of what is happening? 
I decided to ask Gemini (Flash 2.0) 3 physics GRE questions from a practice test I found online, which included an answer key. 
Gemini provided step by step reasoning and got all 3 right, even when I shuffled the multiple choice answers.
I asked Gemini to show me a closed form solution to the time independent Schrodinger differential equation and it showed me solutions for three different potentials.
I then asked Gemini to show me a closed form solution to the quantum chromodynamics field equations, and it said no solutions are known and discovering one would be a major breakthrough.

My point is that discoveries aren't made from deep learning models trained on the internet.
These consumer AIs are extremely useful for research and study, 
creating and editing documents and images,
analyzing data


# A framework for evaluating AI

Not


# The Facets

## Discovering Novel Solutions

Designs that humans would not have come up with.
This is what essentially won the 2024 Nobel Prize.
Requires custom models that often use sythentic training data, for example from a physics engine

Examples:
* Alphafold
* Writing CUDA code (for AI!)
* Designing new printed circuit boards
* Playing video games
* Robotics

## Personal Agent

Orchestrator of actions in one's life or in specific situations. 
This is being tauted by the most vocal AI startup CEOs.

## Information Retrieval

A.k.a. search against the internet or other corpus of documents.
This is using AI to summarize multiple search results in prosaic narrative format.

## OCR on Steroids

AI today "solves" OCR.

## NER on Steroids

AI today "solves" NER.
Examples of this are easy to generate: upload a document or image and ask the AI what is in it.
An "on steroids" part of using AI for this is that you can also ask for internet-scale information about the named entity.

* Web scraping

## Style Transfer

ABC

## Transfer Learning

ABC

## 