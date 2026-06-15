---
title: When to Hire a Computer Performance Engineering Team (2025) part 1 of 2
url: http://www.brendangregg.com/blog//2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html
published: "2025-08-03T14:00:00Z"
feed: gregg
guid: http://www.brendangregg.com/blog//2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html
---

# When to Hire a Computer Performance Engineering Team (2025) part 1 of 2

_As a leader in computer performance I've been asked by companies about how (and why) to form a performance engineering team, and as this is broadly useful I'll share my advice here._

Large tech companies in the US hire performance engineers (under that or other titles) to ensure that infrastructure costs and service latency don't grow too high, and that their service is reliable under peak load. A new performance team can likely find enough optimizations to _halve_ infrastructure spend in their first couple of years, even for companies that have been using commercial performance or observability tools. Performance engineers do much more than those tools, working with development teams and vendors to build, test, debug, tune, and adopt new performance solutions, and to find deep optimizations that those tools can miss.

I previously worked on the performance engineering team for Netflix, a large tech consumer running on hundreds of thousands of AWS instances. I'm now doing similar work at Intel (a large tech vendor) for Intel and their customers. As a leader in this space I've also interacted with other performance teams and staff doing performance work at many companies. In this post I'll explain what these teams do and when you should consider forming one. In part 2 I'll provide sample job descriptions, specialties, advice, pitfalls, comments on AI, and what to do if you can't hire a performance team.

It's easy for hardware vendors like Intel to justify hiring performance engineers, as the number one factor in sales is beating a competitor's performance. However, my focus in this post is on non-vendor tech-heavy companies who hire these staff to drive down costs and latency outliers (e.g., banks, telecomms, defence, AI, tech-based companies, and anyone else who is spending more than $1M/year on back-end compute and AI).

# What is the ROI of performance engineering?

The main ROIs are **infrastructure cost savings**, **latency reductions**, **improved scalability and reliability**, and **faster engineering**. The cost savings alone can justify a performance team and can help calculate its size, and I'll explore that in depth, but the other ROIs are worth considering and may be more important to a company depending on its stage of growth.

## Infrastructure Cost Savings and Margin Improvements

