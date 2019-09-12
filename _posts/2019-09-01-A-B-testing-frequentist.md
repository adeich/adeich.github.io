---
title: "A general, frequentist model for A/B testing."
date:   2019-09-01
published: true
sidebar: toc
layout: post
permalink: "/A-B-Testing-in-python"
thumb: "/images/AB_binomial_thumb.png"
excerpt: "Starting with a broadly adaptable Python code snippet for an A/B hypothesis test, I explain what the statistics behind it mean and what the code's p-value can tell us."
---

A/B testing is, at its core, the following puzzle: we run two, slightly different versions of the same experiment: call them versions $$A$$ and $$B$$. After, say, 100 trials of each, we see that one version (say, $$A$$) succeeds slightly more times than $$B$$ does. The question is, is $$A$$ likely to continue its superiority into the future, or is the difference we see in our data due more likely due to random chance?

### An illustrative example: campaign donation button.
You've set your political campaign's website to randomly serve, with equal probability, one of two different videos to viewers: (1) a negative attack ad of the political opponent, Mr. Meow-Meow, and (2) a positive ad for your candidate, Whiskers McGee.

For each viewer, your web server records which ad each viewer watched, then whether the viewer donated a fixed amount of $1.

After letting this experiment gather data for a day, you end up with the following:

|Video | Total visitors | Visitors who donated |
|--|--|--|
|Attack Ad | $$n_A=550$$ | $$k_A = 200$$ |
|Positive Ad | $$n_B=550$$ | $$k_B = 220$$ |



From this data, our fundamental question "Which version is better?" really breaks down into three, similar-but-subtly-different parts:

1. Which of these versions of the website causes people to donate more money, if either?
1. How likely is it that our experimental observations will agree with long-term behavior (say, three months down the road)?
1. How stable or unstable does the estimate of our error, above, depend on our choice? And what is our confidence in this?

