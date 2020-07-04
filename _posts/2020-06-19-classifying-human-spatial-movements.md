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

For my Insight project I collaborated with [Pathr.ai](https://pathr.ai/), a start up based in Mountain View. Pathr uses computer vision to help brick-and-mortar businesses understand how people move through their physical space. To allow Pathr to begin to **classify customer spatial movements**, I developed a set of **trajectory metrics** and an accompanying trajectory analysis pipeline which achieved a separation of simulation trajectories by type. 



![](images/classifying_timelapse.jpeg)


# **Customer spatial analysis: finally becoming automated**

How many customers visit each part of the sales floor, and when? At what point do customers change their minds and decide to make a purchase? Which types of customer-staff interactions lead to higher store profits?

These are fundamental questions for anyone looking to run a competitive retail store today.

As e-commerce websites like Amazon consume the low-margin, high-inventory sales market, the brick-and-mortar retail industry shifts its focus towards high-quality customer experience. How a customer _feels_ about the physical, in-store human interactions has become a primary driver of brick-and-mortar retail profits. For stores to stay competitive today, a store's employees must understand how customers explore and interact with the sales floor.

Until now, if you wanted to gather such customer data for your store, you faced a slow and inaccurate process. To estimate the fraction of visiting customers who make a purchase, say, you'd post someone outside the front door and manually count for hours the number of customers entering the store. An inaccurate headache, to say the least.

And for decades, virtually all brick-and-mortar sales floors have been covered by high-quality security camera feeds. Until 2020, virtually none have applied AI analysis to their data. Recent advances in computer vision is finally making this possible.

This is where [Pathr.ai](https://pathr.ai/) comes in: they are the first to market, as of mid-2020, with an anonymized computer vision system that yields an automated, real time heat map analysis for retail stores.

Moving beyond this early success, Pathr will need to add further data insights to remain competitive in this new market, as several other competitors are working to enter into this retail AI space. My project focused on trajectory analysis, as I will describe below.

# **How Pathr maps customer spatial movement and generates real time analysis**

![pipeline](images/classifying_highlevel.png){: .align-center}


Here's how Pathr's data flow works: A client store streams the video feeds from multiple cameras. Pathr then applies computer vision, combined with geometry measurements of the store's layout, to precisely locate every person in the field of view.

Each person in the store is represented by an (x, y) point, determined by the center of that person's AI-determined bounding box. In order to follow a person's trajectory through the store, the computer vision system assigns them a temporary unique ID that is discarded as soon as the person leaves the field of view. The computer vision is completely anonymizing.

So each trajectory here represents the movement of a single person as a list of (x, y, t) rows, recorded at 30 frames per second:

![pipeline](/images/classify_single_trajectory.png){: .align-center}

Compared to GPS geospatial data, which is accurate to ~1 meter, Pathr's system presently achieves centimeter-precision in measuring customer location. This is thanks in part to the high pixel density of the typical store's security camera system. So in actual application, if x and y are floats with units of meters, x and y will contain meaningful position information to a depth of ~3 decimal points. There is a lot of detail in this data!


Presently, Pathr provides its retail customers with a heat-map analysis product that answers a retailer's fundamental counting question, **"how many people visited each part of the store, and when?"**

# **Pathr is working to add trajectory classification**


Meanwhile, Pathr is also working to add the ability to analyze each **individual customer trajectory**, so as to classify a customer's behavior in real time. Useful insights from such trajectory analysis include, for example,

 * distinguishing customers from staff
 * real time identification of customers who could use assistance
 * revealing precisely how customer store use patterns translate to purchasing behavior
 * detecting shoplifting events as they occur


However, classifying on raw trajectories is difficult; a important obstacle to establishing a functional trajectory analysis product. **Developing such a trajectory classification framework is the focus of my project.**

# **What makes classifying trajectories hard**

To restate our high-level goal, given a customer trajectory (a set of ~1000 (x, y, t) rows), we want to  quickly figure out which abstract categories it falls into. Here's are some of the primary reasons that's difficult.

First, trajectories are not fixed in length. At 30 frames-per-second, each trajectory accrues an additional 1,800 points for every minute a person is in the store. A staff member, for example, working for 2-hours, would result in measured trajectory 100,000 points long, approximately 1 gigabyte of data. In an enormous store with hundreds of employees and customers, performing real time analysis becomes nearly impossible at this scale.

Second, from a machine learning perspective, each additional point (x, y, t) adds an additional 3 dimensions; dimensionality increases dramatically with time. Meanwhile, we don't know _a priori_ over what time- or space-scales the important information occurs. Whether it's 10 points or 10,000 points, the scale of interest will vary across different types of behaviors, as will the signal-to-noise ratio. **Our desired classification behavior should remain stable across different sample window sizes.**

![pipeline](/images/classifying_metric2.png){: .align-center}
*How an ideal classification metric should separate trajectories.*



Third, customer behaviors of interest often live deep below a lot of unrelated information and noise. For instance, how much does a trajectory's absolute location in the store matter? At what scale does the computer vision introduce error? How much do a customer's small side-to-side movements tell us about their macro behaviors, or vice versa?

These questions do not have simple answers; especially not on the raw trajectory data itself.

In short, **we need to project windows of each trajectory into a lower dimensional space that captures its key information.** And we need to then discard the raw trajectory data quickly to keep our memory use minimal. What follows is how I went about optimizing this projection to maximally distill information.


# **I developed a trajectory embedding that extracts identifying information from movement**

To convert each trajectory into a form that is meaningful to a classifier, I developed a **trajectory embedding**, which is just a fixed set of roughly 50 metrics, computed over a specified sample window. So the classification operation becomes: _first_ we map the trajectory through the embedding (essentially a dimensionality reduction operation), _then_ pass the embedded form to the classifier.


![pipeline](/images/classifying_metric1.png){: .align-center}



As to how I chose an intial set of metrics, I focused on statistics that distill kinematic information around various spatial and temporal rates of change. I chose the following statistics in particular because, while capturing key movement descriptions, they are independent of the trajectory's length, sample frequency, and starting location.

**Trajectory metrics that achieved proof-of-concept success:**
1. mean velocity
1. standard deviation of velocity
1. mean acceleration
1. standard deviation of acceleration
1. ratio of bounding rectangle to arc length
1. the lowest 50 fft frequencies in both x and y

After computing these, the embedded trajectories (a ~100-dimensional space) are then rotated via PCA into a space where I keep only the first ~10 principle components. This output, finally, is what I pass to the classifier.

# **A pipeline to allow rapid experimentation on metric sets**

I found that I'd need to iterate through several combinations of statistics to maximize visual clustering. Further, it seemed a likely use case that Pathr will probably require custom metric sets for each new retail client--there is likely not a one-size-fits-all solution, at least not yet, and the ability to rapidly prototype embeddings will be helpful.

To address this, I built a **trajectory embedding pipeline** that applies an open-ended list of metric functions, then shows how well the trajectories separate by category.

![pipeline](/images/classify_pipeline.png){: .align-center}

The pipeline, which utilizes efficient array operations from `Pandas` and `numpy`, can process about 100MB of trajectory data into a metric set in about 30-seconds on my older laptop. I was able to try 60+ combinations drawn from 15 statistics functions in a few hours, arriving at the function set I listed, above.

Additionally, I built the pipeline such that if you want to incorporate an additional metric function into the set, it requires no extra pipeline engineering: it's just a matter of defining a new function and adding its name to the list.

# **Proof-of-concept dataset analysis**

#### **Dataset 1: triangles, circles, and rectangles**

For an initial proof-of-concept test, Pathr and I began with a set of shape trajectories -- a simulated dataset in which someone is moving in the shape of 100 circles, 100 rectangles, and 100 triangles. While all shapes had a consistent orientation, they varied by location and scale.

![](/images/classifying_shapes.png)

With the pipeline I described above, over the course of an hour I tried 60+ combinations of different metric functions on this dataset, visually inspecting the quality of separation in the plot shown at right. In parallel, I used a Random Forest classifier with 5-fold cross-validation on each resulting metric set; while optimum classification accuracy wasn't my goal here, it was a useful extra metric by which to check the practicality of the clustering.

The metric set I settled on achieved a proof-of-concept prediction accuracy of 94%, using only the metrics I described above (mean velocity, mean acceleration, etc) + PCA. I stuck only with metrics that were broadly transferrable to any kind of motion.

#### **Dataset 2: Factory Floor Simulation**

My second proof-of-concept dataset represented a (simulated) factory floor, with 135 people moving each for 7,500 trajectory points, with 3 different behavior roles that required separation. This dataset was 2.5GB and processed in 3 minutes.

![](/images/classifying_factory.png)

With this messier factory data, I applied the same metric set as with the shapes, and achieved a 78% prediction accuracy: this is a much better-than-random baseline but there's still much room for improvement. As you can see in the above plot, there is some clustering by label, but it's not as well separated as with the shapes trajectories.

# **An abstract distance metric**

Pathr additionally asked for a distance metric between any two trajectories, one that can express with a single number how 'close' two trajectories are by category. Especially useful for low-computing cost classification.

I applied a euclidean distance metric to the embedded data to show how far apart each trajectory was. The euclidean distance between two vectors is just an $$N$$-dimensional generalized Pythagorean distance:

$$\text{euclidean distance}(a, b) = \sqrt{(a_1 - b_1)^2 + (a_2 - b_2)^2 + \cdots + (a_n - b_n)^2}$$

This achieved an early success on the shapes data: the triangles are well distinguished.  

![](/images/classifying_distance_metric.png){: .align-center}

The metric set still needs improvement in order to meaningfully differentiate between factory workers.




# **Key takeaways from my work**

1. The ability to quickly try combinations of metric functions is crucial for finding a practical trajectory metric set; in other words, hardcoding a pipeline around some fixed functions will be inevitably unwieldy.
1. My starting grab-bag of rates-of-change kinematics metrics + PCA works pretty well! It will serve as a good benchmark, going forward.
1. Training on actual, hand-labeled human data will be key for pushing further. It seems very likely that Pathr's simulation data is not capturing certain intrinsic and eccentric human movements.




# **Future directions**


After my initial success, there remains wide room for improvement, especially as I didn't yet apply noise filtering or deep learning.

Adding noise filtering is a crucial step in smoothing out human movements and small errors due to computer vision, but it will require delicate controls to surface the correct scale of filtering.

The future work that I'm really excited about is deriving abstract embeddings using supervised deep learning, which would train on the raw trajectories themselves. Ultimately, a combination of human, hand-selected metric functions and deep learned metric functions will probably get the best results.