An appropriately-sized performance team should be targeting 5-10% cost savings per year through tuning and product adoptions. (I'll explain appropriate sizes in the When To Hire section.) For many large companies a 5% result would be considered "good" and a 10% would be "great." Achieving this in practice can mean finding large wins (15-80%) on parts of the infrastructure, which become 5-10% overall. Wins are cumulative, so a team hitting 5% savings each year will multiply to become 28% after their 5th year (like compound interest). Even a modest 2% per year will become significant over time. While these compounded numbers can become large, a team needs to continue finding new cost savings each year to justify long-term retention, and should always be focused on the _next_ 5-10%.

![](/blog/images/2025/cumulative-cost-savings.png)

Companies may invest in this work for more than just the cost savings: It can be about **developing a competitive advantage** in their area by providing a better cost/performance ratio, especially for companies with similar tech-based services that pass costs on to customers.

For sites that haven't employed performance engineers before, there can be enough low-hanging fruit that the team can halve infrastructure costs in their first couple of years (50%). It all depends on the number of staff, their level of expertise, how much perf work other staff are already doing (senior developers, SREs), how much custom code is running, and how complex and volatile the stack is.

It would great if we could publicly share specific results, which would look something like this:

"This year we helped reduce our company's infrastructure spend by 5%, from $60M/year to $57M/year, however, since our user base also grew by 3%, we actually **reduced cost-per-user by 8%**, saving $5M/year in this and all future years."

However, these numbers are usually considered financially sensitive as they can reveal company growth, financial health, confidential infrastructure discounts, etc. As a performance engineer I can talk publicly about percent wins on a back-end service, but I usually can't map it to dollar signs. That doesn't help other companies to understand the value of performance engineering. It's not so much of a problem in Silicon Valley, since staff change companies all the time and word spreads about the latest practices in tech. But in far away countries performance engineering doesn't really exist yet, even though there are companies with sufficiently large infrastructure spend.

Continuing the above example, a typical 8% win could be composed of:

- **2%: direct optimizations**. We looked across all the infrastructure and found on average 5% wins on 40% of the infrastructure (on the rest we found nothing, this year).
- **3%: developer/SRE enablement**. We developed and adopted custom observability tools and helped developers use them, who in turn found some 15% wins across 20% of our infrastructure.
- **3%: vendor adoptions**. We supported a major adoption project that replaced 10% of our infrastructure for a 30% win.

With developer/SRE enablement and vendor adoptions, the performance team isn't finding the wins directly but is enabling other teams and vendors to do so. For example, when I worked at Netflix we built and maintained the flame graph "self-service" application, which developers used daily to find wins, and we worked on multiple product adoptions every year. This all needs to be considered as part of the performance team’s ROI.

## Latency Reductions

Reducing the response time or latency of a service is a large part of performance engineering. This involves analyzing average latency, 99th percentile latency, and outlier latency; ensuring latency SLA/SLOs are met; and ensuring acceptable latency during perturbations or peak usage.

Many of the cost optimizations described earlier will also reduce average latency, but latency variance or outliers can remain. For example, a once-every-5-minute system task may have negligible cost and CPU footprint, but it may briefly perturb the application and cause latency outliers. These are debugged differently, often using monitoring, logs, distributed tracing, system-level tracing, packet logs, and custom ad-hoc tools. Sometimes the high latency is caused by the request type itself (the system is fine, but the end-user has requested a slow thing) or is an expected consequence of load (queueing theory, tail latency). Other times it can be from complex interactions across multiple layers of the software stack or from interactions across multiple network endpoints.

As a related aside: One performance anti-pattern is when a company, to debug one performance problem, installs a monitoring tool that periodically does work and causes application latency outliers. Now the company has two problems. Tip: try turning off all monitoring agents and see if the problem goes away.

While latency is the main consideration to improve end user experience, others include throughput and parallelism.

## Improved Scalability and Reliability

Systems under load can respond with exponential latency or a cascading failure, causing disruptions or a service outage. Performance engineers can test resource scalability with custom load generators and benchmarks, and use analysis tools to study all parts of the system to find and solve bottlenecks. A performance engineer will not just measure scalability limits, but should also explain what the limiting factors are and how to address them to scale further.

A stable and performant service will also earn trust in your company, and can help you grow customers more quickly. It may be a requirement for satisfying enterprise SLA/SLOs.

I'll share a scalability story from my time at Sun Microsystems (a vendor). My goal was to achieve the number one throughput in the industry for a storage system, which would require exceeding 1M IOPS. The expected bottleneck was the rotational disks. I developed my own load generators and analysis tools and concluded that the real bottleneck was, surprisingly, the CPU interconnect. The interconnect was AMD HyperTransport 1, so AMD sent me a new systemboard with HT3 and faster CPUs. I installed it and…performance was identical. I was upset with myself for getting it wrong, until I discovered that AMD had sent me a HT1 board by mistake. They then sent me a real HT3 board and the performance increased by up to 75%! The CPU interconnect (when present) is just one of many components that companies typically don't check, and commercial observability tools don't check either.

## Faster Engineering

Performance engineers can take care of components outside of a developer's code base so the developers can stay focused, and also provide them with a greater performance budget so that they can adopt expensive features earlier. For some early stage companies this ROI may be their most important (and is sometimes called _engineering velocity_). In detail:

- **Eliminate outside performance distractions**: A development team can be coding at one hundred miles an hour in their codebase, but then run into a performance problem in a library, kernel, hypervisor, or hardware component, and need to slow down to learn some new thing and its analysis tools. Or, it could be some new performance technology that someone saw on hackernews and the development team wonder if they should stop to evaluate it. These performance activities can be offloaded to the perf team so the developers stay focused on their code and at full speed. This is the same as how OS, deployment, and reliability issues can be offloaded to SRE and DevOps teams.
- **Bypass expensive project failures**: Good performance engineers know the internals of the software and hardware stack (including the Linux kernel). Many developers don't and most of the time they don't need to. But this can lead to developers proposing ideas based on incorrect assumptions. (This actually happens a lot.) Having a one hour meeting with a performance engineering team can save months of engineering effort: A developer talks about their ideas A and B, and the perf team says "A won't work and this is why, but B sounds good and we can help." Months saved in one hour.

  At one large tech company many years ago I analyzed an engineering proposal to adopt a new event-based coding framework on the belief it would improve performance by _ten fold_. My analysis showed the real gain would have been _less than 10%_. The project was abandoned. This saved having all the engineers rewrite all the company's code, something we were expecting would take a year. This is one of the biggest performance wins of my career, not measured as a percentage but rather as engineering hours saved.
- **Develop accelerator tools**: As mentioned earlier, performance teams can develop custom observability tools that developers use, finding issues sooner and accelerating development. For example, I've been publishing here about AI flame graphs, which involved kernel and hardware internals to build.
- **Faster feature adoption**: Reducing costs and latency can enable developers to do more with a limited infrastructure and latency budget, offering more capabilities for their service or allowing more processing to be crammed in per request. Some companies have cost limits for adding features, so optimization work can mean a feature is now approved. All this work can mean a company can outpace their competitors with feature adoption, while maintaining a reliable, low-latency, cost/performance competitive service.

# What do performance engineers do?

For non-vendor tech companies, in summary:

**A. Test, debug, and tune new software and hardware products to find performance improvements, and drives company-wide adoption.**

Examples: New cloud instance types, language runtimes, JVM versions, JVM subsystems (new GC algorithms or compilers: Graal vs c2), system libraries (glibc vs tcmalloc etc.), kernels (Linux vs BSD) and versions, compilers (gcc, llvm, icc), processor features (AVX, QAT, etc.), hardware accelerators, and so on. It can take months to debug, fix, and patch everything so the latest thing delivers its performance claim.

**B. Develop in-house performance solutions, such as custom analysis tools, that other teams use to find performance wins.**

Examples: Custom monitoring using Prometheus and Grafana, one-click flame graphs, and analysis tools using eBPF: All of this is open-source based, but someone has to get it working locally, integrate them with existing local tools, teach other teams how to use them, and maintain them.

**C. Does deep-dive analysis to identify and reduce workload bottleneck(s) and latency outliers.**

Examples: Using code profilers (CPU flame graphs), distributed tracers (OpenTelemetry and products), application logs, system counters (Linux: sysstat), system tracers (Linux: eBPF, Ftrace, perf), static and dynamic instrumentation (Linux: kprobes, uprobes), debuggers (gdb, etc.), hardware counters (Linux: perf), and on rare occasions hardware instruction tracing. A lot of hands-on live debugging over an SSH session, following methodologies to efficiently find the root-cause(s), which can require the development of custom tools (mini load generators, observability tools, etc.).

**D. Optimize software and hardware via tunable parameters and configuration choices.**

Examples: System tunables (Linux: sysctls), network tunables (socket options, qdiscs), device tunables, runtime tunables (Java -XX:\*), library settings, environment variables, etc. As with (C), the team needs SSH access to do this and likely superuser privileges.

**E. Work with development teams (internal and external) to catch non-scalable solutions early in development, and to suggest or test later performance improvements.**

Examples: Identifying communication layer will flood network links when horizontally scaled; A developer has a good optimization idea but can't get it to work and needs some help; There's a performance-related pull request on some software the company uses but the request is two years old and needs someone to fix code conflicts, test it, and advocate for merging it.

**F. Develop proof-of-concept demonstrations of new performance technologies.**

Examples: Linux eBPF and io\_uring can provide significant performance improvements when developed into hot-path kernel-based accelerators, but someone needs to at least build a POC to show it would work for the company. These are typically too esoteric for developers to try on their own.

**G. Develop performance improvements directly for internal and external code.**

Examples: Performance engineers get a lot done by asking the right people, but sometimes no one has the time to code that Linux/runtime/database performance fix the company needs, so a perf engineer takes it on. We aren't as quick as full-time developers since we are hopping between different languages all the time, and as a new code base committer will typically come under extra (and time-consuming) scrutiny.

**H. Capacity planning activities: purchase guidance, choosing metrics to monitor, and bottleneck forecasting.**

Examples: Modeling and performance characterization for hardware purchases, resource utilization monitoring to forecast capacity issues (nowadays often done by developers and SREs using monitoring tools); propose the best metrics to be watched in those monitoring tools for alert generation and auto-scaling rules; work with business side of the company to help define practical SLA/SLOs.

**I. Perform knowledge sharing to uplift engineering.**

Examples: Performance education to help developers produce more efficient software; act as a conduit to share performance learnings between teams (that may otherwise be siloed) to avoid rework and rediscovery.

**J. Provide in-house expertise to guide purchasing performance solutions.**

Examples: Providing in-house expertise for performance topics like observability, telemetry, and eBPF can help the company choose better commercial products by evaluating their capabilities and overhead costs, and can recognize which are just Prometheus and Grafana, or my open source eBPF tools, in a suit. Without expertise you're vulnerable to being ripped off, or may adopt a tool that increases infrastructure costs more than the gains it provides (I've seen some that have overhead exceeding 10%).