These are great questions, but they are better served by Bayesian approaches (I highly recommend David Robinson's [article on Bayesian approaches to A/B testing](http://varianceexplained.org/r/bayesian_ab_baseball/), written in R).

Instead, with the frequentist mindset, we say "Let us make some reasonable assumptions about the nature of the data, then ask how likely the data we're seeing is, given those assumptions." In A/B testing, our most fundamental question is whether A and B are equally effective, so our null hypothesis will tend to assume they *are* equally effective and look for how likely or unlikely this hypothesis is, given the data and a few other assumptions.

To further motivate talking about formal statistical distribution, I want to note that a bar graph, a fundamental visualization of our data, is not very informative to the naked eye about the likelihood of the difference due to chance:

![bar plot](\images\AB_barplot.png)

Further, our data, which consists only of four numbers {$$n_A, n_B, k_A,k_B$$}, is admittedly simple. An amazing thing about the tools of frequentist statistics is that there are several, powerful insights we can gain from this tiny data set if we make some very reasonable assumptions about the statistical nature of our experiment.

What follows are the application of several related statistical models for helping us examine these questions.

### 1. Null Hypothesis Significance Testing

See my post about [NHST](/Null-Signficance-Hypothesis-Testing)


#### NHST: assume people's preferences are drawn as Bernoulli variables.
Here is a model: every person looking at your website will draw from within themselves one of two Bernoulli variables:

|Population | model |
|-|-|
| People who see attack ad | $$A \sim \mathrm{Bernoulli}(p_A)$$ |
| People who see positive ad | $$B \sim \mathrm{Bernoulli}(p_B)$$ |


Basically, when a person opens the website and they see one of the two ads, this then chooses one of two coins $$A$$ or $$B$$ within them, and *then* this coin is flipped, i.e. the random sample is drawn. A Bernoulli random variable, as a reminder, is the abstraction of an ideal single coin flip that has probability $$p$$ of heads and $$1-p$$ of tails. So this model says that each person can respond with different probability, $$p_A$$ or $$p_B$$ of making the binary decision of donating money, depending on whether event $$A$$ or $$B$$ happens to them first.

Our model decided, all that follows is strictly from mathematical equivalence. A sum (in our case, $$n=550$$) of Bernoulli random numbers all with probability $$p$$ follows a binomial distribution, $$\mathrm{Bin}(n, p)$$:

$$\sum_{i=1}^{n} \mathrm{Bern}(p) = \mathrm{Bin}(n, p) $$

This distribution always has mean $$\mu=np$$ and variance $$\sigma^2=np(1-p)$$.

Having fixed upon a model, we can now ask precise model-behavior-matching questions of the data, such as: if we vary the difference in means $$\mu_A$$ and $$\mu_B$$, how will this affect the distribution which results from mixing an equal number of samples from $$A$$ and from $$B$$, with no knowledge of which sample came from which population? This is exactly where hypothesis testing comes in.

Intuitively, it seems totally plausible that, in reality, the two binomial distributions $$A$$ and $$B$$ have differing means, lining up with the data we recorded. For instance, consider if their means lined up exactly with our sample means:

![Binomial model](/images/AB_binomial_model.png 'title'){: .align-center}

But it's also totally plausible that $$A$$ and $$B$$ could have the same mean, still resulting in the data we recorded. As NHST (null hypothesis significance testing) frames it, the question becomes "How unlikely would our data be, given an assumed null distribution?". Ok, well how do we *quantify* how unlikely the data is?

#### Choosing a test statistic

Technically, a *statistic* is any function that maps your dataset to a single number. For example, a dataset $$X = \{x_1, x_2, ... x_n\}$$ has mean $$\overline{X}$$ and variance $$\sigma^2_x$$. These latter numbers are both *statistics*, because they are functions of the data with a single numerical output.  A bit confusingly, the word 'statistic' is commonly overloaded to refer either to the function or to a particular value of its output.

In NHST, statisticians tend to use the **z-**, **t-** and **f-tests**, each of which comprises a function (whose formula you look up) that maps the dataset to a single number (called, in case of the z-test, the "z-statistic" or "z-score") and a resulting, fixed distribution that each statistic follows, regardless of where the data comes from.

The reasons these statistical tests are so useful to null-hypothesis likelihood testing is that under specific data assumptons (such as normally-distributed data, known/unknown means and variances, etc) each statistic has a proven, known distribution--usually a normal or a t-distribution with $$\mu=0, \sigma=1$$. That statisticians are able to prove the distributions of these statistics following from the set of assumptions is incredible.

In the case of this article's canonical A/B test problem, our dataset happens to match up with assumptions of the [two-sample t-test](https://en.wikipedia.org/wiki/Student%27s_t-test#Independent_two-sample_t-test), which applies to two datasets, $$X_1$$ and $$X_2$$, and with the null hypothesis that their two means are equal, $$H_0: \{\overline{X_0} = \overline{X_1}\}$$. Further, the data must have the following properties:

|t-test requirements | Applies to our A/B success-rate data? |
| -- | -- |
|Data distributed normally | Yes. We are treating $$N$$  as our 'sample mean', for whose size, $$N \approx 200$$,  the CLT says means are distributed normally.
|Two sample sizes are equal | 200 vs 220. Roughly. |
|Variances are equal | Also roughly. |   

The t-statistic is defined as

$$t = \frac{\bar {X}_1 - \bar{X}_2}{s_p \cdot \sqrt{\frac{1}{n_1}+\frac{1}{n_2}}}$$

where $$s_p$$, called the *pooled variance*, is

$$ s_p = \sqrt{\frac{\left(n_1-1\right)s_{X_1}^2+\left(n_2-1\right)s_{X_2}^2}{n_1+n_2-2}}. $$

If the aforementioned assumptions about our data are true, $$t$$ will follow the [student's t-distribution](https://en.wikipedia.org/wiki/Student%27s_t-distribution):

![t distribution](/images/AB_t_distribution.png){: .align-center}

The proof that *t* follows the t-distribution is left as an exercise to the reader.

Let's pause to put into perspective why this is so amazing: no matter the dataset, if the above assumptions apply, then the t-statistic will *always* be distributed according to *this* t distribution. The same mean ($$0$$), same variance ($$\approx1$$), same cumulative distribution function, etc.

Put another way, collect *all possible datasets in the universe* which are consistent with said assumptions; for each dataset compute the t-statistic; plot all of these uncountably-many t-statistics on a histogram, and they will converge to the t distribution, plotted above.

So let's compute our dataset's t-statistic in Python:

~~~~ python
from scipy.stats import t

nA, nB = 550., 550.
kA, kB = 200., 220.
sigmaA = nA * pA * (1 - pA) # these variance formulae come   
sigmaB = nB * pB * (1 - pB) # from our binomial assumption.
pooled_variance = (((nA - 1)*(sigmaA**2) + \
               (nB - 1)*(sigmaB**2))/(nA + nB - 2))*(1/nA + 1/nB)
t_test_stat = (kA - kB)/np.sqrt(pooled_variance)
t_test_stat # = -2.601
~~~~

We plot $$t=\pm 2.601$$ on the x-axis of the t-distribution. Specifically, we want to know what $$t=\pm2.601$$'s resulting two-tailed *p*-value is, i.e. what is the fraction of area under the curve for $$\|x\|\geq (t=-2.601)$$?

Here we visualize this area under the curve, whose sum is the *p*-value for our t-statistic:

![CDF for p-value](\images\AB_t_p_value.png)

More directly, we compute this area by plugging our t-statistic into the t distribution's CDF. The CDF's output (times 2) is our two-tailed p-value:

~~~~ python
from scipy.stats import t

2 * t.cdf(-2.601, df=1100) # = 0.009411
~~~~

Also, here is the fast way to run the above calculation, thanks to `scipy.stats`'s convenience function, `ttest_ind_from_stats()`:

~~~~ python
from scipy.stats import ttest_ind_from_stats

nA, nB = 550., 550.
kA, kB = 200., 220.
pA, pB = kA/nA, kB/nB

ttest_ind_from_stats(mean1=kA,
                     std1 = nA * pA * (1 - pA),
                     nobs1=nA,
                     mean2=kB,
                     std2 = nB * pB * (1 - pB),
                     nobs2=nB,
                     equal_var=False)

# Ttest_indResult(statistic=-2.5579810729965056, pvalue=0.010661706527777672)
~~~~

#### What it means

Our data's p-value turned out to be 0.009. Which is to say, assuming that the null hypothesis is true ($$H_0$$ being that A and B have the same  mean; i.e. versions A and B are equally effective at getting ad donations), then there is a $$0.009$$ chance of seeing means as far apart as we did (200 vs. 220).  

Do we reject the null hypothesis, then? That depends on how ok we are in this setting making Type I errors (incorrectly rejecting the null hypothesis). If our rejection threshold is as high as $$\alpha=0.99$$, meaning we accept the risk of Type I error 1% of the time, then we reject the null hypothesis, thereby deciding that "B is more effective than A and we should go with it".




<!--

So, for our NHST here, how do we turn this complicated model into a reject / don't-reject decision function? As always, we need a single number that encapsulates the relevant model behavior, a *statistic*. So, said relevant model behavior here being the difference between the two means of $$A$$ and $$B$$, (and this is what some other people have figured out) we choose to look at the *difference* between the two means, call it $$\Delta AB = \mu_A - \mu_B$$. This is our test statistic, and since our model gives us distributions on $$A$$ and $$B$$ (binomial distributions, illustrated above), the sum (or difference) of these variables is expressible also as a known distribution.

We will choose as our null hypothesis the state of knowledge that seems most plausible / least interesting in the case of A/B testing: that versions A and B are equally effective. (Similarly, this is like saying that the least interesting outcome is that they're equally effective). Which is to say, $$\mu_A = \mu_B$$, so the difference $$\Delta AB$$ is a bell-shaped curve, centered on 0.

Here are the two key facts in deducing the distribution of $$\Delta AB$$:

* A binomial distribution $$B(n, p)$$ is almost exactly approximated by $$N(\mu=np, \sigma^2=np(1-p))$$ for n larger than 20-ish.
* The sum of two binomial distributions is $$N(\mu_1, \sigma_1^2) \pm N(\mu_2, \sigma_2^2) = N(\mu_1 \pm \mu_2, \sigma_1^2 + \sigma_2^2) $$.

So from this, we transform our null hypothesis $$H_0$$ into its equivalent *null distribution*:

|Variable| meaning | Value (from our particular data) |
| ---    | ----    | ---          |
|$$n_A$$ | number of attack ad views | 550 |
|$$n_B$$ | number of positive ad views | 550 |
|$$k_A$$ | number of attack ad donations donations | 200 |
|$$k_B$$ | number of positive ad donations | 220 |
|$$p_A$$ | probability of attack ad viewer donating | (approximation only!) $$\frac{k_A}{n_A}=0.36$$ |
|$$p_B$$ | probability of positive ad viewer donating | (approximation only!) $$\frac{k_B}{n_B}=0.44$$|



$$
\begin{align}
A &\sim N(n_A p_A, \frac{p_A (1-p_A)}{n_A})\\
B &\sim N(n_B p_B, \frac{p_B (1-p_B)}{n_B})\\
\rightarrow A - B &\sim \mathrm{N}(0, \frac{p_A (1-p_A)}{n_A} + \frac{p_B (1-p_B)}{n_B})
\end{align}
$$

$$A - B$$ is, crucially, a *single* distribution, and is essentially another (though approximate) representation of our data. It is upon $$A-B$$ that we perform hypothesis tests, below.

[illustration-plot of $$A-B$$]

### student's t-test
Named after a mathematician who went by the pseudonym "student" (styled all lower-case), the student's t-test statistic applies to normally-distributed data where both $$\mu_s$$ and $$\sigma_s$$ are **unknown**.

It's worth noting that while we don't actually know the true $$p_A$$ or $$p_B$$ for our A/B test problem, we can estimate them with the sample means, $$\bar{A} = \frac{k_A}{n_A}=0.36$$ and $$\bar{B} = \frac{k_B}{n_B}=0.44$$.

| thing | variable name | how we calculate | resulting numerical value |
| - | - | - |
| sample mean | $$\overline{X} $$ | $$\overline{A - B} $$ | = $$220 - 200 = 20$$ |
| sample variance | $$s^2$$ | $$n_A p_A (1-p_A) + n_B p_B (1-p_B)$$ | $$s^2 = 259 \\ \sigma = \sqrt{s^2} = 16.01$$ |
| population size | $$n$$ | $$n_{A-B} = n_A + n_B $$ | 550 + 550 = 1100 |  
| Test statistic | $$t$$  | $$\frac{\overline{X} - \mu_0}{s^2/\sqrt{n}}$$ | $$\frac{20 - 0}{16.01 /\sqrt{1100}}=2.56$$ |



a *comment* -->


#### References:
Peter Wills, 2019 blog post: http://www.pwills.com/posts/2019/08/24/stats.html

Chris Stucchio, 2015: https://cdn2.hubspot.net/hubfs/310840/VWO_SmartStats_technical_whitepaper.pdf

David Robinson, 2013 Stack Exchange post: https://stats.stackexchange.com/questions/47771/what-is-the-intuition-behind-beta-distribution/47782#47782

AB testing NHST https://byrony.github.io/understanding-ab-testing-and-statistics-behind.html
