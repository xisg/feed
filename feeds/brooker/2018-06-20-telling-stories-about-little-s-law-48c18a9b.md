---
title: Telling Stories About Little's Law
url: http://brooker.co.za/blog/2018/06/20/littles-law.html
published: "2018-06-20T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2018/06/20/littles-law
---

# Telling Stories About Little's Law

Building Up Intuition with Narrative

[Little’s Law](https://en.wikipedia.org/wiki/Little's_law) is widely used as a tool for understanding the behavior of distributed systems. The law says that the mean concurrency in the system (𝐿) is equal to the mean rate at which requests arrive (λ) multiplied by the mean time that each request spends in the system (𝑊):

𝐿 = λ𝑊

As I’ve [written about before](//brooker.co.za/blog/2017/12/28/mean.html), Little’s law is useful because it gives us a clear way to reason about the capacity of a system, which is often difficult to observe directly, based on quantities like arrival rate (requests per second) and latency which are easier to measure directly. Concurrency is a useful measure of capacity in real systems, because it directly measures consumption of resources like threads, memory, connections, file handles and anything else that’s numerically limited. It also provides an indirect way to think about contention: if the concurrency in a system is high, then it’s likely that contention is also high.

I like Little’s Law as a mathematical tool, but also as a narrative tool. It provides a powerful way to frame stories about system behavior.

### Feedback

The way Little’s Law is written, each of the terms are long-term averages, and λ and 𝑊 are independent. In the real world, distributed systems don’t tend to actually behave this nicely.

Request time (𝑊) tends to increase as concurrency (𝐿) increases. [Amdahl’s Law](https://en.wikipedia.org/wiki/Amdahl%27s_law) provides the simplest model of this: each request has some portion of work which is trivially parallelizable, and some portion of work that is forced to be serialized in some way. Amdahl’s law is also wildly optimistic: most real-world systems don’t see throughput level out under contention, but rather see throughput drop as contention rises beyond some limit. The [universal scalability law](http://www.perfdynamics.com/Manifesto/USLscalability.html) captures one model of this behavior. The fundamental reason for this is that contention itself has a cost.

Even in the naive, beautiful, Amdahl world, latency increases as load increases because throughput starts to approach some maximum. In the USL world, this increase can be dramatically non-linear. In both cases 𝑊 is a function of 𝐿.

Arrival rate (λ) also depends on request time (𝑊), and typically in a non-linear way. There are three ways to see this relationship:

- Arrival rate drops as request time increases (λ ∝ 1/𝑊). In this model there is a finite number of clients and each has its own finite concurrency (or, in the simplest world is calling serially in a loop). As each call goes up, clients keep their concurrency fixed, so the arrival rate drops.
- Arrival rate does not depend on latency. If clients don’t change their behavior based on how long requests take, or on requests failing, then there’s no relationship. The widely-used Poisson process client model behaves this way.
- Arrival rate increases as request time increases (λ ∝ 𝑊). One cause of this is _timeout and retry_: if a client sees a request exceed some maximum time (so 𝑊>𝜏) they may retry. If that timeout 𝜏 is shorter than their typical inter-call time, this will increase the per-client rate. Other kinds of stateful client behavior can also kick in here. For example, clients may interpret long latencies as errors that don’t only need to be retried, but can trigger entire call chains to be retried.

The combination of these effects tends to be that the dynamic behavior of distributed systems has scary cliffs. Systems have plateaus where they behave well where 𝑊, 𝐿 and λ are either close-to-independent or inversely proportional, and cliffs where direct proportionality kicks in and they spiral down to failure. Throttling, admission control, back pressure, backoff and other mechanisms can play a big role in avoiding these cliffs, but they still exist.

### Arrival Processes and Spiky Behavior

The mean, [like all descriptive statistics](//brooker.co.za/blog/2017/12/28/mean.html), doesn’t tell the whole story about data. The mean is very convenient in the mathematics of Little’s law, but tends to hide effects caused by high-percentile behavior. Little’s law’s use of long-term means also tends to obscure the fact that real-world statistical processes are frequently non-stationary: they include trends, cycles, spikes and seasonality which are not well-modeled as a single stationary time series. Non-stationary behavior can affect 𝑊, but is most noticeable in the arrival rate λ.

There are many causes for changes in λ. Seasonality is a big one: the big gift-giving holidays, big sporting events, and other large correlated events can significantly increase arrival rate during some period of time. Human clients tend to exhibit significant daily, weekly, and yearly cycles. People like to sleep. For many systems, though, the biggest cause of spikes is the combination of human biases and computer precision: _cron_ jobs. When humans pick a time for a task to be done ( _backup once a day_, _ping once a minute_), they don’t tend to pick a uniformly random time. Instead, they cluster the work around the boundaries of months, days, hours, minutes and seconds. This leads to significant spikes of traffic, and pushes the distribution of arrival time away from the Poisson process ideal[1](#foot1).

Depending on how you define _long term mean_, these cyclic changes in λ can either show up in the distribution of λ as high percentiles, or show up in λ being non-stationary. Depending on the data and the size of the spikes it’s still possible to get useful results out of Little’s law, but they will be less precise and potentially more misleading.

### Telling Stories

Somewhat inspired by Little’s law, we can build up a difference equation that captures more of real-world behavior:

Wn+1 = 𝑊(Ln, λn, t)

λn+1 = λ(Ln, Wn, t)

Ln+1 = λn+1 𝑊n+1

I find that this is a powerful mental model, even if it’s lacking some precision and is hard to use for clean closed-form results. Breaking the behavior of the system down into time steps provides a way to tell a story about the way the system behaves in the next time step, and how the long-term behavior of the system emerges. It’s also useful for building simple simulations of the dynamics of systems.

Telling stories about our systems, for all its potential imprecision, is a powerful way to build and communicate intuition.

_The system was ticking along nicely, then just after midnight a spike of requests from arrived from a flash sale. This caused latency to increase because of increased lock contention on the database, which in turn caused 10% of client calls to time-out and be retried. A bug in backoff in our client meant that this increased call rate to 10x the normal for this time of day, further increasing contention._ And so on…

Each step in the story evolves by understanding the relationship between latency, concurrency and arrival rate. The start of the story is almost always some triggering event that increases latency or arrival rate, and the end is some action or change that breaks the cycle. Each step in the story offers an opportunity to identify something to make the system more robust. Can we reduce the increase in 𝑊 when λ increases? Can we reduce the increase in λ when 𝑊 exceeds a certain bound? Can we break the cycle without manual action?

The typical resiliency tools, like backoff, backpressure and throttling, are all answers to these types of questions, but are far from the full set of answers. Telling the stories allows us to look for more answers.

### Footnotes

1. Network engineers have long known that the Poisson model is less bursty than many real systems. [An Empirical Workload Model for Driving Wide-Area TCP/IP Network Simulations](https://pdfs.semanticscholar.org/3226/e025b4ab4afa664b2c9b0418227ee76ac13c.pdf) and [Wide Area Traffic: The Failure of Poisson Modeling](http://cs.uccs.edu/~cchow/pub/master/xhe/doc/p226-paxson-floyd.pdf) are classics in that genre. I’m not aware of good research on this problem in microservice or SoA architectures, but I’m sure there are some interesting results to be found there.
