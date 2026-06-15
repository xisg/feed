---
title: Exponential Value at Linear Cost
url: http://brooker.co.za/blog/2023/09/08/exponential.html
published: "2023-09-08T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2023/09/08/exponential
---

# Exponential Value at Linear Cost

What a deal!

Binary search is kind a of a magical thing. With each additional search step, the size of the haystack we can search doubles. In other words, the value of a search is _exponential_ in the amount of effort. That’s a great deal. There are a few similar deals like that in computing, but not many. How often, in life, do you get exponential value at linear cost?

Here’s another important one: redundancy.

If we have $N$ hosts, each with availability $A$, any one of which can handle the full load of the system, the availability of the total system is:

$A\_{system} = 1 - (1 - A)\\^N$

It’s hard to overstate how powerful this mechanism is, and how important it has been to the last couple decades of computer systems design. From RAID to cloud services, this is the core idea that makes them work. It’s also a little hard to think about, because our puny human brains just can’t comprehend the awesome power of exponents (mine can’t at least).

If you want to try some numbers, give this a whirl:

What you’ll realize pretty quickly is that this effect is very hard to compete with. No matter how high you make the availability for a single host, even a very poor cluster quickly outperforms it in this simple model. Exponentiation is extremely powerful.

Unfortunately it’s not all good news. This exponentially powerful effect only works when all these hosts fail independently. Let’s extend the model just a little bit to include the effect of them being in the same datacenter, and that datacenter having availability $D$. We can easily show that the availability of the total system then becomes:

$A\_{system} = D \* (1 - (1 - A)\\^N)$

Which doesn’t look nearly as good.

Which goes to show how quickly things go wrong when there’s some correlation between the failures of redundant components. System designers must pay careful attention to ensuring that designs consider this effect, almost beyond all others, when designing distributed systems. Exponential goodness is our most powerful ally. Correlated failures are its kryptonite.

This observation is obviously fairly basic. It’s also critically important.
