---
layout: post
title: "Gibbs sampling"
categories: Statistics
author: Cam Benesch
meta: "Chicago"
---

We'll give an overview of Gibbs sampling here, and point to a bit of interesting research regarding its use. Some background in probability is assumed.

# Overview

In statistics, a probability distribution describes many outcomes, assigning a probability to each outcome. Sometimes, each outcome requires multiple values to describe it. For instance, we might have a probability distribution jointly describing people’s height and weight. One outcome would be 180 pounds, and 53 inches, which is described by multiple values. Such a multidimensional distribution is referred to as a joint distribution. 

We might want to compute a statistic which summarizes a joint distribution. For instance, we might want to know the average BMI. Some distributions are well-studied and easy to represent, so for those joint distributions, we can use or derive a formula for the statistic. For others, we might not be able to directly compute the statistic, so we have to choose a bunch of samples, and use those to estimate the statistic. In our example, choosing 10 people and averaging their BMIs would help us estimate of the average BMI. This is called sampling. 

Sampling isn’t always as easy as choosing people. Sometimes the distribution is only described abstractly, and we don’t have the population at our disposal. In this case, we need to generate a sample on our own. This can often be done on a computer, but sometimes a joint distribution is too complicated for the computer to sample from directly. 

So we might first give the program an arbitrary height, say 60 inches. Then out of people who are 60 inches tall, the computer samples a weight, say 145 pounds. Then, out of people who weigh 145 pounds, the computer samples a height. And so on. Of course, we are assuming that these conditional samples are simple enough to compute. If that is the case, we can follow the procedure described to generate a sample from the joint distribution, and use the average BMI of that sample as our estimate. This technique is an example of Gibbs sampling. 

In this post, we'll motivate sampling in general, and introduce Gibbs sampling for a two-dimensional joint distribution. Then we can formally define it for two or more dimensions. Gibbs sampling has been an active area of research for decades, so I'll discuss a few relatively recent results. Gibbs samples form Markov chains, which means draws aren’t independent from one another, as would ideally be the case. Two techniques which attempt to address this are called burn-in and thinning, and I'll discuss when and why they might be useful. I'll give a well-known example of where Gibbs sampling fails. The order of sampling from the conditionals may also affect our Gibbs sample, and we'll discuss the size of such an effect. Finally, I'll illustrate an application called Hierarchical Bayes, which is an extension of classical Bayesian estimation involving a multi-level prior distribution. 

