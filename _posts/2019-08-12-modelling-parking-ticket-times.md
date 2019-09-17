---
title: "How Safe Is It To Park Over The Time Limit?"
date:   2019-08-12
published: true
sidebar: toc
layout: post
excerpt: "Applying a statistical model with a Monte Carlo simulation, I quantify the chance of getting a ticket over time, and confirm that, indeed, you are safe (< 1% chance) parking a few minutes over the time limit. Surprisingly, the randomness of visits, assuming fixed mean time, doesn't largely affect ticketing probability."
thumb: "/images/parking_thumb.jpg"
---
**A statistical model, a Monte Carlo Simulation, and a recommendation about safe it is to overpark.**

![aviators](/images/parking_cartoon1.jpg)
*The illegally parked: shiver in your boots!*

In my neighborhood in San Francisco, there is a two-hour time limit for unmetered, free parking---one of many stressful aspects of life in the city. When I think of everyone else using the parking spots, I feel grateful to the parking enforcement for keeping the streets working healthily. But when I think of my own parking needs, I feel stressed and anxious and competitive. I watch the little three-wheeled parking enforcement vehicles glide around the streets, hunting with cold, unyielding diligence the hapless, the illegally parked.

And until a recent, earth-shattering realization, I have fastidiously moved my car before the exact, 2-hour limit, for fear of yet another $70 ticket. I have received three over the last year (one due to forgetfulness; two to accidentally parking on street cleaning day).

That is, until a month ago, when a friend, who is a math teacher and generally thoughtful, pointed out that the ticketing odds right at the two-hour mark are exactly zero. And we wondered, further, what might be the probability of getting a ticket several minutes after the limit passes. Of that question, the following computer simulation was born.

To predict the actual chance of getting a ticket if you park over the limit by, say, 1 minute, or a 10 minutes, I've built a probabilistic model of ticket vehicle visits, and, through numerical simulation, made predictions about ticketing likelihood as a function of time. Results are are summarized at the end of this article.



### The order of events leading to a ticket

To receive a parking ticket, the following chain of events always occurs:

* The car is parked at time $$t=0$$.
* Interval **A**: duration until first arrival of enforcement vehicle.
* The car's 2-hour timer starts.
* Interval **B**: Repeated, random arrivals, and if two hours hasn't passed since the first arrival, the process continues.
* At the first arrival after two hours, the ticket is given.

![events](/images/parking_timeline.png)

Evidently there are two windows here granting you extra time before your car gets ticketed: the PEV won't show up till *some time* after you park, and won't give you a ticket till *some time* after the actual 2-hour window has elapsed. In this light, our challenge is to describe the expected distribution of the sum of these two, extra time windows.

But modeling the total elapsed time-until-ticket turns out, however, to be a non-trivial problem. The parking enforcement vehicles (*PEV*)s do not arrive at precise, regular intervals; nor do they arrive with perfect randomness. The world is full of noise; the PEV's laps are littered with red lights; various discoveries of illegally parked cars; delivery trucks blocking the way; errant dogs, children, squirrels. The problem becomes, how do we quantify the noise within this periodicity?


Further, even in cases where the PEV is driving without a fixed route (the driver is distracted by a mind-bending audiobook, say), or if there are two or more PEVs on the same beat, there will still be *some* periodicity, due to the physical constraint that there is a minimum possible time between two visits (no parking-checker-person, no matter how incredible, could check twice time in the same second). Our model must cover all of these cases.

Let's illustrate what these two extremes of PEV-visit noisiness look like:

{% capture notice-2 %}
**Perfectly periodic** visits over a 120-minute window (mean period $$\mu = 12$$):

![CDF for long times](/images/parking_periodic.png 'title'){: .align-center}
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>


{% capture notice-3 %}
**Perfectly non-periodic** visits (drawn from a uniform distribution with again $$\mu=12$$):

