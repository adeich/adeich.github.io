---
title: "How Safe Is It To Park Over The Time Limit?"
date:   2019-08-12
published: true
sidebar: toc
layout: post
---
**A Monte Carlo Simulation that looks at parameter dependence on the outcomes.**

THE CONVERSATION THAT BEGAT THIS STUDY

>Me: I have to run and move my car, my two-hours is nearly up.
>
>Roommate, who is a math teacher: No hurry. There's basically zero chance of getting a ticket right after the two-hour mark.
>
>Me: What do you think that chance will be after ten minutes?
>
>Roommate and me: Hmm ...


## The physical scenario

### Parking in two-hour zones: a little stressful.

Around my neighborhood in San Francisco, most parking spots have a two-hour time limit, enforced by little three-wheeled vehicles, keeping the local streets functioning healthily. That said, from a driver's perspective, they are a real source of stress. If they pass by your car twice, in the same spot, and if more than two hours have passed between, they leave you a stiff, $70 ticket. Up until recently I have been very careful to move my car before the *exact* two-hour mark. However, my roommate and other friends have recently pointed out to me that there are several physical effects which cause there to be zero chance of getting a ticket at precisely the two-hour mark.

To predict the actual chance of getting a ticket if you park over the limit by, say, 1 minute, or a 10 minutes, I've built a model of ticket vehicle visits, and, through numerical simulation, made some predictions about ticketing likelihood. Results are are summarized at the end of this article.




## Thinking about a model for this process

### The order of events leading to a ticket
Let's review the fundamentals here.

1. You park your car.
2. After some initial time interval, the parking person first arrives at your car.
3. Several intermediate arrivals, but if two hours hasn't passed since the first arrival, nothing changes.
4. At the first arrival after the two-hour mark, the ticket is given.

This is a non-trivial problem because the parking enforcement vehicles (*PEV*) do not arrive at precise, regular intervals.

### Periodicity, or lack thereof
The arrival of the parking enforcement vehicle (*PEV*) is highly periodic, but not perfectly so. The PEV does laps (a sensical, repeating pattern, centered on a certain frequency) yet also it drives the streets of a noisy, random world, full of varying cars illegally parked, children and dogs running out into the road, delivery truck blocking the way. So the PEV will not arrive with perfect periodicity---there will be some kind of random noise added to the baseline consistent visit intervals.

It's also interesting to think about how, even in cases where the PEV is driving without a fixed route (the driver is distracted by an mind-bending audiobook, say), or if there are two or more PEVs on the same beat, there will still be *some* periodicity, due to the physical reason of there being a minimum possible time between two visits.

Let's illustrate these two extremes of PEV visits:

{% capture notice-2 %}
**Perfectly periodic** visits over a 120-minute window (mean period $$\mu = 12$$)

![CDF for long times](/images/parking_periodic.png 'title'){: .align-center}
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>


{% capture notice-3 %}
**Perfectly non-periodic** visits (drawn from a uniform distribution with again $$\mu=12$$)

![CDF for long times](/images/parking_non_periodic.png 'title'){: .align-center}
{% endcapture %}
<div class="notice">{{ notice-3 | markdownify }}</div>

It is fascinating that to humans, this author very much included, truly random events often don't match our feelings of what randomness should look like, at least not without a lot of re-training of our intuition. To me, the random data points look like they tend to arrive in clumps; as though there's a certain intelligence behind them.

This latter distribution is very common in the natural world: any non-correlated i.i.d.[^2] events, such as photons from constant-intensity source hitting a detector, or rain drops striking a tin roof, or births happening around the world.

This perfectly random case is called a Poisson process, about which we know the following, useful properties:
1. The probability density function (pdf) for the time between arrivals is drawn from a negative exponential distribution,
$$f(t) = \lambda e^{-\lambda t}$$, where the mean arrival time is $$\mu = 1/\lambda$$.
2. The probability of observing $$N$$ events during $$\mu$$ interval-per-event , or, equivalently, $$\lambda$$ events-per-interval (because $$\mu=1/\lambda$$) is $$P(n=N)=e^{-\lambda}\frac{\lambda^n}{n!}$$.  

