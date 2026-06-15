---
title: '[$] Automatic mTHP creation in 7.2'
url: https://lwn.net/Articles/1077208/
published: "2026-06-11T14:33:27Z"
feed: lwn
guid: https://lwn.net/Articles/1077208/
---

# [$] Automatic mTHP creation in 7.2

The Linux kernel has long tried to use huge pages as a way to improve
performance, sometimes with more success than others. The size of huge
pages has traditionally been imposed by the hardware, which typically only
offers a couple of relatively large options. In more recent times, though,
the use of multi-size transparent huge pages (mTHPs), with more flexible
sizing implemented in software, has been growing. If all goes well, the
7.2 development cycle will include the addition of [a new feature](https://lwn.net/ml/all/20260605161422.213817-1-npache@redhat.com),
contributed by Nico Pache, to make the use of mTHPs even more transparent.
