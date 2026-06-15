---
title: The Fundamental Mechanism of Scaling
url: http://brooker.co.za/blog/2021/01/22/cloud-scale.html
published: "2021-01-22T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2021/01/22/cloud-scale
---

# The Fundamental Mechanism of Scaling

It's not Paxos, unfortunately.

A common misconception among people picking up distributed systems is that replication and consensus protocols—Paxos, Raft, and friends—are the tools used to build the largest and most scalable systems. It’s obviously true that these protocols are important building blocks. They’re used to build systems that offer more availability, better durability, and stronger integrity than a single machine. At the most basic level, though, they don’t make systems scale.

Instead, the fundamental approach used to scale distributed systems is _avoiding_ co-ordination. Finding ways to make progress on work that doesn’t require messages to pass between machines, between clusters of machines, between datacenters and so on. The fundamental tool of cloud scaling is coordination avoidance.

**A Spectrum of Systems**

With this in mind, we can build a kind of spectrum of the amount of coordination required in different system designs:

_Coordinated_ These are the kind that use paxos, raft, chain replication or some other protocol to make a group of nodes work closely together. The amount of work done by the system generally scales with the offered work ( _W_) and the number of nodes ( _N_), something like O( _N_ \\* _W_) (or, potentially, worse under some kinds of failures).

_Data-dependent Coordination_ These systems break their workload up into uncoordinated pieces (like shards), but offer ways to coordinate across shards where needed. Probably the most common type of system in this category is sharded databases, which break data up into independent pieces, but then use some kind of coordination protocol (such as two-phase commit) to offer cross-shard transactions or queries. Work done can vary between O( _W_) and O( _N_ \\* _W_) depending on access patterns, customer behavior and so on.

_Leveraged Coordination_ These systems take a coordinated system and build a layer on top of it that can do many requests per unit of coordination. Generally, coordination is only needed to handle failures, scale up, redistribute data, or perform other similar management tasks. In the happy case, work done in these kinds of systems is O( _W_). In the bad case, where something about the work or environment forces coordination, they can change to O( _N_ \\* _W_) (see [Some risks of coordinating only sometimes](http://brooker.co.za/blog/2019/05/01/emergent.html) for more). Despite this risk, this is a rightfully popular pattern for building scalable systems.

_Uncoordinated_ These are the kinds of systems where work items can be handled independently, without any need for coordination. You might think of them as embarrassingly parallel, sharded, partitioned, geo-partitioned, or one of many other ways of breaking up work. Uncoordinated systems scale the best. Work is always O( _W_).

This is only one cut through a complex space, and some systems don’t quite fit[1](#foot1). I think it’s still useful, though, because by building a hierarchy of coordination we can think clearly about the places in our systems that scale the best and worst. The closer a system is to the uncoordinated end the better it will scale, in general.

**Other useful tools**

There are many other ways to approach this question of when coordination is necessary, and how that influences scale.

The CAP theorem[2](#foot2), along with a rich tradition of other impossibility results[3](#foot3), places limits on the kinds of things systems can do (and, most importantly, the kinds of things they can offer to their clients) without needing coordination. If you want to get into the details there, the breakdown in Figure 2 of [Highly Available Transactions: Virtues and Limitations](http://www.bailis.org/papers/hat-vldb2014.pdf) is pretty clear. I like it because it shows us both what is possible, and what isn’t.

The [CALM theorem](https://arxiv.org/pdf/1901.01930.pdf) is very useful, because it provides a clear logical framework for whether particular programs can be run without coordination, and something of a path for constructing programs that are coordination free. If you’re going to read just one distributed systems paper this year, you could do a lot worse than [Keeping CALM](https://arxiv.org/pdf/1901.01930.pdf).

[Harvest and Yield](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.33.411) is another way to approach the problem, by thinking about when systems can return partial results[4](#foot4). This is obviously a subtle topic, because the real question is when your clients and customers can accept partial results, and how confused they will be when they get them. At the extreme end, you start expecting clients to write code that can handle any subset of the full result set. Sometimes that’s OK, sometimes it sends them down the same rabbit hole that CALM takes you down. Probably the hardest part for me is that partial-result systems are hard to test and operate, because there’s a kind of mode switch between partial and complete results and [modes make life difficult](https://aws.amazon.com/builders-library/avoiding-fallback-in-distributed-systems/). There’s also the minor issue that there are 2N subsets of results, and testing them all is often infeasible. In other words, this is a useful too, but it’s probably best not to expose your clients to the full madness it leads to.

Finally, we can think about the work that each node needs to do. In a _coordinated_ system, there is generally one or more nodes that do O( _W_) work. In an uncoordinated system, the ideal node does O( _W_/ _N_) work, which turns into O(1) work because _N_ is proportional to _W_.

**Footnotes**

1. Like systems that coordinate heavily on writes but mostly avoid coordination on reads. [CRAQ](https://www.usenix.org/legacy/event/usenix09/tech/full_papers/terrace/terrace.pdf) is one such system, and a paper that helped me fall in love with distributed systems. So clever, and so simple once you understand it.
2. Best described by [Brewer and Lynch](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.6951&rep=rep1&type=pdf).
3. See, for example, Nancy Lynch’s 1989 paper [A Hundred Impossibility Proofs for Distributed Computing](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.13.5022). If there were a hundred of these in 1989, you can imagine how many there are now, 32 years later. Wow, 1989 was 32 years ago. Huh.
4. I wrote [a post](http://brooker.co.za/blog/2014/10/12/harvest-yield.html) about it back in 2014.