For this simulation I am using property (1), summing up the time between arrivals, though it would also work to have designed the simulation around (2); the latter would require a different set of physical models about the PEV lap.


A big question here is: how do we capture this quasi-periodicity with our probabilistic model?

### My proposed solution and model for the visit times.
Say that the parking enforcement vehicle (PEV) arrives, on average, every $$\mu$$ minutes---this is the mean wait time. We can all easily agree on $$\mu$$ empirically, because we just count the number of visits $$N$$ during time interval $$\Delta T$$ and then calculate $$\mu = \Delta T / N$$.

Here I describe my model of the process, including plausibility arguments.

* Time *t=0* starts at the moment you park the car.

#### Interval till first arrival
* If the PEV is traveling with perfect periodicity of $$T=\mu$$, then the time until its first arrival, $$\lambda_0$$, is randomly drawn from a uniform distribution $$\lambda_0 \sim \mathrm{uniform}[0, \mu]$$, since the phase of the parking vehicle's loop is uncorrelated with your own arrival.
* However, if the PEV arrives with total non-periodicity, then its arrival time is given by a negative exponential distribution, $$\lambda_0 \sim \lambda e^{-\lambda t}$$ = $$\mathrm{exponential}(1/\mu)$$(where $$\lambda = 1/\mu$$), as mentioned above, because there's no maximum lap time.
* The combination of these two behaviors---perfectly periodic and perfectly non-periodic---we capture with a weighted sum between the two, where $$\sigma/\mu$$ is the weight:

$$\lambda_0 \sim (\sigma/\mu) (\mathrm{uniform}[0, \mu]) + (1 - \sigma/\mu) (\mathrm{exponential}(1/\mu))$$

* It's important to note that in this model, $$0< \sigma \leq \mu$$, so this weight will vary over (0, 1].

#### All subsequent arrival intervals
* Subsequent arrival intervals are then given by the Gamma distribution:

$$\lambda_i = \mathrm{gamma}(a, \mu, \sigma),$$

* The gamma distribution applies to processes that have a large number $$(n \gtrapprox 50)$$ of subsequent, exponentially-derived events.

* We will then simply add times drawn from this distribution to our total elapsed time until two-hours is exceeded.

The monte carlo simulation thus takes the form of this simple nested loop (pseudocode):
{% highlight guess_lang %}
N := 10,000
data := array(length=N)
for n going from 1 to N:
  sum := generate_t0() # randomly drawn initial time
  while sum < 120: # minutes
    sum := sum + generate_t_i() # randomly drawn subsequent time
  data[n] := sum   
{% endhighlight %}

That is, for each iteration of the time-until-ticket simulation, we sum randomly drawn arrival times until 2 hours is exceeded; the total elapsed time is then recorded.

