---
title: Pass@k is Mostly Bunk
url: http://brooker.co.za/blog/2026/01/21/pass-k.html
published: "2026-01-21T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2026/01/21/pass-k
---

# Pass@k is Mostly Bunk

Exponentially better results? I'll take three!

Measuring the success of AI agents isn’t easy. It’s very sensitive to what _success_ means, it can require a lot of samples, its highly context sensitive. Generally hard. So it doesn’t help that one of the most common metrics used for agents is (mostly) bunk. I’m talking about _pass@k_.

What is _pass@k_? It’s the probability that at least one of _k_ different attempts will succeed. A six-sided die, where _pass_ means rolling a 6, has a pass@3 of 45% and a pass@10 of 83%. A D20 has a pass@25 of 72%, and a pass@100 of 99.4%.

![](/blog/images/d6_trimmed.jpg)

99.4%! What a great evaluation result! Clearly the model is doing something meaningful and useful! No, it’s doing something meaningful and useful 5% of the time.

The problem with pass@k is that’s _exponentially_ forgiving. There’s a value of _k_, a fairly low one generally, that can make anything look good. Here’s that six-sided die again:

Humans interacting with agents aren’t nearly that forgiving. They, in general, aren’t saying “well, I tried 10 times and it worked once, so I’m happy”. They’re saying “I tried 10 times and it only worked once, what a piece of junk”. They’re also doing multiple steps, and only happy when they all work. Exponentially unforgiving (for which pass^k is a much better metric).

Why only _mostly_ bunk? There are cases, where tasks are simple, evaluators are reliable, and humans are out of the loop, that the idea of getting exponentially better success rate with linear additional cost is good. I’ve made a similar argument about distributed systems [in the past](https://brooker.co.za/blog/2023/09/08/exponential.html). But these tasks aren’t ubiquitous. Pass@k should be a metric that’s rarely used, and carefully justified every time it is used.

If we’re going to drive the field of agentic AI forward, we need to keep ourselves honest on metrics.

_Footnotes_

1. It’s mildly interesting how none of the image generation models reliably generate images of legal dice.
