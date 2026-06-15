---
title: No More Blue Fridays
url: http://www.brendangregg.com/blog//2024-07-22/no-more-blue-fridays.html
published: "2024-07-21T14:00:00Z"
feed: gregg
guid: http://www.brendangregg.com/blog//2024-07-22/no-more-blue-fridays.html
---

# No More Blue Fridays

In the future, computers will not crash due to bad software updates, even those updates that involve kernel code. In the future, these updates will push [eBPF](https://ebpf.io/) code.

Friday July 19th provided an unprecedented example of the inherent dangers of kernel programming, and has been called the largest outage in the [history](https://en.m.wikipedia.org/wiki/2024_CrowdStrike_incident#Cost) of information technology. Windows computers around the world encountered blue-screens-of-death and boot loops, causing [outages](https://www.washingtonpost.com/technology/2024/07/19/microsoft-windows-outage-blue-screen-bsod/) for hospitals, airlines, banks, grocery stores, media broadcasters, and more. This was caused by a [config update](https://www.crowdstrike.com/blog/falcon-update-for-windows-hosts-technical-details/) by a security company for their widely used product that included a kernel driver on Windows systems. The update caused the kernel driver to [try to read invalid memory](https://medium.com/@shyamsundarb/technical-details-of-the-windows-bsod-disaster-due-to-crowdstrike-58de2371c19c), an error type that will crash the kernel.

For Linux systems, the company behind this outage was already in the process of adopting eBPF, which is immune to such crashes. Once Microsoft's [eBPF support for Windows](https://github.com/microsoft/ebpf-for-windows) becomes production-ready, Windows security software can be ported to eBPF as well. These security agents will then be safe and unable to cause a Windows kernel crash.

eBPF (no longer an acronym) is a secure kernel execution environment, similar to the secure JavaScript runtime built into web browsers. If you're using Linux, you likely already have eBPF available on your systems whether you know it or not, as it was included in the kernel several years ago. eBPF programs cannot crash the entire system because they are safety-checked by a software verifier and are effectively run in a sandbox. If the verifier finds any unsafe code, the program is rejected and not executed. The verifier is rigorous -- the Linux implementation has [over 20,000 lines of code](https://github.com/torvalds/linux/blob/master/kernel/bpf/verifier.c) \-\- with contributions from industry (e.g., Meta, Isovalent, Google) and academia (e.g., [Rutgers University](https://people.cs.rutgers.edu/~sn624/verified-sandboxing.html), [University of Washington](https://unsat.cs.washington.edu/projects/jitterbug/)). The safety this provides is a key benefit of eBPF, along with heightened security and lower resource usage.

Some eBPF-based security startups (e.g., [Oligo](https://www.oligo.security/blog/recent-crowdstrike-outage-emphasizes-the-need-for-ebpf-based-sensors), [Uptycs](https://www.uptycs.com/blog/crowdstrike-outage-less-intrusive-malware-detection)) have made their own statements about the recent outage, and the advantages of migrating to eBPF. Larger tech companies are also adopting eBPF for security. As an example, Cisco acquired the eBPF-startup Isovalent and has announced a new eBPF security product: [Cisco Hypershield](https://blogs.cisco.com/security/cisco-hypershield-reimagining-security), a fabric for security enforcement and monitoring. [Google](https://www.youtube.com/watch?v=N4YKcMV8iaY) and [Meta](https://lpc.events/event/17/contributions/1602/) already rely on eBPF to detect and stop bad actors in their fleet, thanks to eBPF's speed, deep visibility, and safety guarantees. Beyond security, eBPF is also used for networking and observability.

The worst thing an eBPF program can do is to merely consume more resources than is desirable, such as CPU cycles and memory. eBPF cannot prevent developers writing poor code -- wasteful code -- but it will prevent serious issues that cause a system to crash. That said, as a new technology eBPF has had some bugs in its management code, including a [Linux kernel panic](https://access.redhat.com/solutions/7068083) discovered by the same security company in the news today. This doesn't mean that eBPF has solved nothing, substituting a vendor's bug for its own. Fixing these bugs in eBPF means fixing these bugs for _all_ eBPF vendors, and more quickly improving the security of everyone.

There are other ways to reduce risks during software deployment that can be employed as well: canary testing, staged rollouts, and "resilience engineering" in general. What's important about the eBPF method is that it is a software solution that will be available in both Linux and Windows kernels by default, and has already been adopted for this use case.

If your company is paying for commercial software that includes kernel drivers or kernel modules, you can make eBPF a requirement. It's possible for Linux today, and Windows soon. While some vendors have already proactively adopted eBPF (thank you), others might need a little encouragement from their paying customers. Please help raise awareness, and together we can make such global outages a lesson of the past.

_Authors: Brendan Gregg, Intel; Daniel Borkmann, Isovalent; Joe Stringer, Isovalent; KP Singh, Google._
