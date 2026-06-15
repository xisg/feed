---
title: Iterator, Generator
url: https://without.boats/blog/iterator-generator/
published: "2023-05-06T00:00:00Z"
feed: boats
guid: https://without.boats/blog/iterator-generator/
---

# Iterator, Generator

I have been devoting a lot of my free time in the past month to thinking about structured
concurrency, and a blog post about that is coming soon, but first I want to revisit iterators and
generators.

In [a previous post](https://without.boats/blog/generators), I wrote about one of the hardest problems for generators:
self-referential generators. Unlike the Future trait when we were designing async functions, the
Iterator trait is already stable, and it does not take a pinned reference to itself. This means an
Iterator cannot be self-referential.