I found that the histogram (which will begin to approximate the true [probability density function](https://en.wikipedia.org/wiki/Probability_density_function) (pdf) as $$N$$  gets big, according to the [Law of Large Numbers](https://en.wikipedia.org/wiki/Law_of_large_numbers)), starts to stabilize around $$N > \mathrm{10,000}$$.

Here's what a histogram of this simulation's typical output looks like:

![CDF for long times](/images/parking_hist_example.png 'title'){: .align-center}
Though, as discussed in the results section, below, this is not as useful a representation of the data for our purposes as a cumulative distribution function (CDF) view.

## Implementation details
The Jupyter notebook containing all the code and outputs can be found in this [GitHub repository](https://github.com/adeich/little_sci_comp_projects/blob/master/Modeling%20getting%20a%20parking%20ticket.ipynb).

### Inability to vectorize the arrival time loops
It would have been great to vectorize (a numpy concept) all these sums of random numbers, but because there is the piece-wise condition on the sum, ('add until 2-hours is exceeded'), we need to add subsequent random numbers within a `while` loop. There are faster ways to do this, but with time constraints I implemented my `while` loop in standard python, having to manually keep track of several array indices:

### Connecting standard Gamma form to that of `scipy.stats.gamma()`
In [scipy.stats.gamma](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.gamma.html), the gamma function is called by `gamma.pdf(x, a, loc, scale)`, where `x` is an array of locations (what is the probability density at a given $$x$$?); `a` is a special 'shape' descriptor; `loc` is the origin, always `0` here; and `scale` parameter is, importantly, different than the standard deviation of the gamma distribution.  

Whereas, the standard form of the gamma pdf (see the [wikipedia article](https://en.wikipedia.org/wiki/Gamma_distribution)) is

$$\begin{align}
\mathrm{PDF_{wikipedia}}(x) &= \frac{1}{\Gamma(k)\theta^k} x^{k-1} e^{-x/\theta}\\
\end{align}$$.

My difficulty here was figuring out how to translate the wikipedia-standard-form arguments, (which are very useful because there, we know that mean=$$k\theta$$; variance=$$k\theta^2$$) into the `scipy` arguments.

Here's the `scipy` arguments, plugged into the wikipedia form:

$$
\mathrm{PDF_{scipy}}(x) = \frac{x^{a-1} e^{-x}}{\Gamma(a)}
$$.

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

So, for a given mean $$\mu$$ and variance $$\sigma^2$$, our actual function call will be

~~~
gamma(x, a=(mean/sigma)**2, loc=0, scale=sigma**2/mean)
~~~

## Simulation results


For visualizing the probability of getting a ticket as a function of how long we're over-parked, here is a plot of the CDF (cumulative distribution function), which tells you the probability (expressed as a percentage) of getting a ticket after given number of minutes after the two-hour mark.

There are different lines for different values of $$\sigma$$, which represents the noisiness away from perfect periodicity by the PEV.


![CDF for long times](/images/parking_CDF_plot_1.png 'title'){: .align-center}

 Above, $$\sigma/\mu$$ is varied from 0.15 to 1, and there is some varying effect on the CDF.


![CDF for short times](/images/parking_CDF_plot_2.png 'title'){: .align-center}
It's also interesting to look at what happens within the regime of extremely precise PEV visits $$(0.01<\sigma/\mu\leq0.15)$$, where we see widely varying CDF dependency over the range [0, 30] minutes.

## Results discussion and big-picture takeaways
Interestingly, up until about 25% ticket probability, there is very little variation, and hence $$\sigma$$-dependency, in time over-parked! This means that *regardless* of where $$\sigma/\mu$$ (the noisiness parameter) falls on $$(0, 1]$$, the resulting CDF ends up being roughly the same.


So, for a 2-hour time limit, I conclude the following rules of thumb. These assume that the parking checkers come by every 30 minutes and have a moderate imprecision to how consistent they keep this periodicity. Depending on noisiness, these values will vary only by about 10%, which reflects a reassuring stability of these predictions.



|Probability of ticket | Equivalently, odds of ticket |Middle-ish overstay|
|--| ----------- | ----------- |
|0.01| 1 in 100 | 5 minutes |
|0.05| 1 in 20  | 11 minutes |
|0.10| 1 in 10  | 16 minutes |
|0.25| 1 in 4   | 25 minutes |
|0.50| 1 in 2   | 40 minutes |
|0.75| 3 in 4   | 70 minutes |
{: .align-center}



In reality, parking checkers may come far more or less frequently than 30 minutes; my results scale linearly with your observed measurements of $$\mu$$ and $$\sigma$$.

### Future directions
One day I would like to get some friends together with some chairs in the shade in front of my apartment and actually measure the passage of these majestic parking enforcement vehicles.

This post is dedicated to the people who drive these three-wheelers---we owe the sanity of our public streets to them.



![parking vehicle](/images/parking_metervehicle.png 'title'){: .align-center}

[^1]: Parking cars is great.
[^2]: Independent and identically-distributed random variable [wikipedia article](https://en.wikipedia.org/wiki/Independent_and_identically_distributed_random_variables).
