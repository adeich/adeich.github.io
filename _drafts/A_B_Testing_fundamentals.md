---
title: "A/B Testing in Python: a tutorial"
date:   2019-08-29
published: true
sidebar: toc
layout: post
---
A/B testing is, at its core, the following puzzle: we run two, slightly different versions of the same experiment: call them versions $$A$$ and $$B$$. After, say, 100 trials of each, we see that one version (say, $$A$$) succeeds slightly more times than $$B$$ does. The question is, is $$A$$ *truly* more likely to continue its superiority into the future, or is the difference we see in our data due more likely due to random chance?

A fascinating, beautiful thing about A/B testing is how rich the thought experiments, the mathematical models, and *what if* questions that roil beneath the surface, given that there are few data sets as slim as the two-or-four numbers that tend to comprise a given A/B dataset. And, just as the models are rich and many, the models' predictions are just as simple as the data: single answers come out: "Yes"; "B is likely better than A" ... 




### An illustrative example: campaign donation button.
You've set your political campaign's website to randomly serve, with equal probability, one of two different videos to viewers: (1) a negative attack ad of the political opponent, Mr. Meow-Meow, and (2) a positive ad for your candidate, Whiskers McGee.

For each view, your web server records which ad each viewer watched, then whether or not they donated a fixed amount of $5.

After letting this experiment gather data for a day, you end up with the following:

|Video | Total visitors | Visitors who donated $5 |
|--|--|--|
|Attack Ad | 550 | 200 |
|Positive Ad | 555 | 220 |

From this data, our fundamental question "Which version is better?" really breaks down into three, similar-but-subtly-different parts:

1. Which of these versions of the website causes people to donate more money, if either?
1. What is the probability of our simulated action agreeing with its real-life results (say, three months down the road)?
1. How stable or unstable does the estimate of our error, above, depend on our choice? And what is our confidence in this?

This data, which consists only of  four numbers, is admittedly simple. An amazing thing about the tools of statistics is that there are several, powerful insights we can gain from just these four numbers.

What follows are the application of several related statistical models for helping us examine these questions.

### 1. Null Hypothesis Significance Testing

In a hypothesis test, fundamentally a Frequentist paradigm, means that we choose a precise model for what probability distribution our random variables are drawn from, and we will either "reject" or "not reject" this model based on how unlikely it says our data is. (I am using quotation marks because this whole rejection terminology is very jargon-ey, as used by freq) This assumed model/distribution is called the *null hypothesis*, and is generally chosen as the safest default belief state.

For example, null-hypothesis significance testing is built into the US courtroom policy of "innocent until proven guilty": here, it is important to the interests of democracy that our courts default to the decision that people are innocent (the null hypothesis) until sufficient data is presented which is unlikely *enough* to agree with the null hypothesis that we can reject it; declaring that it is likely enough that the person is guilty that we choose to reject the null hypothesis.   

#### NHST: assume people's preferences are drawn as Bernoulli variables.

#### NHST: student's t test




#### References:
Peter Wills, 2019 blog post: http://www.pwills.com/posts/2019/08/24/stats.html
Chris Stucchio, 2015: https://cdn2.hubspot.net/hubfs/310840/VWO_SmartStats_technical_whitepaper.pdf
David Robinson, 2013 Stack Exchange post: https://stats.stackexchange.com/questions/47771/what-is-the-intuition-behind-beta-distribution/47782#47782
