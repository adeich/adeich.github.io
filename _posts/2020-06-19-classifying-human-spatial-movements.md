---
title: "Classifying Customer Spatial Movement In Retail Stores"
date:   2020-06-19
published: true
sidebar: toc
layout: post
permalink: "/classifying-human-spatial-movements"
thumb: "images/classifying_thumb.png"
excerpt: "Consulting for Pathr.ai, I developed a method for separating spatial trajectories of brick-and-mortar retail stores by customer behavior."
---

For my Insight project I collaborated with [Pathr.ai](https://pathr.ai/), a start up based in Mountain View. Pathr uses computer vision to help brick-and-mortar businesses understand how people move through their physical space. To allow Pathr to begin to **classify customer spatial movements**, I established a set of **trajectory metrics** and an accompanying trajectory analysis pipeline. 



![](images/classifying_timelapse.jpeg)


# **Customer spatial analysis, now more important than ever, is becoming automated**

How many customers visit each part of the sales floor, and when? At what point do customers change their minds and decide to make a purchase? Which types of customer-staff interactions lead to higher store profits?

These are fundamental questions for anyone looking to run a competitive retail store today.

As e-commerce websites like Amazon consume the low-margin, high-inventory sales market, the brick-and-mortar retail industry shifts its focus towards high-quality customer experience. How a customer _feels_ about the physical, in-store human interactions has become a primary driver of brick-and-mortar retail profits. For stores to stay competitive today, a store's employees must understand how customers explore and interact with the sales floor.

Until now, if you wanted to gather such customer data for your store, you faced a slow and inaccurate process. If you want to estimate the fraction of visiting customers who make a purchase, say, a store would post an employee outside the front door and manually count for hours the number of customers entering the front door.

Over the last decade computer vision has been steadily improving in its accuracy and ease of implementation. All the while, virtually all brick-and-mortar sales floors are covered by high-quality security camera footage. And, until 2020, virtually no one has applied AI analysis to their data.

This is where Pathr comes in: the first to market, as of mid-2020, with an anonymized computer vision system that produces an automated heat map analysis for retail stores. And the heat map updates in real time. Moving beyond this early success, Pathr will need to add further data insights to remain competitive in this new market, as several other competitors are working to enter into this retail AI space.

# **How Pathr maps customer spatial movement and generates real time analysis**

![pipeline](images/classifying_highlevel.png){: .align-center}


Here's how Pathr's data flow works: A client store streams the video feeds from multiple cameras. Pathr then applies computer vision to locate every person in the field of view.

Each person in the store is represented by a 2D (x, y) point determined by the center of that person's AI-determined bounding box. In order to follow each person's progress through the store, the computer vision system assigns them a temporary unique ID that is discarded as soon as the person leaves the field of view.

So each trajectory here represents the movement of a single person as a list of (x, y, t) rows, recorded at 30 frames per second:

![pipeline](/images/classify_single_trajectory.png){: .align-center}

Compared to GPS geospatial data, which is accurate to ~1 meter, Pathr's system presently achieves centimeter-precision in measuring customer location. This is thanks in part to the high pixel density of the typical store's security camera system.

So in actual application, if x and y are floats with units of meters, x and y will contain meaningful position information to a depth of ~5 significant figures. There is a lot of detail in this data!


Presently, Pathr provides its retail customers with a heat-map analysis product that answers the fundamental counting question, **"how many people visit each part of the store, and when?"**

# **Pathr is working to add trajectory classification**


Meanwhile, Pathr is also working to add the ability to analyze each **individual customer trajectory**, so as to classify a customer's behavior in real time. Useful insights from such trajectory analysis include, for example,

 * distinguishing customers from staff
 * real time identification of customers who could use assistance
 * revealing precisely how customer store use patterns translate to purchasing behavior
 * detecting shoplifting events as they occur


However, classifying on raw trajectories is difficult. A generally large obstacle to establishing a functional trajectory analysis product. **Developing such a trajectory classification framework is the focus of my project.**

# **What makes classifying trajectories hard**

To restate our high-level goal, given a customer trajectory (a set of ~1000 (x, y, t) rows), we want to  quickly figure out which abstract categories it falls into. Here's are some of the primary reasons that's difficult.

First, trajectories are not fixed in length. At 30 frames-per-second, each trajectory accrues an additional 1,800 points for every minute a person is in the store. A staff member, for example, working for 2-hours, would result in measured trajectory 100,000 points long, nearly a gigabyte of data. In an enormous store with hundreds of employees, performing real time analysis becomes nearly impossible with this much data.

Second, from a machine learning perspective, each additional row (x, y, t) acts as an additional 3 dimensions, so dimensionality increases dramatically. At the same time, we don't know _a priori_ over what time- or space-scales the important information occurs. Whether it's 10 points or 10,000 points, the scale of interest will vary across different types of behaviors, as will the signal-to-noise ratio. **Our desired classification behavior should remain stable across different sample window sizes.**

![pipeline](/images/classifying_metric2.png){: .align-center}
*How an ideal classification metric should separate trajectories.*



Third, customer behaviors of interest often live deep below a lot of unrelated information and noise. For instance, how much does a trajectory's absolute location in the store matter? How much error does the computer vision system introduce? How much do a customer's small side-to-side movements tell us about their macro behaviors, or vice versa?

And do we normalize by distance and direction? It apparently varies in information-usefulness whether we measure the absolute distance a customer travels--varying needs for different behaviors. **We need to be able to capture information contained only in abstract, certain segments of each feature.** And discard the raw trajectory data as quickly as possible, because it piles up quickly.

Due to all of this stack of noise, dimension, and variable scales of interest, applying a supervised learning classifier to raw trajectory data doesn't work very well. As a basic test, I applied a random forest model to our raw simulation trajectories (each clipped to a fixed length of 500 points) and I found that the classifier could barely guess better than random.

Likewise at the other end of the abstract-to-concrete spectrum, if you apply a set of hand-selected rules to trajectories, it doesn't classify well either. For example, if you say "If the trajectory velocity spends more than 80% of the time within X velocity range, then assign it Y label," this tends to perform no better than a random guess, since the interesting distinctions arise only in a much higher dimensional space.


# **My solution: a trajectory embedding that extracts identifying information from movement**

To convert each trajectory into a form that is meaningful to a classifier, I developed a **trajectory embedding**, which is just a fixed set of roughly 50 metrics, computed on each trajectory. So the classification operation becomes: _first_ we map the trajectory through the embedding (essentially a dimensionality reduction operation), _then_ pass the embedded form to the classifier.


![pipeline](/images/classifying_metric1.png){: .align-center}



For my embedding, I focused on statistics that distill kinematic information around various rates of change. I chose the following statistics in particular because, while capturing key movement descriptions, they are independent of the trajectory's length, sample frequency, and starting location.

**Trajectory metrics that achieved proof-of-concept success:**
1. mean velocity
1. standard deviation of velocity
1. mean acceleration
1. standard deviation of acceleration
1. ratio of bounding rectangle to arc length
1. the lowest 50 fft frequencies in both x and y

After computing these, the embedded trajectories (a ~100-dimensional space) are then rotated via PCA into a space where I keep only the first ~10 principle components. This output, finally, is what I pass to the classifier.

# **A pipeline to allow rapid experimentation on metric sets**

I knew I would have to iterate through many combinations of statistics to maximize visual clustering. Further, it seems likely that Pathr will probably require custom metric sets for each new retail space they're analyzing--there is probably not a one-size-fits-all solution, at least not yet.

To address this, I built a trajectory embedding pipeline which applies a purposefully open-ended list of embedding functions, then shows how well the trajectories separate by category.

![pipeline](/images/classify_pipeline.png){: .align-center}

My pipeline, utilizing high-speed array operations with `Pandas` and `numpy`, can distill about 100MB of trajectory data into a metric set in about 30-seconds on my older laptop. I was able to try 60+ combinations drawn from 15 statistics functions in a few hours, arriving at the function set I listed, above.

Additionally, incorporating future statistics metrics into the set requires no additional pipeline engineering, as it's just a matter of defining a new function and adding its name to the list.

# **Proof-of-concept dataset analysis**

#### **#1: Shapes Data**
For an initial proof-of-concept test, Pathr and I began with a set of shape trajectories -- a simulated dataset in which someone is moving in the shape of 100 circles, 100 rectangles, and 100 triangles. While all shapes had the same orientation, they varied by location and scale.

![](/images/classifying_shapes.png)

Wielding the pipeline I described above, over the course of an hour I tried 60+ combinations of different metric functions on this dataset, visually inspecting the quality of separation in the plot shown at right. I also ran a Random Forest classifier with 5-fold cross-validation on each resulting metric set; while optimum classification accuracy wasn't my goal here, it was a useful metric to check how my embedding was doing.

The metric set I settled on achieved a prediction accuracy of 95%, using only the metrics I described above (mean velocity, mean acceleration, etc) + PCA. If I had chosen metrics aimed these shapes's specific qualities, it would have yielded a higher prediction accuracy, but I stuck only with metrics that were broadly transferrable to any kind of motion.

#### **#2: Factory Floor Simulation**

My second proof-of-concept demonstration represented a (simulated) factory floor, with 135 people moving each for 7,500 trajectory points, with 3 different behavior roles that required separation. This dataset was 2.4GB, a good test of my pipeline's memory usage.

![](/images/classifying_factory.png)

With this messier data, I applied the same metric set as with the shapes, and achieved a 75% prediction accuracy: this is a much better-than-random baseline but there's a lot of room for improvement. As you can see in the above plot, there is some clustering by label, but it's not as well separated as with the shapes trajectories.

With my trajectory metric set, there is clearly a lot of room for improvement, as I didn't apply noise filtering or deep learning.



# **Insights and future directions**

* Grab-bag of kinematics metrics + PCA works pretty well as a start.
* Initial metric set achieved high classification accuracy.
* I found that the simulation data was too artificial for testing classification accuracy.
* To apply deep learning to learn abstract metrics requires a better processing architecture.

# Future directions:

* a proper filter stack, with fine-grained visualization to see the effect of each filter on the trajectories.
* finer customization control over the filter windows for each metric--i.e. for velocity, discard all data that's outside the window $$0 \; \mathrm{m/s} \leq v \leq 5 \; \mathrm{m/s} $$.
* t-SNE parameter tuning.
* fusion embedding, combining deep learning metric functions with hand-selected metrics.
