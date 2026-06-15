---
title: Global Executors
url: https://without.boats/blog/global-executors/
published: "2019-11-14T00:00:00Z"
feed: boats
guid: https://without.boats/blog/global-executors/
---

# Global Executors

One of the big sources of difficulty on the async ecosystem is spawning tasks. Because there is no API in std for spawning tasks, library authors who want their library to spawn tasks have to depend on one of the multiple executors in the ecosystem to spawn a task, coupling the library to that executor in undesirable ways.
Ideally, many of these library authors would not need to spawn tasks at all.
