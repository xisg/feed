---
title: '[$] Single-hop block replication with RMR and BRMR'
url: https://lwn.net/Articles/1074291/
published: "2026-06-18T13:25:34Z"
feed: lwn
guid: https://lwn.net/Articles/1074291/
---

# [$] Single-hop block replication with RMR and BRMR

How can cloud providers efficiently supply durable virtual block devices? Remote Direct Memory Access (RDMA) provides a way for servers in a cluster to share chunks of memory, but there still needs to be a protocol that operates on top of RDMA to provide the guarantees expected of a block device. The kernel's RDMA transport library (RTRS) provides a way to send messages via RDMA. I [presented](https://lwn.net/images/2026/RMR_BRMR.pdf) about two new components built on top of RTRS at the 2026 [Linux
Storage, Filesystem, Memory Management and BPF Summit](https://events.linuxfoundation.org/lsfmmbpf/): Reliable Multicast over RTRS (RMR) and Block device over RMR (BRMR). These modules, which I am working on with Jia Li, could be a way for cloud providers to expose durable block devices with as little overhead as possible. To accomplish that, however, we need some discussion and feedback from the community before sending the modules upstream.
