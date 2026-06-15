---
title: 'The Waker API II: waking across threads'
url: https://without.boats/blog/wakers-ii/
published: "2019-01-11T00:00:00Z"
feed: boats
guid: https://without.boats/blog/wakers-ii/
---

# The Waker API II: waking across threads

In the previous post, I provided a lot of background on what the waker API is trying to solve. Toward the end, I touched on one of the tricky problems the waker API has: how do we handle thread safety for the dynamic Waker type? In this post, I want to look at that in greater detail: what we’ve been doing so far, and what I think we should do.
