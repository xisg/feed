---
title: A four year plan for async Rust
url: https://without.boats/blog/a-four-year-plan/
published: "2023-11-07T00:00:00Z"
feed: boats
guid: https://without.boats/blog/a-four-year-plan/
---

# A four year plan for async Rust

Four years ago today, the Rust async/await feature was released in version 1.39.0. The announcement
[post](https://blog.rust-lang.org/2019/11/07/Async-await-stable.html) says that “this work has been a long time in development – the key
ideas for zero-cost futures, for example, were first proposed by Aaron Turon and Alex Crichton in
2016”. It’s now been longer since the release of async/await than the time between the first design
work on futures and the release of async/await syntax. Despite this, and despite the fact that
async/await syntax was explicitly shipped as a “minimum viable product,” the Rust project has
shipped almost no extensions to async/await in the four years since the MVP was released.

This fact has been noticed, and I contend it is the primary controllable reason that async Rust has
developed a negative reputation (other reasons, like its [essential complexity](https://without.boats/blog/why-async-rust), are
not in the project’s control). It’s encouraging to see project leaders like Niko Matsakis
[recognize](https://smallcultfollowing.com/babysteps/blog/2023/10/14/eurorust-reflections/) the problem as well. I want to outline the features that I think async Rust needs
to continue to improve its user experience. I’ve organized these features into features that I think
the project could ship in the short term (say, in the next 18 months), to those that will take
longer (up to three years), and finally a section on a potential change to the language that I think
would take years to plan and prepare for.
