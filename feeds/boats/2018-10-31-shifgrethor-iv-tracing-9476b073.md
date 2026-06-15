---
title: 'Shifgrethor IV: Tracing'
url: https://without.boats/blog/shifgrethor-iv/
published: "2018-10-31T00:00:00Z"
feed: boats
guid: https://without.boats/blog/shifgrethor-iv/
---

# Shifgrethor IV: Tracing

The post before this one covered how shifgrethor handles rooting: how we track for the garbage collector that this object is alive. That isn’t sufficient for implementing a tracing garbage collector though: the idea of a tracing garbage collector is that we can trace from rooted objects through all of the objects they reference. That way, instead of having to root everything you use, you can only root a few objects from which all of the live objects can be traced.