To elaborate on (A), the testing of new products: Other staff will try a technology by configuring it based on the README, run a load test, and then share the result with management. Some companies hire dedicated staff for this called "performance testers." Performance engineers get more out of the same technology by running analyzers during the test to understand its limiter ("active benchmarking"), and will tune the technology to get an extra 5%, 50%, or more performance. They may also discover that the limiter is an unintended target (e.g., accidentally testing a caching layer instead). Any performance test should be accompanied by an explanation of the limiting factor, since no explanation will reveal the test wasn't analyzed and the result may be bogus. You can simply ask "why is the result not double?".

As an aside: "CPU bound" isn't an explanation. Do you mean (a) clockspeed, (b) thread pool size, (c) core count, (d) memory bus (which kernels misleadingly include in %CPU counters), or something else (like power, thermal, CPU subsystem bottleneck)? Each of those leads to a different actionable item for the company (E.g.: (a) faster processors; (b) more threads; (c) more cores; (d) faster memory, bigger caches, less NUMA, or software techniques like zero copy). That's just the generic stuff. The code behind any CPU bound workload will also be analyzed to look for inefficiencies, and sometimes their instructions as well.

Day to day, a performance engineer can spend a lot of time fixing broken builds and configuring workloads, because you're the first person testing new patches and bleeding-edge software versions.

