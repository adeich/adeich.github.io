---
layout: post 
title: "How safe is it to park over the time limit?"
date:   2019-08-12 
published: true
sidebar: toc
---

### Summary of ticketing predictions 
For a 2-hour time limit, assuming the parking checkers come by every 30 minutes and have a moderate imprecision to how consistent they keep this periodicity. Depending on model parameters, these values vary only by about 10%. 

|Probability of ticket | Odds of ticket |Overstay|
|--| ----------- | ----------- |
|0.01| 1 in 100 | 4 minutes | 
|0.05| 1 in 20  | 10 minutes |
|0.10| 1 in 10  | 15 minutes |
|0.25| 1 in 4   | 25 minutes |
|0.50| 1 in 2   | 45 minutes |
|0.75| 3 in 4   | 75 minutes |

In reality, parking checkers may come far more or less frequently; scale these numbers accordingly. For explanation and interesting subtleties of this prediction, see below.

### Background for car-free readers 

I often park on streets in San Francisco that have a 2-hour time limit. I have tended to be very careful to come retrieve the car by exactly the 2-hour mark, because there's a $70 ticket (the same order of magnitude as a typical day's wages), and I've so far suffered several of them due to occasional forgetfullness. 

But I have recently realized, thanks to a conversation with a friend, that the odds of getting a ticket even just a few minutes outside the 2 hour window are slim. Here, I've modelled just how risky it is to park over the time limit by various amounts, and how this depends on the behavior of the parking enforcement professionals.

![parking vehicle](/images/metervehicle.png 'title'){: .align-center}



In order for you to get a ticket after you leave your car, two events need to occur, in this sequence:

1. At some time after you've parked, a parking-rule checker first arrives, recording your car's presence.
2. A checker (not necessarily the same one; I suspect they can communicate with each other!) arrives again, at least 2-hours after their first arrival, at which time they give the ticket.


