---
title: The 7.1 kernel has been released
url: https://lwn.net/Articles/1077758/
published: "2026-06-14T18:47:53Z"
feed: lwn
guid: https://lwn.net/Articles/1077758/
---

# The 7.1 kernel has been released

Linus has [released the 7.1 kernel](https://lwn.net/Articles/1077814/).
"So it's only Sunday morning back home, but it's Sunday afternoon where
I am right now, so I'm doing the 7.1 release at the regular time -
just not in the regular timezone."

Significant changes in 7.1 include
the removal of support for some old 486-based architectures,
some [new `clone()` flags](https://lwn.net/Articles/1059673/) making
process management easier,
[BPF support](https://lwn.net/Articles/1062286/) for io\_uring,
zero-copy-I/O support for the [ublk](https://docs.kernel.org/block/ublk.html) user-space block
driver,
initial (incomplete) [sub-scheduler support](https://lwn.net/Articles/1056014/)
in sched\_ext,
more [swapping improvements](https://lwn.net/Articles/1057102/),
a [completely rewritten NTFS\
implementation](https://lwn.net/Articles/1055062/#ntfs),
and much more. See the LWN merge-window summaries ( [part 1](https://lwn.net/Articles/1067250/), [part 2](https://lwn.net/Articles/1067785/)) for details.
