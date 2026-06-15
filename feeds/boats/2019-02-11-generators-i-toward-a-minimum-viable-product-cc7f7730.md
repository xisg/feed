---
title: 'Generators I: Toward a minimum viable product'
url: https://without.boats/blog/generators-i/
published: "2019-02-11T00:00:00Z"
feed: boats
guid: https://without.boats/blog/generators-i/
---

# Generators I: Toward a minimum viable product

We’re still not finished with the design of async/await, but it’s already become clear that it’s time to get the next phases of the feature into the pipeline. There are two extensions to the minimal async/await feature we’ve currently got that seem like the clear high priority:
Async methods: allowing async fn to be used in traits. Generators: allowing imperative control flow to create Iterators and Streams the same way async fn allows imperative control flow to create a Future.
