---
title: 'Async/Await II: Narrowing the Scope of the Problem'
url: https://without.boats/blog/async-ii-narrowing-the-scope/
published: "2018-01-31T00:00:00Z"
feed: boats
guid: https://without.boats/blog/async-ii-narrowing-the-scope/
---

# Async/Await II: Narrowing the Scope of the Problem

Last time we talked about the broader problem that generators with references across yield points represent: self-referential structs. This time, I want to narrow in on the specific problem that needs to be solved to make generators work, and also discuss some ideas for solutions that I think are false starts.
(I still don’t have a proposal about what to do in this post, but it will come soon enough!)
