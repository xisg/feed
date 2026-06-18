---
title: '[$] Some buffer-heads cleanup work'
url: https://lwn.net/Articles/1077767/
published: "2026-06-17T14:05:04Z"
feed: lwn
guid: https://lwn.net/Articles/1077767/
---

# [$] Some buffer-heads cleanup work

Jan Kara has been [working
on cleaning up](https://lwn.net/ml/all/20260326082428.31660-1-jack@suse.cz/) how [buffer
heads](https://www.kernel.org/doc/html/v7.1-rc7/filesystems/buffer.html) are used by some kernel filesystems. In a short filesystem-track session at the 2026 [Linux Storage,
Filesystem, Memory Management, and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/), he gave an update on that work and where it is headed. Topics included generic infrastructure to track buffer heads for metadata, a buffer-head cleanup for the Amiga filesystem, and some planned locking fixes.
