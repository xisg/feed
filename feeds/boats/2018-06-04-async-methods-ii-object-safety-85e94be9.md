---
title: 'Async Methods II: object safety'
url: https://without.boats/blog/async-methods-ii/
published: "2018-06-04T00:00:00Z"
feed: boats
guid: https://without.boats/blog/async-methods-ii/
---

# Async Methods II: object safety

Last time, we introduced the idea of async methods, and talked about how they would be implemented: as a kind of anonymous associated type on the trait that declares the method, which corresponds to a different, anonymous future type for each implementation of that method. Starting this week we’re going to look at some of the implications of that. The first one we’re going to look at is object safety.
