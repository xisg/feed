---
title: '[$] Suspending and resuming BPF programs'
url: https://lwn.net/Articles/1076210/
published: "2026-06-19T15:55:23Z"
feed: lwn
guid: https://lwn.net/Articles/1076210/
---

# [$] Suspending and resuming BPF programs

BPF programs can be used to extend many aspects the Linux kernel, but BPF programs must run to completion in the same context that they began. Kumar Kartikeya Dwivedi is working on changing that by allowing BPF programs to be expressed as coroutines. He spoke about his work at the 2026 [Linux Storage, Filesystem, Memory-Management and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/). While still experimental, the change promises to make long-running BPF tasks significantly easier to write.