What I've described here is for companies that consume tech. For vendors that sell it, performance engineering includes design modeling, analysis of prototype and in-development software and hardware, competitive benchmarking, non-regression testing of new product releases, and pre- and post-sales performance analysis support. (I explain this in more detail in Systems Performance 2nd edition, chapter 1.)

# When to Hire a Performance Team and How Many

Most companies I've encountered are already doing some kind of performance work scattered across projects and individuals, but they don't yet have a central performance engineering team looking deeply at everything. This leaves their attention spotty, ok in some areas, poor to absent in others. A central performance team looks at everything and prioritizes work based on the potential ROI.

Here are a few rough rules to determine when you should start forming a company-wide performance engineering team and how to size it (see caveats at the end):

## **(A) One engineer at $1M/year infrastructure spend, then one per $10M to $20M/year**

That first engineer finds some of the low-hanging fruit, and should be cost effective as your company grows past $1M/year. I'd then consider another performance engineer for every $10M to $20M, and maintain a 3:1 junior:senior ratio. The values you use depend on your performance engineer's skill and the complexity of your environment, and how aggressively you wish to improve performance. At a $20M spend, 5% yearly wins means $1M in savings per staff member (minus their cost); whereas for a $10M spend you'd need to hit 10% wins yearly for $1M in savings.

