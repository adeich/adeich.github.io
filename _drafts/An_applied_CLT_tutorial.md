---
title: "The central limit theorem: a powerful technology"
date:   2019-08-29
published: true
sidebar: toc
layout: single
---

Or: how statistics (a.k.a. being clever) lets you take a lot of decent sensors and make a much better combined sensor from them.

Say you have a digital thermometer that costs $1 and has an accuracy of $$\pm 1$$ degree F. Say that, on a cold day, it reads 32 degrees. How close do you expect this is to the temperature a highly accurate ($$\pm 0.1$$ degree F) thermometer would read?

Now say that you buy a second, identical thermometer to the first. Putting them side by side, they are each measuring the same air.

* How likely do you expect these to agree on an integer temperature?
* Or, say they read 32 and 34 degrees, respectively. What, if anything, does this tell us about what the true air temperature actually is?

Incensed, outraged at the lot the universe has given you, you buy another eight of these thermometers, and put them all side by side, each taking a measurement. At one instant in time, they read

32, 32, 33, 32, 31, 31, 32, 34, 30, 32

Have we learned anything new about the probable nature of the air in the cave? (Guessing by the title of the post--yes.)

Random walks!
