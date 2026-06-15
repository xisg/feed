---
title: Compile time unit arithmetic in C++
url: https://blog.m-ou.se/compile-time-unit-arithmetic/
published: "2013-09-29T18:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/compile-time-unit-arithmetic/
---

# Compile time unit arithmetic in C++

In [Software for Infrastructure](http://www.stroustrup.com/Software-for-infrastructure.pdf),
Bjarne Stroustrup shows a way to use templates to make the C++ compiler
aware of the unit of a value (e.g. kilograms, seconds, etc.), such that it can check consistent use
and prevent disasters like the well known
[error at NASA in 1999](http://en.wikipedia.org/wiki/Mars_Climate_Orbiter#Cause_of_failure)
caused by mixing incompatible units.
In this article, I show how to extend this idea to support any number of base units and
linearly related units (e.g. centimetres, metres and inches) by teaching
the compiler how to do arithmetic on units.
