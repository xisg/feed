---
title: Control Planes vs Data Planes
url: http://brooker.co.za/blog/2019/03/17/control.html
published: "2019-03-17T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2019/03/17/control
---

# Control Planes vs Data Planes

Are there multiple things here?

If you want to build a successful distributed system, one of the most important things to get right is the block diagram: what are the components, what does each of them own, and how do they communicate to other components. It’s such a basic design step that many of us don’t think about how important it is, and how difficult and expensive it can be to make changes to the overall architecture once the system is in production. Getting the block diagram right helps with the design of database schemas and APIs, helps reason through the availability and cost of running the system, and even helps form the right org chart to build the design.

One very common pattern when doing these design exercises is to separate components into a _control plane_ and a _data plane_, recognizing the differences in requirements between these two roles.

### No true monoliths

The _microservices_ and _SOA_ design approaches tend to push towards more blocks, with each block performing a smaller number of functions. The _monolith_ approach is the other end of the spectrum, where the diagram consists of a single block. Arguments about these two approaches can be endless, but ultimately not important. It’s worth noting, though, that there are almost no true monoliths. Some kinds of concerns are almost always separated out. Here’s a partial list:

1. Storage. Most modern applications separate business logic from storage and caching, and talk through APIs to their storage.
2. Load Balancing. Distributed applications need some way for clients to distribute their load across multiple instances.
3. Failure tolerance. Highly available systems need to be able to handle the failure of hardware and software without affecting users.
4. Scaling. Systems which need to handle variable load may add and remove resources over time.
5. Deployments. Any system needs to change over time.

Even in the most monolithic application, these are separate components of the system, and need to be built into the design. What’s notable here is that these concerns can be broken into two clean categories: data plane and control plane. Along with the monolithic application itself, _storage_ and _load balancing_ are data plane concerns: they are required to be up for any request to succeed, and scale O(N) with the number of requests the system handles. On the other hand, _failure tolerance_, _scaling_ and _deployments_ are control plane concerns: they scale differently (either with a small multiple of N, with the rate of change of N, or with the rate of change of the software) and can break for some period of time before customers notice.

### Two roles: control plane and data plane

Every distributed system has components that fall roughly into these two roles: data plane components that sit on the request path, and control plane components which help that data plane do its work. Sometimes, the control plane components aren’t components at all, and rather people and processes, but the pattern is the same. With this pattern worked out, the block diagram of the system starts to look something like this:

![Data plane and control plane separated into two blocks](https://s3.amazonaws.com/mbrooker-blog-images/control_data_binary.png)

My colleague Colm MacCárthaigh likes to think of control planes from a control theory approach, separating the system (the data plane) from the controller (the control plane). That’s a very informative approach, and you can hear him talk about it here:

I tend to take a different approach, looking at the scaling and operational properties of systems. As in the example above, data plane components are the ones that scale with every request[1](#foot1), and need to be up for every request. Control plane components don’t need to be up for every request, and instead only need to be up when there is work to do. Similarly, they scale in different ways. Some control plane components, such as those that monitor fleets of hosts, scale with O(N/M), which _N_ is the number of requests and _M_ is the requests per host. Other control plane components, such as those that handle scaling the fleet up and down, scale with O(dN/dt). Finally, control plane components that perform work like deployments scale with code change velocity.

Finding the right separation between control and data planes is, in my experience, one of the most important things in a distributed systems design.

### Another view: compartmentalizing complexity

In their classic paper on [Chain Replication](https://www.cs.cornell.edu/home/rvr/papers/OSDI04.pdf), van Renesse and Schneider write about how chain replicated systems handle server failure:

> In response to detecting the failure of a server that is part of a chain (and, by the fail-stop assumption, all such failures are detected), the chain is reconfigured to eliminate the failed server. For this purpose, we employ a service, called the _master_

Fair enough. Chain replication can’t handle these kinds of failures without adding significant complexity to the protocol. So what do we expect of the master?

> In what follows, we assume the master is a single process that never fails.

Oh. Never fails, huh? They then go on to say that they approach this by replicating the _master_ on multiple hosts using Paxos. If they have a Paxos implementation available, then why do they just not use that and not bother with this Chain Replication thing at all? The paper doesn’t say[2](#foot2), but I have my own opinion: it’s interesting to separate them because Chain Replication offers a different set of performance, throughput, and code complexity trade offs than Paxos[3](#foot3). It is possible to build a single code base (and protocol) which handles both concerns, but at the cost of coupling these two different concerns. Instead, by making the _master_ a separate component, the chain replicated data plane implementation can focus on the things it needs to do (scale, performance, optimizing for every byte). The control plane, which only needs to handle the occasional failure, can focus on what it needs to do (extreme availability, locality, etc). Each of these different requirements adds complexity, and separating them out allows a system to compartmentalize its complexity, and reduce coupling by offering clear APIs and contract between components.

### Breaking down the binary

Say you build awesome data plane based on chain replication, and an awesome control plane ( _master_) for that data plane. At first, because of its lower scale, you can operate the control plane manually. Over time, as your system becomes successful, you’ll start to have too many instances of the control plane to manage by hand, so you build a control plane for that control plane to automate the management. This is the first way the control/data binary breaks down: at some point control planes need their own control planes. Your _controller_ is somebody else’s _system under control_.

One other way the binary breaks down is with specialization. The _master_ in the chain replicated system handles fault tolerance, but may not handle scaling, or sharding of chains, or interacting with customers to provision chains. In real systems there are frequently multiple control planes which control different aspects of the behavior of a system. Each of these control planes have their own differing requirements, requiring different tools and different expertise. Control planes are not homogeneous.

These two problems highlight that the idea of control planes and data planes may be too reductive to be a core design principle. Instead, it’s a useful tool for helping identify opportunities to reduce and compartmentalize complexity by introducing good APIs and contracts, to ensure components have a clear set of responsibilities and ownership, and to use the right tools for solving different kinds of problems. Separating the control and data planes should be a heuristic tool for good system design, not a goal of system design.

### Footnotes:

1. Or potentially with every request. Things like caches complicate this a bit.
2. It does compare Chain Replication to other solutions, but doesn’t specifically talk about the benefits of seperation. Murat Demirbas pointed out that Chain Replication’s ability to serve linearizable reads from the tail is important. He also pointed me at the [Object Storage on CRAQ](https://www.usenix.org/legacy/event/usenix09/tech/full_papers/terrace/terrace.pdf) paper, which talks about how to serve reads from intermediate nodes. Thanks, Murat!
3. For one definition of Paxos. Lamport’s [Vertical Paxos](https://www.microsoft.com/en-us/research/publication/vertical-paxos-and-primary-backup-replication/#) paper sees chain replication as a flavor of Paxos, and more recent work by Heidi Howard et al on [Flexible Paxos](https://arxiv.org/pdf/1608.06696v1.pdf) makes the line even less clear.