Consider that as your spend keeps growing you will keep adding more staff, which makes their job harder as there is less low-hanging fruit to find. However, your site will also be growing in scale and complexity, and developing new performance issues for the growing team to solve. Also, smaller percent wins become more impactful at large scale, so I expect such a growing perf team to remain cost effective. (To a point: the largest teams I've seen stop at around 150 staff.)

## **(B) Staff spend should equal or exceed observability monitoring spend**

If you're spending $1M/year on an observability product, you can spend $1M/year on a performance engineering team: e.g., 3 to 4 good staff. If you're only spending $50k/year on an observability product, you can't hire a performance engineer at that price, but you can bring in a consultant or pay for performance training and conference attendance. As I'd expect staff to halve infrastructure costs over time, just the savings on monitoring alone (which typically scale with instance/server count) will pay for the new staff. Because these new engineers are actively reducing infrastructure spend, the total savings are much greater.

## **(C) When latency or reliability is prohibitive to growth**

I’ve heard some small companies and startups say they spend more money on coffee than they do back-end compute, and don't want to waste limited developer time on negligible cost reductions. However, when a wave of new customers arrive they may hit scalability issues and start losing customers because latency is too high or reliability is too inconsistent. That's usually a good time for small companies to start investing in performance engineering.

## **Caveats for A-C**

- **You likely already have some perf engineers under different titles**, such as senior developers or SREs who are focusing on perf work, leaving less work for the central performance team. Account for these focused staff when you are determining how many performance engineers to hire.
- **There will be a backpressure effect**: If a new team halves your infrastructure, do you then halve the team? Up to you. But first think about what a perf engineer does: They have to work on anything, any operating system, language, or hardware the company needs. So you have these extra staff that can go deep on anything. Just some personal examples: There was an 18 month stretch when Netflix needed more core SREs on rotation, so myself and other perf engineers were temporarily assigned to do SRE work; Netflix would also have me do kernel crash dump analysis and application core dump analysis, since I was already using debuggers at that level.
- **You ideally have more performance engineers than technologies** so staff members can specialize. E.g.: If your stack is AWS, Intel, Linux kernel, Ubuntu OS, containers, lambda, gRPC, Java, golang, Python, Cassandra, and TensorFlow, that's 12 performance engineers. Plus at least one for locally developed code. Want to go multi-cloud? Add another engineer for each CSP.
- **Remember that performance wins are cumulative** for future years. A novice team could struggle to get up to speed with both performance engineering and your complex environment, and could also discover that you already had senior staff find all the low-hanging fruit. They might therefore only deliver 2% cost savings in each of their first three years. But that still combines to be 6% going forward.

# Companies and Global Staff

Here are some example articles about performance engineering work at non-vendor companies:

- **[Netflix](https://www.brendangregg.com/Slides/YOW2018_CloudPerfRCANetflix/)**: _Cloud Performance Root Cause Analysis_ \[me, 2018\]
- **[Meta](https://engineering.fb.com/2025/01/21/production-engineering/strobelight-a-profiling-service-built-on-open-source-technology/)**: Strobelight: _A profiling service built on open source technology_ \[Jordan Rome, 2025\]
- **[Pinterest](https://engineering.linkedin.com/performance/who-moved-my-99th-percentile-latency)**: _Debugging the One-in-a-Million Failure: Migrating Pinterest’s Search Infrastructure to Kubernetes_ \[Samson Hu, et al. 2025\]
- **[LinkedIn](https://engineering.linkedin.com/performance/who-moved-my-99th-percentile-latency)**: _Who moved my 99th percentile latency?_ \[Richard Hsu, Cuong Tran, 2015\]
- **[eBay](https://innovation.ebayinc.com/stories/speed-by-a-thousand-cuts/)**: _Speed by a Thousand Cuts_ \[Senthil Padmanabhan, 2020\]
- **[Twitter](https://blog.x.com/engineering/en_us/topics/infrastructure/2019/expand-the-edge)**: _#ExpandTheEdge: Making Twitter Faster_ \[Todd Segal, Anthony Roberts, 2019\]
- **[Salesforce](https://engineering.salesforce.com/how-salesforce-makes-performance-engineering-work-at-enterprise-scale-8c2c1323e81d/)**: _How Salesforce makes performance engineering work at Enterprise Scale_ \[Archana Sethuraman\]
- **[Uber](https://www.uber.com/en-AU/blog/perfinsights)**: _PerfInsights: Detecting Performance Optimization Opportunities in Go Code using Generative AI_ \[Lavanya Verma, Ryan Hang, Sung Whang, Joseph Wang\]
- **[Twitch](https://blog.twitch.tv/en/2016/07/05/gos-march-to-low-latency-gc-a6fa96f06eb7/)**: _Go’s march to low-latency GC_ \[Rhys Hiltner, 2016\]
- **[Airbnb](https://medium.com/airbnb-engineering/creating-airbnbs-page-performance-score-5f664be0936)**: _Creating Airbnb’s Page Performance Score_ \[Andrew Scheuermann, 2021\]
- **[Stripe](https://stripe.com/blog/using-ml-to-detect-and-respond-to-performance-degradations-in-slices-of-stripe-payments)**: _Using ML to detect and respond to performance degradations in slices of Stripe payments_ \[Lakshmi Narayan, Lakshmi Narayan, 2025\]
- **[DoorDash](https://careersatdoordash.com/blog/how-doordash-standardized-and-improved-microservices-caching/)**: _How DoorDash Standardized and Improved Microservices Caching_ \[Lev Neiman, Jason Fan, 2023\]
- **[Roblox](https://blog.haydz6.com/2020/11/redesigned-highly-scaled-data-store-lower-99th-percentile-latency-100x)**: _How We Redesigned a Highly Scaled Data Store to Lower 99th Percentile Latency 100x_ \[Andrew Korytko, 2020\]
- **[Capital One](https://www.capitalone.com/tech/cloud/cost-optimization-best-practices)**: _Driving cloud value through cost optimization & efficiency_ \[Jerzy Grzywinski, Brent Segner, 2024\]

I would like to add **Bank of America**, **Wells Fargo**, **JPMorgan Chase**, and **CitiGroup** to this list since they have many staff with the title "performance engineer" (as you can find on LinkedIn) but it's hard to find public articles about their work. I'd also like a canonical list of central performance engineering teams, but such org chart data can also be hard to find online, and staff don't always call themselves "performance engineers." Other keywords to look out for are: _insights_, _monitoring_, and _observability_; some are just called "support engineers".

Note that there is also a lot of performance engineering done at hardware, software, and cloud vendors ( **Intel**, **AMD**, **NVIDIA**, **Apple**, **Microsoft**, **Google**, **Amazon**, **Red Hat**, etc.) not listed here, as well as at performance solution companies. In this post I just wanted to focus on non-vendor companies.

### Global Staff

I've never seen concrete data on how many people are employed worldwide in performance engineering. Here are my guesses:

- Staff identifying as performance engineers at non-vendor companies: <1,000.
- Staff identifying as performance engineers at SW/HW vendors: >10,000.
- Staff who have become focused on performance engineering work under other titles (developer, SRE, support): >100,000.
- Staff who occasionally do performance engineering work: I'd guess most developers.

It's possible LinkedIn can provide better estimates if you have enterprise access.

# Conclusion

There are many reasons to hire a performance engineering team, such as infrastructure cost savings, latency reductions, improved scalability and reliability, and faster engineering. Cost savings alone can justify hiring a team, because a team should be targeting 5-10% cost reductions every year, which over the years adds up to become significantly larger: 28%-61% savings after 5 years.

In this post I explained what performance engineers do and provided some suggested rules on hiring:

A) One engineer at >$1M infrastructure spend, then another for every $10-20M.

B) Performance staff spend should equal or exceed observability monitoring spend.

Note that you likely already have some senior developers or SREs who are focusing on perf work, reducing the number of new performance engineers you need.

I've met people who would like to work as performance engineers but their employer has no such roles (other than _performance testing_: not the same thing) despite spending millions per year on infrastructure. I hope this post helps companies understand the value of performance engineering and understand when and how many staff to hire.

Hiring good performance engineers isn't easy as it's a specialized area with a limited talent pool. In part 2 I'll discuss how to hire or train a performance engineering team and provide sample job descriptions and tips, and what to do if you can't hire a performance team.

### _Thanks_

_Thanks for the feedback and suggestions: Vadim Filanovsky (OpenAI), Jason Koch (Netflix), Ambud Sharma (Pinterest), Harshad Sane (Netflix), Ed Hunter, Deirdre Straughan._
