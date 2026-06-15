---
title: From failure to Fehler
url: https://without.boats/blog/failure-to-fehler/
published: "2020-04-06T00:00:00Z"
feed: boats
guid: https://without.boats/blog/failure-to-fehler/
---

# From failure to Fehler

About two and a half years ago I wrote a Rust library called failure, which quickly became one of the most popular error handling libraries in Rust. This week, its current maintainer decided to deprecate it, a decision I strongly support. This week, I also released a new and very different error-handling library, called fehler. I wanted to discuss these two libraries briefly.
A brief history of failure When I released failure, the most popular error handling library by far was error-chain.
