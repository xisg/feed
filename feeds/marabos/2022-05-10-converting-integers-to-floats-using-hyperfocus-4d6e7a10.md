---
title: Converting Integers to Floats Using Hyperfocus
url: https://blog.m-ou.se/floats/
published: "2022-05-10T00:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/floats/
---

# Converting Integers to Floats Using Hyperfocus

A few years ago, due to some random chain of events, I ended up implementing a conversion from 128 bit integers to 64 bit floats.
This would’ve turned out to be a complete waste of time,
except that my final version is faster than the builtin conversion of every compiler I tested.
In this blog post, I’ll explain what happened, how floats work, how this conversion works, and how it got a bit out of hand.
