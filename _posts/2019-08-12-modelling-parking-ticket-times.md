---
title: "How Safe Is It To Park Over The Time Limit? A Monte Carlo Simulation"
date:   2019-08-12
published: true
sidebar: toc
---
**With careful consideration of model and parameter dependence.**

## The physical scenario

### Parking in two-hour zones: a little stressful.

Around my neighborhood in San Francisco, most parking spots have a two-hour time limit, enforced by little three-wheeled vehicles, keeping the local streets functioning healthily. That said, from a driver's perspective, they are a real source of stress. If they pass by your car twice, in the same spot, and if more than two hours has passed between, they leave you a stiff, $70 ticket. Up until recently I have been very careful to move my car before the *exact* two-hour mark. However, friends have recently pointed out to me that there are several effects of chance which make the odds of getting a ticket precisely at two-hours zero.

To quantify the actual time you can park over the limit for, say, a 1% chance, or a 10% chance, I have made a simulation of ticket vehicle visits. Results are are summarized at the end.

### The order of events leading to a ticket.

1. You park your car.
2. After some initial time duration, the parking person first arrives at your car.
3. Several intermediate arrivals may occur, but if two hours hasn't passed, nothing changes.
4. At the first arrival after the two-hour mark, the ticket is given.

## Modeling the visit times.
My model is characterized entirely by two parameters: mean visit time, $$\mu$$, and $$\sigma$$, a description of how non-periodic the arrivals are.

* Time starts at *t=0* at the moment you park the car.
* The time until the first arrival is randomly drawn from a uniform distribution $$t_0 \sim \mathrm{uniform}[0, \mu]$$, since the phase of the parking vehicle's loop is uncorrelated with your own arrival.
* Subsequent arrival times are then given by

$$t_i \sim C + \mathrm{gamma}(a, \mu - C, \sigma),$$

* where $$C$$ is the minimum lap time (physically corresponding to duration of the parking enforcement traveling at full speed the whole way around the lap, without stopping to give tickets or for stop signs, etc.). The gamma distribution applies to processes that have a large number $$(n \gtrapprox 50)$$ of subsequent, exponentially-derived events.

The total time is computed thus:
~~~~
sum = t0() # randomly drawn
while sum < 120: # minutes
  sum = sum + t_i()
~~~~

For each iteration of the time-until-ticket simulation, we sum randomly drawn arrival times until 2 hours is exceeded; the total elapsed time is then recorded. I found that the histogram (which will begins to approximate the true [probability density function](https://en.wikipedia.org/wiki/Probability_density_function) (pdf) as $$N$$  gets big, according to the [Law of Large Numbers](https://en.wikipedia.org/wiki/Law_of_large_numbers)), starts to stabilize around $$N > \mathrm{10,000}$$.

Here is what a typical such histogram looks like:
![CDF for long times](/images/parking_hist_example.png 'title'){: .align-center}
Though, as discussed in the results section, below, this is not as useful a representation of the data for our purposes as a cumulative distribution function (CDF) view.

## Implementation details
The Jupyter notebook containing all the code and outputs can be found in this [GitHub repository](https://github.com/adeich/little_sci_comp_projects/blob/master/Modeling%20getting%20a%20parking%20ticket.ipynb).

### Inability to vectorize the arrival time loops
After

## Simulation results


However, this view of the data does not allow us to figure out what the odds are of getting a parking ticket at specific times of being over-parked. For that, here is a plot of the CDF (cumulative distribution function), which tells you the probability (expressed as a percentage) of getting a ticket after given number of minutes after the two-hour mark.


![CDF for long times](/images/parking_CDF_plot_1.png 'title'){: .align-center}
![CDF for short times](/images/parking_CDF_plot_2.png 'title'){: .align-center}


## Takeaways
For a 2-hour time limit, the following rules of thumb assume that the parking checkers come by every 30 minutes and have a moderate imprecision to how consistent they keep this periodicity. Depending on model parameters, these values vary only by about 10%, which reflects a reassuring stability of predictions.

|Probability of ticket | Odds of ticket |Middle-ish overstay|
|--| ----------- | ----------- |
|0.01| 1 in 100 | 5 minutes |
|0.05| 1 in 20  | 11 minutes |
|0.10| 1 in 10  | 16 minutes |
|0.25| 1 in 4   | 25 minutes |
|0.50| 1 in 2   | 40 minutes |
|0.75| 3 in 4   | 70 minutes |

In reality, parking checkers may come far more or less frequently; my results scale linearly with your observed measurements of $$\mu$$ and $$\sigma$$.


![parking vehicle](/images/parking_metervehicle.png 'title'){: .align-center}
