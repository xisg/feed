---
title: Consensus is Harder Than It Looks
url: http://brooker.co.za/blog/2020/10/05/consensus.html
published: "2020-10-05T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2020/10/05/consensus
---

# Consensus is Harder Than It Looks

And it looks pretty hard.

In his classic paper [How to Build a Highly Available System Using Consensus](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.61.8330&rep=rep1&type=pdf) Butler Lampson laid out a pattern that’s become very popular in the design of large-scale highly-available systems. Consensus is used to deal with unusual situations like host failures (Lampson says _reserved for emergencies_), and leases (time-limited locks) provide efficient normal operation. The paper lays out a roadmap for implementing systems of this kind, leaving just the implementation details to the reader.

The core algorithm behind this paper, Paxos, is famous for its complexity and subtlety. Lampson, like many who came after him[1](#foot1), try to build a framework of specific implementation details around it to make it more approachable. It’s effective, but incomplete. The challenge is that Paxos’s subtlety is only one of the hard parts of building a consensus system. There are three categories of challenges that I see people completely overlook.

**Determinism**

> “How can we arrange for each replica to do the same thing? Adopting a scheme first proposed by Lamport, we build each replica as a deterministic state machine; this means that the transition relation is a function from (state, input) to (new state, output). It is customary to call one of these replicas a ‘process’. Several processes that start in the same state and see the same sequence of inputs will do the same thing, that is, end up in the same state and produce the same outputs” - Butler Lampson (from [How to Build a Highly Available System Using Consensus](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.61.8330&rep=rep1&type=pdf)).

Conceptually, that’s really easy. We start with a couple of replicas with _state_, feed them _input_, and they all end up with _new state_. Same inputs in, same state out. Realistically, it’s hard. Here are just some of the challenges:

- _Concurrency_. Typical runtimes and operating systems use more than just your program’s state to schedule threads, which means that code that uses multiple threads, multiple processes, remote calls, or even just IO, can end up with non-deterministic results. The simple fix is to be resolutely single-threaded, but that has severe performance implications[2](#foot2).
- _Floating Point_. Trivial floating-point calculations are deterministic. Complex floating point calculations, especially where different replicas run on different CPUs, have code built with different compilers, may not be[3](#foot3). In [Physalia](https://www.usenix.org/system/files/nsdi20-paper-brooker.pdf) we didn’t support floating point, because this was too hard to think about.
- _Bug fixes_. Say the code that turns _state_ and _input_ into _new state_ has a bug. How do you fix it? You can’t just change it and then roll it out incrementally to different replicas. You don’t want to deploy all your replicas at once (we’re trying to build an HA system, remember?) So you need to come up with a migration strategy. Maybe a flag sequence number. Or complex migration code that changes _buggy new state_ into _good new state_. Possible, but hard.
- _Code updates_. Are you sure that version _N+1_ produces exactly the same output as version _N_ for all inputs? You shouldn’t be, because even in the well-specified world of cryptography [that’s not always true](https://hdevalence.ca/blog/2020-10-04-its-25519am).
- _Corruption_. In reality, _input_ isn’t just _input_, it’s also a constant stream of failing components, thermal noise, cosmic rays, and other similar assaults on the castle of determinism. Can you survive them all?

And more. There’s always more.

Some people will tell you that you can solve these problems by using _byzantine_ consensus protocols. Those people are right, of course. They’re also the kind of people who solved their rodent problem by keeping a leopard in their house. Other people will tell you that you can solve these problems with blockchain. Those people are best ignored.

**Monitoring and Control**

> Although using a single, centralized, server is the simplest way to implement a service, the resulting service can only be as fault tolerant as the processor executing that server. If this level of fault tolerance is unacceptable, then multiple servers that fail independently must be used. - Fred Schneider (from [Implementing Fault-Tolerant Services Using the State Machine Approach: A Tutorial](https://www.cs.cornell.edu/fbs/publications/SMSurvey.pdf))

The whole point of building a highly-available distributed system is to exceed the availability of a single system. If you can’t do that, you’ve added a bunch of complexity for nothing.

> Complex systems run in degraded mode. - Richard Cook (from [How Complex Systems Fail](https://how.complexsystems.fail/))

Depending on what you mean by _failed_, distributed systems of _f+1_, _2f+1_ or _3f+1_ nodes can entirely hide the failure of _f_ nodes from their clients. This, combined with a process of repairing failed nodes, allows us to build highly-available systems even in the face of significant failure rates. It also leads directly to one of the traps of building a distributed system: clients can’t tell the difference between the case where an outage is _f_ failures away, and where it’s just one failure away. If a system can tolerate _f_ failures, then _f-1_ failures may look completely healthy.

Consensus systems cannot be monitored entirely _from the outside_ (see [why must systems be operated?](//brooker.co.za/blog/2016/01/03/correlation.html)). Instead, monitoring needs to be deeply aware of the implementation details of the system, so it can know when nodes are healthy, and can be replaced. If they choose the wrong nodes to replace, disaster will strike.

> Control planes provide much of the power of the cloud, but their privileged position also means that they have to act safely, responsibly, and carefully to avoid introducing additional failures. - Brooker, Chen, and Ping (from [Millions of Tiny Databases](https://www.usenix.org/system/files/nsdi20-paper-brooker.pdf))

**Do You Really Need Strong Consistency?**

> It is possible to provide high availability and partition tolerance, if atomic consistency is not required. - Gilbert and Lynch

The typical state-machine implementation of consensus provides a strong consistency property called _linearizability_. In exchange, it can’t be available for all clients during a network partition. That’s probably why you chose it.

Is that why you chose it? Do you need _linearizability_? Or would something else, like _causality_ be enough? Using consensus when its properties aren’t really needed is a mistake a lot of folks seem to make. Service discovery, configuration distribution, and similar problems can all be handled adequately without strong consistency, and using strongly consistent tools to solve them makes systems less reliable rather than more. Strong consistency is not better consistency.

**Conclusion**

Despite these challenges, consensus is an important building block in building highly-available systems. Distribution [makes building HA systems easier](http://brooker.co.za/blog/2020/01/02/why-distributed.html). It’s a tool, not a solution.

Think of using consensus in your system like getting a puppy: it may bring you a lot of joy, but with that joy comes challenges, and ongoing responsibilities. There’s a lot more to dog ownership than just getting a dog. There’s a lot more to high availability than picking up a Raft library off github.

**Footnotes**

1. Including [Raft](https://raft.github.io/raft.pdf), which has become famous for being a more understandable consensus algorithm. [Virtual Synchrony](https://www.cs.rutgers.edu/~pxk/417/notes/virtual_synchrony.html) is less famous, but no less a contribution.
2. There are some nice patterns for building deterministic high-performance systems, but the general problem is still an open area of research. For a good primer on determinism and non-determinism in database systems, check out [The Case for Determinism in Database Systems](http://paperhub.s3.amazonaws.com/878608b83ccf413ea73acfd6b78860a1.pdf) by Thomson and Abadi.
3. Bruce Dawson has an [excellent blog post](https://randomascii.wordpress.com/2013/07/16/floating-point-determinism/) on the various issues and challenges.
4. Bailis et al’s [Highly Available Transactions: Virtues and Limitations](https://dsf.berkeley.edu/papers/vldb14-hats.pdf) paper contains a nice breakdown of the options here, and Aphyr’s post on [Strong Consistency Models](https://aphyr.com/posts/313-strong-consistency-models) is a very approachable breakdown of the topic. If you really want to go deep, check out Dziuma et al’s [Survey on consistency conditions](https://projects.ics.forth.gr/tech-reports/2013/2013.TR439_Survey_on_Consistency_Conditions.pdf)
