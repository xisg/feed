---
title: Serial, Parallel, and Quorum Latencies
url: http://brooker.co.za/blog/2021/10/20/simulation.html
published: "2021-10-20T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2021/10/20/simulation
---

# Serial, Parallel, and Quorum Latencies

Why are they letting me write Javascript?

I’ve written [before](https://brooker.co.za/blog/2021/04/19/latency.html) about the latency effects of series (do X, then Y), parallel (do X and Y, wait for them both), and quorum (do X, Y and Z, return when two of them are done) systems. The effects of these different approaches to doing multiple things are quite intuitive. What may not be intuitive, though, is the impact of quorums, and how much quorums can reduce tail latency.

So I put together this little toy simulator.

The knobs are:

- **serial** The number of things to do in a _chain_.
- **parallel** The number of parallel _chains_.
- **quorum** The number of chains we wait to complete before being done.
- **runs** How many times to sample.

So, for example, a traditional 3-of-5 Paxos system would have serial=1, parallel=5, and quorum=3. A length-3 chain replication system would have serial=3, parallel=1, quorum=1. The per-node service time distribution is (for now) assumed to be exponentially distributed with mean 1.

**Examples to Try**

- Compare a 3-length chain to a 3-of-5 Paxos system. First, set serial=3, parallel=1, and quorum=1 and see how the 99th percentile latency is somewhere around 8s. Now, try serial=1, parallel=5, quorum=3. Notice how the 99th percentile is now just over 2ms. There’s obviously a lot more to chain-vs-quorum in the real world than what is captured here.
- Compare a 3-of-5 quorum to 4-of-7. The effect isn’t as big here, but the bigger quorum leads to a nice reduction in high-percentile latency.
- Check out the non-linear effect of longer serial chains. The 99th percentile doesn’t increase by 10x between serial=1 and serial=10. Why?

Have fun!
