---
title: 'Async/Await I: Self-Referential Structs'
url: https://without.boats/blog/async-i-self-referential-structs/
published: "2018-01-25T00:00:00Z"
feed: boats
guid: https://without.boats/blog/async-i-self-referential-structs/
---

# Async/Await I: Self-Referential Structs

This is the first in a series of blog posts about generators, async & await in Rust. By the end of this series, I will have a proposal for how we could expediently (within the next 12 months) bring generators, async & await to stable Rust, and resolve some of the most difficult ergonomics problems in the futures ecosystem.
But that proposal is still several posts away. Before we can get to a concrete proposition, we need to understand the scope & nature of the problem we need to solve.
