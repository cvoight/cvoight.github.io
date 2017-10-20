---
layout: post
title:  "Why F#? – Part 1"
date:   2017-10-20 00:00:00 -0400
categories: why fsharp
---

In the Beginning, There Was Dynamo
==================================
Earlier this year, I started using [Dynamo](http://dynamobim.org) to try to automate some tasks in Revit. Dynamo is fantastic because you don't need to understand much about Revit's internals, the Revit API, or programming in order to do all sorts of cool things. But after building a couple of useful bits in it, I quickly became frustrated with its control flow and capabilities (even when supplemented by excellent packages, such as archi-lab and SteamNodes).

Building programs in Dynamo is slow. Figuring out the sequence of nodes to perform a task, placing them, and hooking everything together is tedious.

Building more advanced programs in Dynamo is impossible without writing your own custom code nodes. Custom code nodes are a fundamental part of why Dynamo is so useful, but Dynamo's strictures are difficult to deal with:
* limited to one output
* dictionaries (and many other collections) have to be wrapped to be passed between nodes
* the Python script editor is quite bad, to put it mildly

These strictures can be dealt with if you're doing a simple task. Take, for example, this snippet, which determines standard breaker ratings (NEC 240.6) for a circuit based on the circuit's apparent current.

{% highlight python %}
import sys
sys.path.append(r'C:\Program Files (x86)\IronPython 2.7\Lib')
from bisect import bisect_left

elementIDs = IN[0]
breakerRatings = dict(zip(elementIDs, IN[1]))
apparentCurrents = dict(zip(elementIDs, IN[2]))
standardBreakerRatings = IN[3]

adjustedBreakerRatings = []

for key in elementIDs:
    if (breakerRatings[key] == 20) and (apparentCurrents[key] > 20):
        index = bisect_left(standardBreakerRatings, apparentCurrents[key])
        adjustedBreakerRatings.append([key, standardBreakerRatings[index]])

OUT = adjustedBreakerRatings
{% endhighlight %}

Unfortunately, this quickly falls apart because not all circuits should be sized the same way. Circuits with mechanical equipment generally need to be sized based on the manufacturer's maximum overcurrent protection (MOCP). Circuits with electrical equipment need to be sized based either on the standard main circuit breaker (MCB) or main lugs only (MLO) rating. (Corresponding to the rating feeding your panel schedules, which is usually the type parameter "MCB Rating".)

And so, after some initial success with Dynamo, I decided it was finally time to delve into the Revit API proper and see what was available. Enter: RevitPythonShell.