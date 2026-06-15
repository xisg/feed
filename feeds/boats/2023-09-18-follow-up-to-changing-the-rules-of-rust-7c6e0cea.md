---
title: Follow up to "Changing the rules of Rust"
url: https://without.boats/blog/follow-up-to-changing-the-rules-of-rust/
published: "2023-09-18T00:00:00Z"
feed: boats
guid: https://without.boats/blog/follow-up-to-changing-the-rules-of-rust/
---

# Follow up to "Changing the rules of Rust"

In [my previous post](https://without.boats/blog/changing-the-rules-of-rust), I described the idea of using an edition mechanism to introduce a new
auto trait. I wrote that the compiler would need to create an “unbreakable firewall” to prevent
using `!Leak` types from the new edition with code from the old edition that assumes values of all
types can be leaked.

The response has been pretty optimistic that ensuring this would be possible, even though I wrote in
the post myself that I “despair” over how difficult it was. I’ve received a great example from Ariel
Ben-Yehuda which demonstrates how this problem is more difficult to solve than you would probably
think.
