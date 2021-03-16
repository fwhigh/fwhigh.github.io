---
title: "Ad Muter: Idea"
date: 2021-03-01 12:00:00 -0700
comments: true
categories: 
  - ad muter
  - machine learning
tags:
  - maker
  - open source
  - bot
excerpt: "I got an idea to build a machine learning model to mute my TV when commercials come on"
---

{% include toc %}

# Intro

Every time commercials come on TV I melodramatically go "ugh!" and mute them. My 3 year old daughter likes to ask me "daddy, do you love commercials?" and
I tell her "no, I hate commercials, do you like commercials?" and, naturally, she tells me she loves commercials. 

I remember my dad muting commercials when I was a kid, and I would roll my eyes at him, too. The circle of life is complete.

Now I love technology and programming and especially data, but as the world's biggest introvert I don't like using my own vocal cords so much, so I have a mixed 
relationship with voice controlled systems. It actually does irk me to say out loud "Ok Google, mute living room." I'd much rather hit a button. 
I'm pretty sure I'm not alone in feeling this way about voice controlled systems. So, as a professional machine learning guy, I thought 
*maybe I can build an automated ad muter*. 

Part of me wants my little tiny kids to understand what I do for a living. I absolutely love what I do and just want to share that with them. 
Can I find a way to share this part of myself with them through this project? They would certainly viscerally beging to understand the impact of my skillset
when the commercials start muting themselves without me grumpily yelling "Ok Google, mute living rooooooom" anymore. 

Part of me wants other students and professionals who are interested to understand my working process, warts and all. 
Reason for that is, I can confidently say from year after year of churning projects out at work and previously in academia that my process is 
somewhat unique and valuable in certain key ways. 

So, here we are. Before I embark on building an ad muter whatsoever, I'm going to start out by telling you about the idea, how I think I'm going to proceed,
and what I'm guessing the challenges are. I'm going to carry you through my entire end to end process. 

# The idea

## What should it be like to use it?

When a commercial comes on, I want my TV to be automatically muted with no intervention from me. When the show comes back on, I want my TV to be 
automatically unmuted. I'm ok with correcting the model, and using those corrections as training labels. 

## What do I imagine the solution will look like?

My first thoughts are, build a supervised machine learning model that uses audio and video features to classify whether what's on right now
is a show or a commercial. I came through academia doing static image analysis so I'm somewhat confident in my abilities there. 
I've never done audio signal processing for ML and would love to learn, so this seems like a great excuse. There might be other
features, like clock time and maybe scraping TV programming schedules for what's on.

While I'm ok with just trying features out, for effieicy I'd like to prioritize features that correspond to my own intuition 
about when commercials are about to start or end. 
How do I personally know a commercial is coming when I'm watching TV? 
During a TV segment, the clock time can be a good indicator. How long the segment has been going on 
seems important, but the minutes on the clock also seems to matter. I once heard that TV stations tend to slot commercials somewhat uniformly 
into certain times in order for Nielsen to be able to accurately measure viewership with differential accuracy calibrated across stations. 
I've noticed that when commercials are on on one station they are often on on another station when you flip through. 
I bet I could use this fact to advantage. 

As for when I know commercials are ending, again how long it's been since commercial segment started seems to matter, and minutes on the clock may matter
again. I've also noticed there that the commercial-to-show time gap is (near) imperceptibly longer than commercial-commercial time gap. 
And often the commercial preceeding the show segment is for an asset of the TV station itself, like another show it airs or another
property of the parent company -- like Disney in the case of ABC in Los Angeles. 
That seems hard to featurize, but it's a thing I've noticed. 

And where would the model live? I could use my Mac Airbook but I bet image and audio signal processing will heat it up pretty good. 
Maybe I should try to deploy it to AWS up front to maintain that option. 

# I believe it can be done

I've noticed I tend to not shoot down ideas at early stages, and I also have worked with people who do. This might be the result of some
combination of my personality and my training during the PhD. I understand the urge to throw your hat in the ring and offer a take, and to 
want to influence people's behavior from your credibility. 

But even so, empricism trumps opinion. Making hypotheses is critically important because
it forces you to gather available information and try to guess correctly what the outcome will be in advance -- this is an extremely useful muscle
to exercise because it makes your guesses more accurate over time. 
But actually trying something out -- prototyping a model in this case -- is the only real way to know if it's possible. 

I firmly stand behind this thinking, and I challenge all data scientists and engineers 
to avoid killing projects too soon based on their own or other people's negative opinion, 
especially if those other people are higher in a power hierarchy. 
Experimentation beats opinion, regardless of title or pay, hands down, all day every day. 
To practice this is to develop true credibility. 

The best measures of whether to embark on a project are to gauge *how valuable* it is
and, above all, *how passionate you are* about building it. I personally feel this ad muter project is fairly useless, but I'm 
passionate about it because I love building ML things, and I want to use it to help my kids to understand me a little better. 

# What's next?

My first goal will be to build a first predictive model on my laptop. 




