---
title: Regions tradeoffs
url: https://smallcultfollowing.com/babysteps/blog/2012/02/22/regions-tradeoffs/?utm_source=atom_feed
published: "2012-02-22T00:00:00Z"
feed: babysteps
guid: https://smallcultfollowing.com/babysteps/blog/2012/02/22/regions-tradeoffs/
---

# Regions tradeoffs

In the last few posts I’ve been discussing various options for
regions. I’ve come to see region support as a kind of continuum,
where the current system of reference modes lies at one end and a
full-blown region system with explicit parameterized types and
user-defined memory pools lies at the other. In between there are
various options. To better explore these tradeoffs, I wrote up a
document that
[outlines various possible schemes and also details use cases that are enabled by these schemes](https://smallcultfollowing.com/babysteps/
/rust/regions-tradeoffs).
I don’t claim this to be a comprehensive list of all possible schemes,
just the ones I’ve thought about so far. In some cases, the
descriptions are quite hand-wavy. I also think some of them don’t
hang together so well.
