+++
Description = ""
categories = ["tutorial"]
comments = "yes"
date = "2015-09-20T14:58:56+01:00"
draft = true
share = "yes"
tags = ["protractor", "testing", "javascript", "selenium"]
title = "Running Protractor scenarios in parallel"

+++
Protractor can run tests in parallel, but only one feature file per Selenium instance. If a single
feature file has more than one scenario, they cannot be parallelised out of the box.

One solution to run scenarios in parallel is to write each new scenario in a new `.feature` file.
However this breaks the natural grouping that feature files provide, and besides, it may already be
too late if your team has many multi-scenario features.

The alternative is to only "flatten" these scenarios into separate feature files just before running
Protractor, and then cleaning them up afterwards.

# The Process

TODO: Walk through `flatten.js` and `flattener.js`.

# Optimization

Okay, so we have improved our testing framework to run all of our scenarios in parallel, no matter
how we structure our feature files. But there is still room for improvement.

Protractor creates a new Selenium instance for each feature file that it processes. This means that
if you have a fixed maximum number of Selenium instances which is less than the number of scenarios,
simply flattening scenarios one per feature file will be inefficient: each of the instances will be
set up and torn down multiple times per run, incurring a few seconds of overhead each time.

The solution, given *N* maximum Selenium instances, is to pack the scenarios into *N* feature files.
This way, each Selenium instance will only be set up and torn down once.

New problem: packing scenarios naively (e.g. randomly) will be inefficient, since the test suite
will take as long as the longest-running feature file. Ideally, all feature files would take roughly
the same amount of time to run through all of their scenarios.

It is impossible to determine the optimal packing a priori, since individual steps can take vastly
different amounts of time, and even the same step may behave differently depending on application 
context. As such, it is necessary to first run the tests, record the durations of each scenario, and
then use the duration data to pack scenarios so that the sums of the durations per feature file are
roughly equal (this is the multi-way number partitioning problem; see [Korf, 2009][korf]).

This packing strategy may be customized to whatever environment you run your tests in. For example,
if you have heterogeneous hardware (some machines faster than others), you may be able to force 
certain features to run on the slower machines, and pack fewer scenarios into them.


[korf]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.150.2326&rep=rep1&type=pdf