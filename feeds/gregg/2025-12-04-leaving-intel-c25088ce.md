---
title: Leaving Intel
url: http://www.brendangregg.com/blog//2025-12-05/leaving-intel.html
published: "2025-12-04T13:00:00Z"
feed: gregg
guid: http://www.brendangregg.com/blog//2025-12-05/leaving-intel.html
---

# Leaving Intel

[![](/blog/images/2022/brendan_intel_banner01.jpg)](https://youtu.be/PIJL2BGjL-4?si=4YcgtkncF1qktB37&t=3536)

_InnovatiON 2022_

[![](/blog/images/2024/firstAIflamegraph-thumb.jpg)](/blog/2024-10-29/ai-flame-graphs.html)

_AI Flame Graphs_

[![](/blog/images/2025/flamescopes1.png)](/blog/2025-05-01/doom-gpu-flame-graphs.html)

_GPU Flame Scope_

[![](/blog/images/2025/Harshad_Brendan_Innovation2022-crop.jpeg)](https://www.linkedin.com/posts/harshad-sane-56711a11_thrilled-to-meet-in-person-with-brendan-gregg-activity-6980768411198922753-Dw_k?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAA8VAMBqJml4viT3EVYGfzv-hLOE0rjdIE)

_Harshad Sane_

[![](/blog/images/2023/SREcon2023_IMG_0071thumb.jpg)](/blog/images/2023/SREcon2023_IMG_0071.jpg)

_SREcon APAC_

![](/blog/images/2025/CloudTeams_titleonly.png)

_Cloud strategy_

[![](/blog/images/2025/brendanoffice2025dec-thumb.jpg)](/blog/images/2025/brendanoffice2025dec.jpg)

_Last day_

I've resigned from Intel and accepted a new opportunity. If you are an Intel employee, you might have seen my fairly long email that summarized what I did in my 3.5 years. Much of this is public:

- [AI flame graphs](/blog/2024-10-29/ai-flame-graphs.html) and released them as [open source](https://github.com/intel/iaprof)
- [GPU subsecond-offset heatmap](/blog/2025-05-01/doom-gpu-flame-graphs.html)
- Worked with Linux distros to enable [stack walking](/blog/2024-03-17/the-return-of-the-frame-pointers.html)
- Was interviewed by the WSJ about eBPF for [security monitoring](https://www.wsj.com/articles/how-the-crowdstrike-tech-outage-reignited-a-battle-over-the-heart-of-microsoft-systems-72b62c90?utm_source=chatgpt.com)
- Provided leadership on the eBPF Technical Steering Committee ( [BSC](https://ebpf.foundation/bsc/))
- Co-chaired USENIX [SREcon APAC 2023](/blog/2023-02-17/srecon-apac-2023.html)
- Gave 6 conference keynotes

It's still early days for AI flame graphs. Right now when I browse CPU performance case studies on the Internet, I'll often see a CPU flame graph as part of the analysis. We're a long way from that kind of adoption for GPUs (and it doesn't help that our open source version is Intel only), but I think as GPU code becomes more complex, with more layers, the need for AI flame graphs will keep increasing.

I also supported cloud computing, participating in 110 customer meetings, and created a company-wide strategy to win back the cloud with 33 specific recommendations, in collaboration with others across 6 organizations. It is some of my best work and features a visual map of interactions between all 19 relevant teams, described by Intel long-timers as the first time they have ever seen such a cross-company map. (This strategy, summarized in a slide deck, is internal only.)

I always wish I did more, in any job, but I'm glad to have contributed this much especially given the context: I overlapped with Intel's toughest 3 years in history, and I had a hiring freeze for my first 15 months.

My fond memories from Intel include
meeting [Linus](/blog/images/2022/Brendan_Linus_Sep2022_01crop.jpg) at an Intel event who said "everyone is using _fleme_ graphs these days" (Finnish accent),
meeting Pat Gelsinger who knew about my work and introduced me to everyone at an exec all hands,
surfing lessons at an Intel Australia and HP offsite ( [mp4](/blog/images/2024/Intel_HP_Urbnsurf_2024.mp4)),
and meeting [Harshad Sane](https://www.linkedin.com/posts/harshad-sane-56711a11_thrilled-to-meet-in-person-with-brendan-gregg-activity-6980768411198922753-Dw_k?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAA8VAMBqJml4viT3EVYGfzv-hLOE0rjdIE) (Intel cloud support engineer) who helped me when I was at Netflix and now has joined Netflix himself -- we've swapped ends of the meeting table. I also enjoyed meeting Intel's hardware fellows and senior fellows who were happy to help me understand processor internals. (Unrelated to Intel, but if you're a Who fan like me, I recently met some other [people](/blog/images/2025/brendan_mccoy.jpg) [as](/blog/images/2025/brendan_hines.jpg) [well](/blog/images/2025/brendan_ford.jpg)!)

My next few years at Intel would have focused on execution of those 33 recommendations, which Intel can continue to do in my absence. Most of my recommendations aren't easy, however, and require accepting change, ELT/CEO approval, and multiple quarters of investment. I won't be there to push them, but other employees can (my CloudTeams strategy is in the inbox of various ELT, and in a shared folder with all my presentations, code, and weekly status reports). This work will hopefully live on and keep making Intel stronger. Good luck.