![CDF for long times](/images/parking_non_periodic.png 'title'){: .align-center}
{% endcapture %}
<div class="notice">{{ notice-3 | markdownify }}</div>


This latter distribution shows up everywhere in nature: any non-correlated i.i.d.[^2] events, such as photons from constant-intensity source hitting a detector, or rain drops striking a tin roof, or births happening around the world.

This perfectly random case is called a Poisson process, about which we know these two distributions:
1. The probability density function (pdf) for the time between arrivals is drawn from a negative exponential distribution,
$$f(t) = \lambda e^{-\lambda t}$$, where the mean arrival time is $$\mu = 1/\lambda$$.
2. The probability of observing $$N$$ events during $$\mu$$ interval-per-event , or, equivalently, $$\lambda$$ events-per-interval (because $$\mu=1/\lambda$$) is $$P(n=N)=e^{-\lambda}\frac{\lambda^n}{n!}$$.  

For this simulation I am using property (1), summing up the time between arrivals, though it would also work to have designed the simulation around (2); the latter would require a different set of physical models about the PEV lap.



### Modeling interval **A**: from parking the car till first PEV arrival.
We are going to assume that the parking enforcement vehicle (PEV) arrives, on average, every $$\mu$$ minutes---this is the mean wait time. In my simulation, I assume $$\mu=30$$ minutes for the 2-hour parking limit.

Empirically, we can measure $$\mu$$ unambiguously by sitting on the sidewalk beneath a tree and counting the number of visits $$N$$ the PEV comes by during time interval $$\Delta T$$. Then $$\mu = \Delta T / N$$.

If the PEV is traveling with perfect periodicity of $$T=\mu$$, then the time until its first arrival, $$\lambda_0$$, is randomly drawn from a uniform distribution $$\lambda_0 \sim \mathrm{uniform}[0, \mu]$$, since the phase of the parking vehicle's loop is uncorrelated with your own arrival:

~~~~ python
from scipy.stats import expon
import matplotlib.pyplot as plt
fig, ax = plt.subplots(1, 1)
x = np.linspace(0, 50, 100)
ax.set_ylabel('probability density')
ax.set_xlabel('Time till next arrival (minutes)')
ax.set_title('Uniform distribution, $\mu$ = 30 minutes')
ax.plot(x, uniform.pdf(x),
       'b-', lw=3, alpha=0.6)
plt.show()
~~~~


![parking time](/images/parking_uniform.png){: .align-center}

If, however, the PEV instead arrives with total non-periodicity (that is, with perfect randomness), then its arrival time is given by a negative exponential distribution, $$\lambda_0 \sim \lambda e^{-\lambda t}$$ = $$\mathrm{exponential}(1/\mu)$$, where $$\lambda = 1/\mu$$:


~~~~ python
from scipy.stats import expon
import matplotlib.pyplot as plt
fig, ax = plt.subplots(1, 1)
scale = 30.
x = np.linspace(expon.ppf(0.01, scale=scale),
                expon.ppf(0.99, scale=scale), 100)
ax.set_ylabel('probability density')
ax.set_xlabel('Time till next arrival (minutes)')
ax.set_title('Exponential distribution, $\mu$ = 30 minutes')
ax.plot(x, expon.pdf(x, scale=scale),
       'r-', lw=3, alpha=0.6, label='expon pdf')
~~~~

![parking time](/images/parking_exponential.png){: .align-center}
Interestingly, the exponential distribution says there's no maximum time between visits---if it's perfectly random, the PEV could take an indefinitely long time, though the likelihood of this shrinks towards zero.


The combination of these two behaviors---perfectly periodic and perfectly non-periodic---we capture with a weighted sum between the two, where $$(\sigma/\mu)$$ is the weight:

 $$\mathrm{Interval \,\,A} \sim (\sigma/\mu) (\mathrm{uniform}[0, \mu]) + (1 - \sigma/\mu) (\mathrm{exponential}(1/\mu))$$

