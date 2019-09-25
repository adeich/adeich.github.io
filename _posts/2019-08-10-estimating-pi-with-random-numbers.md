---
title: "A (Simulated) Rain Gauge For Estimating π"
date:   2019-08-10
published: true
sidebar: toc
layout: post
permalink: "/estimating-pi-with-random-numbers"
thumb: ""
excerpt: "For amusement, I propose a rain collection device from which π can be estimated by the quantities of water it collects, and I simulate the falling rain with datasets of random numbers. The pain purpose of this article is to demonstrate common frequentist and bayesian analysis methods for confidence interval estimates on the generated dataset."
---

### Let's be clear: this is not an elegant or quick way to compute $$\pi$$

For example, to efficiently calculate $$\pi$$'s numerical value with limitless precision, there's a neat application from calculus: The [Taylor expansion](https://en.wikipedia.org/wiki/Taylor_series) of arctan is

$$\mathrm{arctan}(x) = x - \frac{x^3}{3} + \frac{x^5}{5} - \frac{x^7}{7} + \cdot\cdot\cdot$$

Check out the [proof of this series on math.stackexchange](https://math.stackexchange.com/questions/29649/why-is-arctanx-x-x3-3x5-5-x7-7-dots). So, since $$\mathrm{arctan}(1)= \pi/4$$, then $$\pi=4 \times \mathrm{arctan}(1)$$, or

$$\pi = 4 \times ( 1 - \frac{1}{3} + \frac{1}{5} - \frac{1}{7} + \cdot\cdot\cdot)$$

The following Python code uses the above formula to generate the first 5 digits of $$\pi$$ in about 3 seconds on my laptop:

~~~ python
magnitude = 6
total = 0

for i in range(10**magnitude):
    total += (-1)**(i%2) * 1./(2*i + 1)
    if i in [10**x for x in range(magnitude)]:
        print('{}: {}'.format(i, 4 * total))

>>>
        1: 2.66666666667
        10: 3.23231580941
        100: 3.15149340107
        1000: 3.14259165434
        10000: 3.14169264359
        100000: 3.14160265349
~~~

Interestingly, while this is fast for the first few digits of $$\pi$$, each subsequent digit takes exponentially longer to compute---getting the 10th digit would take my laptop several weeks. However, people have long been thinking about [clever ways to compute $$\pi$$](https://en.wikipedia.org/wiki/Approximations_of_%CF%80) and they have been incredibly successful (527 digits of $$\pi$$ calculated *by hand* (!!) in 1873; the current, computer-aided record is ~22 trillion digits in 2019).

But on to the rain collector.

### Estimating $$\pi$$ as a ratio of areas

Consider a circle inscribed in a square.

![circle square](\images\pi_circle_square.png)

The ratio $$q$$ of the area of the circle to that of the square is

$$
\begin{align}
q &= A_\mathrm{circle}/A_\mathrm{square} \\
     &= \pi r^2 / 4 r^2 \\
     &= \pi/4
\end{align}
$$

or, in terms of *q*, $$\pi=4q$$.

Now let's numerically simulate *q* by piling up lots of uniform, random points (x, y) (raindrops) in the rectangle and count how many of these fall inside the circle. The condition for checking whether a point $$(x, y)$$ is inside a circle of radius 1 is $$x^2 + y^2 < 1$$.

~~~ python
from scipy.stats import uniform
fig, ax = plt.subplots(1, 1)

circle = plt.Circle((1, 1), radius=1, facecolor="None",
                   edgecolor='k')
square = plt.Rectangle((0, 0), 2, 2, facecolor="None",
                      edgecolor='k')
plt.gca().add_patch(circle); plt.gca().add_patch(square)

a = uniform.rvs(size=(1000, 2)) * 2 - 1
inside_circle = np.where(np.sqrt(a[:,0]**2 + a[:,1]**2) < 1 )
outside_circle = np.where(np.sqrt(a[:,0]**2 + a[:,1]**2) > 1 )

plt.scatter(a[inside_circle].T[0] + 1, a[inside_circle].T[1] + 1, s=2,
            marker='o', c='blue', alpha=0.7)
plt.scatter(a[outside_circle].T[0] + 1, a[outside_circle].T[1] + 1, s=2,
            c='k', alpha=0.9)

plt.axis('scaled'); plt.axis('off')
~~~

![circle square](\images\pi_circle_square_rain.png){: .align-center}

For 10,000 raindrops, here, then, is an estimate of $$\pi$$:

~~~ python
def generate_ratio(n):
    a = uniform.rvs(size=(n, 2)) * 2 - 1
    inside_circle = np.where(np.sqrt(a[:,0]**2 + a[:,1]**2) < 1 )
    return len(inside_circle[0])/float(n)

print(4 * generate_ratio(10000))

>>> 3.146
~~~

It works!

### How many samples do we need?

Let's see how the estimation of $$\pi$$ varies as the number of samples goes from 1 to 10,000:

~~~ python
from collections import OrderedDict

N = 10000
v_generate_ratio = np.vectorize(generate_ratio)
n_range = np.arange(1, N+1)
many_n = v_generate_ratio(n_range) #this takes a few seconds for N=10000.

fig, ax = plt.subplots(1, 1)
fig.set_figheight(5)
fig.set_figwidth(8)

ax.scatter(n_range, 4 * many_n, marker='.', alpha=0.05)
ax.set_ylim(np.pi + 0.2, np.pi - 0.2)
ax.hlines([np.pi], 0, len(n_range), linestyle='--',
           label='$\pi$, exact value')
for sign in [-1, 1]:
    ax.plot(n_range, np.pi + sign*1/np.sqrt(n_range),
            linestyle='--', color='red', alpha=0.7,
           label='$\sigma = \pm 1/\sqrt{n}$')
    ax.plot(n_range, np.pi + sign*2/np.sqrt(n_range),
            linestyle='--', color='blue', alpha=0.7,
           label='$2\sigma = \pm 2/\sqrt{n}$')

ax.set_xlabel('Sample size (number of raindrops)')
ax.set_ylabel('Random estimate of $\pi$')
ax.set_title('Estimate of $\pi$ as a function of sample size')


handles, labels = plt.gca().get_legend_handles_labels()
by_label = OrderedDict(zip(labels, handles))
ax.legend(by_label.values(), by_label.keys())
~~~

![sdfs](\images\pi_as_sample_size.png)

I've also plotted the $$1\sigma$$ and $$2\sigma$$ lines predicted by the central limit theorem. The lines appear to be doing a good job approximating the 67% and 95% confidence intervals for the estimates of $$\pi$$.

### Generating a dataset for predicting confidence intervals

Say we didn't actually know the value of $$\pi$$. Let's generate a dataset of relatively small sample size and estimate some confidence intervals, using our (secret) knowledge of the true value of $$\pi$$ as a reality check:

~~~ python
n, N = 100, 1000
data = 4 * np.array([v_generate_ratio(N) for i in range(n)])
print(data)

>>>
[3.12  3.132 3.124 3.104 3.132 3.076 3.184 3.112 3.128 3.22  3.224 3.144
 3.148 3.148 3.116 3.056 3.096 3.164 3.236 3.136 3.256 3.12  3.132 3.156
 3.188 3.216 3.084 3.236 3.144 3.056 3.164 3.076 3.14  3.136 3.16  3.14
 3.048 3.184 3.244 3.168 3.216 3.088 3.108 3.204 3.2   3.128 3.028 3.184
 3.176 3.112 3.08  3.152 3.148 3.228 3.112 3.152 3.124 3.144 3.156 3.16
 3.04  3.156 3.16  3.088 3.1   3.252 3.116 3.112 3.152 3.112 3.18  3.152
 3.196 3.156 3.152 3.072 3.088 3.092 3.248 3.104 3.084 3.184 3.224 3.16
 3.148 3.176 3.2   3.14  3.128 3.136 3.124 3.168 3.084 3.092 3.164 3.176
 3.12  3.14  3.136 3.084]
~~~

Here's a histogram of this data:

![hist](/images/pi_hist_dataset.png){: .align-center}


If we just look at the sample mean and standard deviation here, they do a decent job:

~~~ python
print("""
      pi_true = {0:.3f}
      pi_est  = {1:.3f} +/- {2:.3f} (predicted by CLT)
      """.format(np.pi, data.mean(), 1/np.sqrt(N), N))

>>>   pi_true = 3.142
      pi_est  = 3.143 +/- 0.032 (predicted by CLT)
~~~

This looks pretty good, yet it turns out we can improve by an order of magnitude the accuracy of our estimate of $$\pi$$ and our confidence interval for it.

### Bayesian method: Markov Chain Monte Carlo
The following code is adapted from Jake VanderPlas' [illuminating article](http://jakevdp.github.io/blog/2014/03/11/frequentism-and-bayesianism-a-practical-intro/) about frequentist-vs-Bayesian statistics in Python.

~~~ python
import emcee

def log_prior(theta):
    return 1  # flat prior

def log_likelihood(theta, data):
    e = 1/np.sqrt(N) # CLT error.
    return -0.5 * np.sum(np.log(2 * np.pi * e ** 2)
                         + (data - theta[0]) ** 2 / e ** 2)

def log_posterior(theta, data):
    return log_prior(theta) + log_likelihood(theta, data)

ndim = 1  # number of parameters in the model
nwalkers = 50  # number of MCMC walkers
nburn = 1000  # "burn-in" period to let chains stabilize
nsteps = 2000  # number of MCMC steps to take

# we'll start at random locations between 0 and 2000
starting_guesses = 2000 * np.random.rand(nwalkers, ndim)

import emcee
sampler = emcee.EnsembleSampler(nwalkers, ndim, log_posterior, args=[data])
sampler.run_mcmc(starting_guesses, nsteps)

sample = sampler.chain  # shape = (nwalkers, nsteps, ndim)
sample = sampler.chain[:, nburn:, :].ravel()  # discard burn-in points
~~~

Here's a histogram of the output of the MCMC sampler:

~~~ python
# plot a histogram of the sample
plt.hist(sample, bins=50, histtype="stepfilled",
         alpha=0.3, density=True)

# plot a best-fit Gaussian
F_fit = np.linspace(sample.mean() - 3*sample.std(),
                    sample.mean() + 3*sample.std(), 50)

plt.plot(F_fit, norm.pdf(F_fit, loc=sample.mean(), scale=sample.std()), '-k')
plt.xlabel("sample mean"); plt.ylabel("P(mean)")


print("""
      pi_true = {0:.4f}
      pi_est  = {1:.4f} +/- {2:.3f}
      """.format(np.pi, sample.mean(), sample.std()))
~~~

~~~
>>> pi_true = 3.1416
    pi_est  = 3.1413 +/- 0.003
~~~

![mcmc](/images/pi_mcmc_hist.png){: .align-center}

The precision increased by a factor of 10!

### Bootstrapping on the mean of the dataset.

~~~ python
from astropy.stats import bootstrap

boot_result = bootstrap(data, bootfunc=np.mean)
plt.hist(boot_result, bins=30, alpha=0.3, density=True)
x = np.linspace(boot_result.mean() - 3*boot_result.std(),
               boot_result.mean() + 3*boot_result.std(), 30)
plt.plot(x, norm.pdf(x, loc=boot_result.mean(), scale=boot_result.std()))
plt.title("Distribution of mean of boostrapped data")

print("""
      pi_true = {0:.4f}
      pi_est  = {1:.4f} +/- {2:.3f}
      """.format(np.pi, boot_result.mean(), boot_result.std()))
~~~

~~~
>>>   pi_true = 3.1416
      pi_est  = 3.1410 +/- 0.004
~~~

![boostrap](/images/pi_boostrap_hist.png){: .align-center}