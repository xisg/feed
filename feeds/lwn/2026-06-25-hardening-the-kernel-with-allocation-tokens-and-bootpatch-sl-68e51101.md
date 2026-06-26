---
title: '[$] Hardening the kernel with allocation tokens and bootpatch-SLR'
url: https://lwn.net/Articles/1078699/
published: "2026-06-25T14:02:39Z"
feed: lwn
guid: https://lwn.net/Articles/1078699/
---

# [$] Hardening the kernel with allocation tokens and bootpatch-SLR

There is a lot of work going into eliminating exploitable bugs from the kernel and preventing the addition of new ones. Even if this work is maximally successful, though, there is no chance that the kernel will be free of these bugs anytime soon. Thus, there is also ongoing interest in hardening the kernel to make the existing bugs more difficult to exploit. The upcoming 7.2 kernel release will include a change to how dynamically allocated structures are placed in memory to make them harder to overwrite, while a project to randomize structure layout at boot time has a rather longer timeline.
