---
title: Coroutines and effects
url: https://without.boats/blog/coroutines-and-effects/
published: "2024-04-20T00:00:00Z"
feed: boats
guid: https://without.boats/blog/coroutines-and-effects/
---

# Coroutines and effects

For the past few months I’ve been mulling over some things that Russell Johnston made me realize
about the relationship between effect systems and coroutines. You can read more of his thoughts on
this subject [here](https://www.abubalay.com/blog/2024/01/14/rust-effect-lowering), but he made me realize that effect systems (like that found in Koka)
and coroutines (like Rust’s async functions or generators) are in some ways isomorphic to one
another. I’ve been pondering the differences between them, trying to figuring out the advantages and
disadvantages of each.

A few weeks ago, Will Crichton posted something on [Twitter](https://twitter.com/tonofcrates/status/1770560175835058573) that helped bring the
contrast into sharper focus for me:

> The entire field of PL right now: what if it was dynamically scoped…. but statically
> typed…………..? (effects, capabilities, contexts, metavariables…)

I’m just a humble language designer (and not a theorist of anything, especially not PL), so my
focus is the difference in user experience and affordance. But this seems like a cutting insight and
this property of effect handlers - static typing but dynamic scoping - seems to me to be a good
jumping off point for understanding the difference between effect handlers and coroutines from a
user perspective.
