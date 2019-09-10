---
title: "How Null Hypothesis Testing Works"
date:   2019-08-29
published: true
sidebar: toc
layout: post
permalink: "/Null-Signficance-Hypothesis-Testing"
---

### General definition
A hypothesis test, fundamentally within the Frequentist paradigm, means that for each "test", we commit to an exact model for what probability distribution our random variables are drawn from, and by looking at how likely this model says we will see the data that we actually recorded, we will either "reject" or "not reject" this model based on a gut-based likelihood threshold. (I am using quotation marks because this whole rejection terminology has very domain-specific meaning, as used by frequentists).

This assumed model/distribution is called the *null hypothesis*, and is generally chosen as the safest default belief state.

For example, null-hypothesis significance testing is built into the US courtroom policy of "innocent until proven guilty": here, it is important to the interests of democracy that our courts default to the decision that people are innocent (the null hypothesis) until sufficient data is presented which is unlikely *enough* to agree with the null hypothesis that we can reject it; declaring that it is likely enough that the person is guilty that we choose to reject the null hypothesis.   

* Always a yes/no question.
* Always follows the following set of steps.

### Example 1: Is a coin fair?


### Exampe 2: A courtroom case of a crashed car.


### How to pick the 'null hypothesis' in the face of seeming symmetry.
It's important to realize that which of the two hypotheses we designate the 'null hypothesis' is always a little bit arbitrary: the math will often look the same.

Helps you choose: which direction of false-positives / false negatives seems the more egregious sin to you?
* In US courtroom system, "innocent until proven guilty" shows that we would much rather a true offender is unconvicted than a true-non-offender is convicted.
* In most science studies, we would much rather mistake a true novel result for the status quo rather than vice versa. (Look at how much more shame and embarrassment is heaped on scientists claiming to break energy conservation vs scientists who accidentlly overlooked earth-shattering new results for a few years).


### Pros and cons of NHST (and Frequentist stats, generally)
Pros
* Has worked quite well in the 20th century.
* Computationally quick.

Cons
* Assumes model that we have NO IDEA of confirming, leading to some ridiculous errors.
* Philosophical meaning of repeated experiments often is a little silly for very small values of $$n$$.
* Simple NHST has only a yes/no binary output, when often we would like a range of outputs. Thus leading to performing NHST over a range of values to estimate the maximum likelihood value, which is now getting into Bayesian territory.


### The meaning of a confidence interval  