It's important to note that in this model, $$0< \sigma \leq \mu$$, so the weight $$(\sigma/\mu)$$ will vary over (0, 1]. The noisiness parameter, $$\sigma$$, is introduced in the next section.

### Modeling interval **B**: repeated subsequent arrival intervals until ticket
Subsequent arrival lap-times are described well by the [Gamma distribution](https://en.wikipedia.org/wiki/Gamma_distribution):

$$
\begin{align}
\lambda_i &= \mathrm{gamma}(a, \mu, \sigma) \\
 &= \frac{1}{\Gamma(k) \theta^k} x^{k - 1} e^{-\frac{x}{\theta}}.
 \end{align} $$

 The gamma distribution applies to processes that are composed of the sum of a large number $$(n \gtrapprox 50)$$ of subsequent, exponentially-drawn events. Which could describe the 50-ish individual segments the PEV must cover in each lap, each of which takes an exponentially-drawn amount of time to pass through (roughly).


~~~~ python
from scipy.stats import gamma
import matplotlib.pyplot as plt
fig, ax = plt.subplots(1, 1)

mean = 30.0
sigma = 5

x = np.linspace(0, 50, 100)
ax.plot(x, gamma.pdf(x, a=(mean/sigma)**2, loc=0, scale=sigma**2 / mean), 'k-', lw=2)

plt.show()
~~~~

![gamma single](/images/parking_gamma_single.png){: .align-center}


The gamma distribution is a sensible choice for this scenario because we can adjust both its mean, which represents the mean period of visits (and which we're keeping in this simulation at a constant $$\mu=30$$ minutes), *and* its standard deviation, $$\sigma$$. Here's what the gamma distribution looks like for various ratios of $$(\sigma/\mu)$$:

![gamma](/images/parking_gamma.png){: .align-center}
*For an explanation of these `a` and `scale` formulae, see the Implementation Details section, below. For the full code used to make this plot, see the [jupyter notebook](https://github.com/adeich/little_sci_comp_projects/blob/master/Modeling%20getting%20a%20parking%20ticket.ipynb).*

So as $$(\sigma/\mu)$$ approaches 0, the gamma function approaches a spike at $$\mu$$---this is just perfectly periodic visit times! Likewise, as $$(\sigma/\mu)$$ approaches 1, the gamma function approaches the exponential distribution---this just becomes, then, the perfectly random visit times discussed in the Interval *A* section, above.

Now, with our two parameters $$\mu$$ and $$\sigma$$, we can quantify the amount of randomness vs periodicity, and explore how the ratio affects the time-likelihood dependence for getting a ticket.


### Putting it all together into a Monte Carlo simulation

For each parking ticket trial, a single time is drawn for interval *A*, as described above. Then, for interval *B*, we keep adding times drawn from the associated gamma distribution to the total elapsed time until two-hours is exceeded, at which point we record the total elapsed time.

Or, to express the form of the Monte Carlo simulation in pseudocode:
{% highlight guess_lang %}
N := 10,000
data := array(length=N)
for n going from 1 to N:
  sum := generate_t0() # randomly drawn initial time
  while sum < 120: # minutes
    sum := sum + generate_t_i() # randomly drawn subsequent time
  data[n] := sum   
{% endhighlight %}

For the full code of the simulation function, see `generate_data()` in the [jupyter notebook](https://github.com/adeich/little_sci_comp_projects/blob/master/Modeling%20getting%20a%20parking%20ticket.ipynb).

I found that the histogram (which will begin to approximate the true [probability density function](https://en.wikipedia.org/wiki/Probability_density_function) (pdf) as $$N$$  gets big, according to the [Law of Large Numbers](https://en.wikipedia.org/wiki/Law_of_large_numbers)), starts to stabilize around $$N > \mathrm{10,000}$$.

Here's what a histogram of this simulation's typical output looks like:

![CDF for long times](/images/parking_hist_example.png 'title'){: .align-center}
Though, as discussed in the results section, below, this is not as useful a representation of the data for our purposes as a cumulative distribution function (CDF) view.

## Implementation details
The Jupyter notebook containing all the code and outputs can be found in this [GitHub repository](https://github.com/adeich/little_sci_comp_projects/blob/master/Modeling%20getting%20a%20parking%20ticket.ipynb).

### Inability to vectorize the arrival time loops
It would have been great to vectorize[^3] all these sums of random numbers, but because there is the piece-wise condition on the sum, ('add until 2-hours is exceeded'), we need to add subsequent random numbers within a `while` loop. Because the numbers are random, there is not a fixed-length array of numbers to add---sometimes it will be few; sometimes many. There are faster ways to implement this while-loop in Python, but with time constraints I implemented my `while` loop somewhat naively, having to manually keep track of several array indices.

### Connecting standard Gamma form to that of `scipy.stats.gamma()`

A difficulty in using the `scipy.stats.gamma()` function here is that its arguments don't match up with our variables $$\mu$$ and $$\sigma$$. I derived a conversion formula, as follows.

In [scipy.stats.gamma](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.gamma.html), the gamma function is called by `gamma.pdf(x, a, loc, scale)`, where `x` is an array of locations ("what is the probability density at a given *x*?"); `a` is a special 'shape' descriptor that does **not** equal $$\sigma$$; `loc` is the origin, always `0` here; and `scale` parameter is, importantly, different than the standard deviation of the gamma distribution.  

Whereas, the standard form of the gamma pdf (see the [wikipedia article](https://en.wikipedia.org/wiki/Gamma_distribution)) is

$$\begin{align}
\mathrm{PDF_{wikipedia}}(x) &= \frac{1}{\Gamma(k)\theta^k} x^{k-1} e^{-x/\theta},\\
\end{align}$$

where mean $$\mu=k\theta$$ and variance $$\sigma^2=k\theta^2$$.

Let's translate the wikipedia-standard-form arguments into the `scipy` arguments. Here's the `scipy` arguments, plugged into the wikipedia form:

$$
\mathrm{PDF_{scipy}}(x) = \frac{x^{a-1} e^{-x}}{\Gamma(a)}.
$$

The `scipy` docs give us a transformation: `gamma.pdf(x, a, loc, scale)` is identically equivalent to `gamma.pdf(y, a) / scale` with `y = (x - loc) / scale`. Plugging these in to the previous equation gives

$$\mathrm{PDF_{scipy}}(x, a, \mathrm{loc}, \mathrm{scale}) = \frac{1}{\mathrm{scale}} \frac{(\frac{(x-\mathrm{loc})}{\mathrm{scale}})^{a-1} e^{-(\frac{(x-\mathrm{loc})}{\mathrm{scale}})}}{\Gamma(a)}$$

Comparing these two forms, we see that

$$
\begin{align}
k &\rightarrow a\\
x_{\mathrm{wikipedia}} &\rightarrow x_{\mathrm{scipy}}-\mathrm{loc}\\
\theta &\rightarrow \mathrm{scale}.
\end{align}
$$

I want to figure out how to substitute the scipy arguments for the equivalent wikipedia measurements of variance $$\sigma^2$$ and mean $$\mu$$; comparing, I see that

$$
\begin{align}
\mathrm{mean} &= \theta k \\
& \rightarrow \mathrm{scale}\cdot a
\end{align}
$$

and

$$
\begin{align}
\sigma &=  \theta\sqrt{k} \\
&\rightarrow \mathrm{scale} \sqrt{a}.
\end{align}
$$

Finally, solving these two equations for `scale` and `a` we get:

$$
\begin{align}
\mathrm{scale} &= \frac{\sigma^2}{\mu};\\
a &= \big{(}\frac{\mu}{\sigma}\big{)}^2.
\end{align}
$$

So, for a given mean $$\mu$$ and variance $$\sigma^2$$, our actual function call will look like

~~~~ python
from scipy.stats import gamma
x = np.linspace(0,60,100)
mean = 30
sigma = 10

gamma.pdf(x, a=(mean/sigma)**2, loc=0, scale=sigma**2/mean)
# outputs a PDF array
~~~~

## Simulation results


For visualizing the probability of getting a ticket as a function of how long we're over-parked, here is a plot of the CDF (cumulative distribution function), which tells you the probability (expressed as a percentage) of getting a ticket after given number of minutes after the two-hour mark. Starting from a 1-d array listing simulated ticket times, I've used `numpy.percentile()` to generate the CDF.

I've plotted several lines for different values of $$\sigma$$, which represents the noisiness away from perfect periodicity by the PEV.


![CDF for long times](/images/parking_CDF_plot_1.png 'title'){: .align-center}

 Above, $$(\sigma/\mu)$$ is varied from 0.15 to 1, and there is some varying effect on the CDF, though only for probabilities above ~25%.

 Next, let's also look at the behaviors for very small values of $$(\sigma/\mu)$$:


![CDF for short times](/images/parking_CDF_plot_2.png 'title'){: .align-center}

Though I expect this precision of periodicity is unlikely in real life, it's very interesting to see the $$\sigma$$-dependent behavior that emerges for small times. I believe this has to do with a sort of aliasing effect.

## Results discussion and big-picture takeaways
Interestingly, up until about 25% ticket probability, there is very little variation, and hence $$\sigma$$-dependency, in time over-parked! This means that *regardless* of where $$(\sigma/\mu)$$ (the noisiness parameter) falls on $$(0, 1]$$, the resulting CDF ends up being roughly the same.


So, for a 2-hour time limit, I conclude the following rules of thumb. These assume that the parking checkers come by every 30 minutes and have a moderate imprecision to how consistent they keep this periodicity. Depending on noisiness, these values will vary only by about 10%, which reflects a reassuring stability of these predictions.



|Probability of ticket | Equivalently, odds of ticket | Parking overstay (approximate) |
|--| ----------- | ----------- |
|0.01| 1 in 100 | 5 minutes |
|0.05| 1 in 20  | 11 minutes |
|0.10| 1 in 10  | 16 minutes |
|0.25| 1 in 4   | 25 minutes |
|0.50| 1 in 2   | 40 minutes |
|0.75| 3 in 4   | 70 minutes |





In reality, parking checkers may come far more or less frequently than 30 minutes; my results scale linearly with your observed measurements of $$\mu$$ and $$\sigma$$.

Another useful way of thinking about the cost of parking ticket is to multiply the chance of getting a ticket by the cost of a ticket. For example, if there is a 1% ticketing chance for staying 5 minutes over, and a ticket costs $70, then you are essentially paying 1% $$\times$$ $70 = $0.70 whenever you park this much over the limit.

#### Future Directions
One day I would like to set some chairs in the shade in front of my apartment and sit there with friends and actually record data on the passage of these majestic parking enforcement vehicles. Then, the puzzle of actually matching $$\sigma$$ to the data, including consideration of confidence intervals, would be a very interesting investigation.

#### Epilogue
This post is dedicated to the people who drive these three-wheelers---we owe the fairness and availability of public parking in crowded cities to their hard work.

![parking vehicle](/images/parking_metervehicle.png 'title'){: .align-center}


### References and further reading
I recommend reading the [bus arrival time simulation blog post](https://jakevdp.github.io/blog/2018/09/13/waiting-time-paradox/) by Jake VanderPlas in 2018.


[^2]: Independent and identically-distributed random variable [wikipedia article](https://en.wikipedia.org/wiki/Independent_and_identically_distributed_random_variables).
[^3]: Here's a good [vectorization tutorial](https://jakevdp.github.io/PythonDataScienceHandbook/03.10-working-with-strings.html) by Jake VanderPlas.
