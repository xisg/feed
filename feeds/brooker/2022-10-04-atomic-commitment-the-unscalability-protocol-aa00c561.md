---
title: 'Atomic Commitment: The Unscalability Protocol'
url: http://brooker.co.za/blog/2022/10/04/commitment.html
published: "2022-10-04T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2022/10/04/commitment
---

# Atomic Commitment: The Unscalability Protocol

2PC is my enemy.

Let’s consider a single database system, running on one box, good for 500 requests per second.

```
┌───────────────────┐
│     Database      │
│(good for 500 rps) │
└───────────────────┘

```

What if we want to access that data more often than 500 times a second? If by _access_ we mean _read_, we have a lot of options. If be _access_, we mean _write_ or even _perform arbitrary transactions on_, we’re in a trickier situation. Tricky problems aside, we forge ahead by splitting our dataset into two shards:

```
┌───────────────────┐  ┌───────────────────┐
│ Database shard 1  │  │ Database shard 2  │
│(good for 500 rps) │  │(good for 500 rps) │
└───────────────────┘  └───────────────────┘

```

If we’re just doing single row reads and writes, we’re most of the way there. We just need to add a routing layer that can decide which shard to send each access to, and we’re done[1](#foot1):

```
              ┌────────────┐
              │   Router   │
              └────────────┘
                     ┬
           ┌─────────┴───────────┐
           ▼                     ▼
┌───────────────────┐  ┌───────────────────┐
│ Database shard 1  │  │ Database shard 2  │
│(good for 500 rps) │  │(good for 500 rps) │
└───────────────────┘  └───────────────────┘

```

But what if we have transactions? To make the complexity reasonable, and speed us on our journey, let’s define a _transaction_ as an operation that does writes to multiple rows, based on some condition, atomically. By _atomically_ we mean that either all the writes happen or none of them do. By _based on some condition_ we mean the transactions can express ideas like “reduce my bank balance by R10 as long as it’s over R10 already”.

But how do we ensure atomicity across multiple machines? This is a classic computer science problem called [Atomic Commitment](https://en.wikipedia.org/wiki/Atomic_commit). The classic solution to this classic problem is [Two-phase commit](https://en.wikipedia.org/wiki/Two-phase_commit_protocol), maybe the most famous of all distributed protocols. There’s a _lot_ we could say about atomic commitment, or even just about two-phase commit. In this post, I’m going to focus on just one aspect: atomic commitment has weird scaling behavior.

**How Fast is our New Database?**

The obvious question after sharding our new database is _how fast is it?_ How much throughput can we get out of these two machines, each good for 500 transactions a second.

The optimist’s answer is 500 + 500 = 1000. We doubled capacity, and so can now do more work. But we need to remind the optimist that we’re solving a distributed transaction problem here, and that at least some transactions go to both shards.

For the next step in our analysis, we want to measure the mean number of shards any given transaction will visit. Let’s call it _k_. For _k = 1_ we get perfect scalability! For _k = 2_ we get no scalability at all: both shards need to be visited on every transaction, so we only get 500 transactions a second out of the whole thing. The capacity of the database is the sum of the per-node capacities, divided by _k_.

**How do we spread the data?**

We haven’t mentioned, so far, how we decide which data to put onto which shard. This is a whole complex topic and active research area of its own. The problem is a tough one: we want to spread the data out so about the same number of transactions go to each shard (avoiding _hot shards_), and we want to minimize the number of shards any given transaction touches (minimize _k_). We have to do this in the face of, potentially, very non-uniform access patterns.

But let’s put that aside for now, and instead model how _k_ changes with the number of rows in each transaction ( _N_), and number of shards in the database ( _s_). Borrowing from [this StackExchange answer](https://stats.stackexchange.com/a/296053), and assuming that each transaction picks uniformly from the key space, we can calculate:

$k = s \\left( 1 - \\left( \\frac{s-1}{s} \\right) ^ N \\right)$

You can picture that in your head, right? If, like me, you probably can’t, it looks like this:

![](https://mbrooker-blog-images.s3.amazonaws.com/blog_k_versus_n_s.png)

_k_ is fairly nicely behaved for small _N_ or small _s_, but things start to get ugly when both _N_ and _s_ are large. Remember that the absolute maximum throughput we can get out of this database is

$\\mathrm{Max TPS} \\propto \\frac{s}{k}$

Let’s consider the example of _N=10_. How does the maximum TPS vary with _s_ as we increase the number of shards from 1 to 10:

$\\mathrm{Max TPS}(s = 1..10, N=10)
\\propto \[1.000000, 1.000978, 1.017648, 1.059674, 1.120290, 1.192614, 1.272359, 1.356991, 1.444974, 1.535340\]$

Oof! For _N = 10_, adding a second shard only increases our throughput by something like 1% for uniformly distributed keys! The classic solution is to hope that your keys aren’t uniformly distributed, and that you can keep _k_ low without causing hotspots. A nice solution, if you can get it.

**But wait, it gets worse!**

This is where our old friend, concurrency, comes back to haunt us. Let’s think about what happens when we get into the state where each shard can only handle one more transaction[2](#foot2), and two transactions come in, each wanting to access both shards.

```
             ┌────┐    ┌────┐
             │ T1 │    │ T2 │
             └────┘    └────┘
                │         │
                │         │
          ┌─────┴─────────┴──────┐
          │                      │
          ▼                      ▼
┌───────────────────┐  ┌───────────────────┐
│ Database shard 1  │  │ Database shard 2  │
│ (can only handle  │  │ (can only handle  │
│     one more)     │  │     one more)     │
└───────────────────┘  └───────────────────┘

```

Clearly, only one of T1 and T2 can succeed. They can also, sadly, both fail. If T1 gets to shard 1 first, and T2 gets to shard 2 first, neither will get the capacity it needs from the other shard. Then both fail[3](#foot3). We can look at this using a simulation, and see how pronounced the effect can be:

![](https://mbrooker-blog-images.s3.amazonaws.com/paper_synth_with_limit_unif_goodput.png)

In this simulation, with Poisson arrivals, offered load far in excess of the system capacity, and uniform key distribution, goodput for _N = 10_ drops significantly as the shard number increases, and doesn’t recover until _s = 6_. This effect is surprising, and counter-intuitive. Effects like this make transaction systems somewhat uniquely hard to scale out. For example, splitting a single-node database in half could lead to worse performance than the original system.

Fundamentally, this is because scale-out depends on [avoiding coordination](https://brooker.co.za/blog/2021/01/22/cloud-scale.html) and atomic commitment is all about coordination. Atomic commitment is the anti-scalability protocol.

**Footnotes**

1. Obviously not _done done_. Building scale-out databases even for single-row accesses turns out to be super hard in other ways. For a good discussion of that, check out the 2022 [DynamoDB paper](https://www.usenix.org/conference/atc22/presentation/vig).
2. Because of thread limits, or concurrency limits, or connection limits, or anything else that limits the total number of outstanding transactions that the shard can handle. The details matter a whole lot in practice, but matter little in this simple model.
3. You might be thinking that we could just queue both of them up. Which we _could_, but that would have other bad impacts. In general, long queues are really bad for system stability.
