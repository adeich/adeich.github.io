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

For my Insight project I collaborated with [Pathr.ai](https://pathr.ai/), a start up based in Mountain View. Pathr is a geospatial analytics company that helps brick-and-mortar businesses understand how people move through their physical space. To help Pathr **sort and classify customer spatial movements according to behavior**, I developed a set of **trajectory metrics**. 



# **Pathr provides real time customer analysis for retail stores by applying AI to security footage**
In brick-and-mortar retail, customer store-use analytics are vital. How many customers visit each part of the sales floor, and when? When did a given customer change their minds and decide to buy something?

As e-commerce websites like Amazon have consumed low-margin, high-inventory sales markets, the brick-and-mortar retail industry has shifted its focus towards high-quality customer experience. How a customer _feels_ about the physical, in-store human interactions has become a primary driver of brick-and-mortar retail profits. For stores to stay competitive today, a store's employees must understand how customers explore and interact with the sales floor.

Before AI, measuring these customer counts was a slow and inaccurate process. If a store wanted to estimate the fraction of visiting customers who made a purchase, say, they would post an employee outside the front door and manually count the number of customers entering the front door.

In the last several years, computer vision has become highly accurate. And until this year, while virtually all brick-and-mortar stores have wide-coverage, high-quality security camera footage, virtually none have applied AI analysis to this data.

This is where Pathr has entered the analysis market, first to market as of 2020 with a real time, anonymized computer vision system. Pathr will need to add further data insights to remain competitive in this new market, as several other competitors are working to enter into the retail AI-analysis.

# **Pathr maps customer spatial movement and generates real time analysis**

![pipeline](images/classifying_highlevel.png){: .align-center}


Here's how Pathr's data flow works: A client store streams the video feeds from multiple cameras. Pathr then applies computer vision to locate every person in the field of view.

Each person in the store is represented by a 2D (x, y) point determined by the center of that person's AI-determined bounding box. In order to follow each person's progress through the store, the computer vision system assigns them a temporary unique ID that is discarded as soon as the person leaves the field of view.

So each trajectory here represents the movement of a single person as a list of (x, y, t) rows, recorded at 30 frames per second:

![pipeline](/images/classify_single_trajectory.png){: .align-center}

Unlike GPS geospatial data, which is only accurate to ~1 meter, Pathr's system presently achieves centimeter-precision in customer location measurement. This is thanks to the pixel density of the typical store's security camera system. So in actual application, if x and y are floats with units of meters, x and y will contain meaningful position information to a depth of ~5 significant figures. There is a lot of detail in this data!


# **Pathr is working to add trajectory classification**
Presently, Pathr provides its retail customers with a heat-map analysis product that answers the fundamental counting question, **"how many people visit each part of the store, and when?""** Answers to this question are vital for businesses to understand how customers use their store.

Meanwhile, Pathr is also working to add the ability to analyze each **individual customer trajectory**, so as to classify a customer's behavior real time. Useful insights from such trajectory analysis include, for example,

 * distinguishing customers from staff
 * identifying customers who could use assistance from those who are just browsing
 * detecting shoplifting events as they occur
 * discovering the moments where a customer decides they'd like to make a purchase


**Establishing a trajectory classification framework is where my project comes in.**

# **Classifying on raw trajectories is difficult**

To restate our high-level goal, given a customer trajectory (a set of ~1000 (x, y, t) rows), we want to  quickly figure out which abstract categories it falls into. Here's why that's difficult.

First, trajectories are not fixed in length. At 30 frames-per-second, we accrue an additional 1,800 points for every minute a person is in the store. From a machine learning perspective, each additional row (x, y, t) acts as an additional 3 dimensions, so dimensionality blows up rapidly. At the same time, we don't know at what scale the important information occurs. Whether it's 10 points or 10,000 points, the scale of interest will vary across different types of behaviors, as will the signal-to-noise ratio. **Our desired classification sorting needs to remain stable across different sample window sizes.**

![pipeline](/images/classifying_metric2.png){: .align-center}



Second, customer behaviors of interest often live deep below a lot of unrelated information and noise. For instance, how much does a trajectory's absolute location in the store matter? How much error does the computer vision system introduce? How much do a customer's small side-to-side movements tell us about their macro behaviors? And it varies in information-usefulness whether we measure the absolute distance a customer travels or, instead, the distance-normalized pattern the customer traveled in. There are fluctuating scaling and normalization needs for different behaviors. **We need to be able to capture information contained only in certain segments of each feature, and discard the rest.**




Due to all of these complications, applying a supervised learning classifier to raw trajectory data doesn't work very well. As a basic test, I applied a random forest model to our raw simulation trajectories (each clipped to a fixed length of 500 points) and I found that the classifier could barely guess better than random.

And at the other end of the abstract-to-concrete spectrum, applying a set of hand-selected rules to trajectories doesn't work well either. For example, if you say "If the trajectory velocity spends more than 80% of the time within X velocity range, then assign it Y label," this usually performs no better than a random guess, since the interesting distinctions arise only in a much higher dimensional space.


# **I designed a trajectory embedding to distill important trajectory information**

To convert each trajectory into a form that is meaningful to a classifier, I developed a **trajectory embedding**, which is just a fixed set of roughly 100 metrics, computed on each trajectory. So the classification operation becomes: _first_ we map the trajectory through the embedding (essentially a dimensionality reduction operation), _then_ pass the embedded form to the classifier.


![pipeline](/images/classifying_metric1.png){: .align-center}



For my embedding, I focused on statistics that distill kinematic information around various rates of change. I chose the following statistics in particular because, while capturing key movement descriptions, they are independent of the trajectory's length, sample frequency, and starting location:

* mean velocity
* standard deviation of velocity
* mean acceleration
* standard deviation of acceleration
* ratio of bounding rectangle to arc length
* the lowest 50 fft frequencies in both x and y.

After computing these, the embedded trajectories (a ~100-dimensional space) are then rotated via PCA into a space where I keep only the first ~10 principle components. This output, finally, is what I pass to the classifier.

# I designed a pipeline to allow rapid experimentation on metric sets

I knew I would have to iterate through many combinations of statistics to maximize visual clustering. Further, it seems likely that Pathr will probably require custom metric sets for each new retail space they're analyzing--there is probably not a one-size-fits-all solution, at least not yet.

To address this, I built a trajectory embedding pipeline which applies a purposefully open-ended list of embedding functions, then shows how well the trajectories separate by category.

![pipeline](/images/classify_pipeline.png){: .align-center}

My pipeline, utilizing high-speed array operations with `Pandas` and `numpy`, can distill about 100MB of trajectory data into a metric set in about 30-seconds on my older laptop. I was able to try 60+ combinations drawn from 15 statistics functions in a few hours, arriving at the function set I listed, above.

Additionally, incorporating future statistics metrics into the set requires no additional pipeline engineering, as it's just a matter of defining a new function and adding its name to the list.

# Proof-of-concept dataset analysis

My first proof-of-concept test was a set of shape trajectories -- a simulated dataset composed of 100 circles, 100 rectangles, and 100 triangles.

My second proof-of-concept demonstration was a simulated factory floor, with 135 people moving around, with 3 different behavior roles.

Insights:

* Grab-bag of kinematics metrics + PCA works pretty well as a start.
* Initial metric set achieved high classification accuracy.
* I found that the simulation data was too artificial for testing classification accuracy.
* To apply deep learning to learn abstract metrics requires a better processing architecture.

# Future directions:

* a proper filter stack, with fine-grained visualization to see the effect of each filter on the trajectories.
* finer customization control over the filter windows for each metric--i.e. for velocity, discard all data that's outside the window $$0 \; \mathrm{m/s} \leq v \leq 5 \; \mathrm{m/s} $$.
* t-SNE parameter tuning.
* fusion embedding, combining deep learning metric functions with hand-selected metrics.
