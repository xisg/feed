---
title: How stagnant is CPU technology?
url: https://lemire.me/blog/2026/01/14/how-stagnant-is-cpu-technology/
published: "2026-01-14T14:52:39Z"
feed: lemire
guid: https://lemire.me/blog/?p=22451
---

# How stagnant is CPU technology?

![](https://lemire.me/blog/wp-content/uploads/2026/01/G-iCzw6XgAAkLsj-150x150.jpeg)

Sometimes, people tell me that there is no more progress in CPU
performance.

Consider these three processors which had comparable prices at
release time.

1. The AMD Ryzen 7 9800X3D (Zen 5, with up to 5.3 GHz boost) was
   released in 2024.
2. The AMD Ryzen 7 7800X3D (Zen 4, with up to 5.1 GHz boost) was
   released in 2023.
3. The AMD Ryzen 7 5800X3D (Zen 3, with 3.4 GHz base) was released
   in 2022.

Let us consider their results on on the
[PartialTweets open benchmark](https://openbenchmarking.org/test/pts/simdjson&eval=55ded682564c03f676d77dbc1f8de67e27c7aadc#metrics)
(JSON parsing). It is a single core benchmark.

2024 processor12.7 GB/s2023 processor9 GB/s2022 processor5.2 GB/s

In two years, on this benchmark, AMD more than doubled the
performance for the same cost.

So what is happening is that processor performance is indeed going
up, sometimes dramatically so, but not all of our software can
benefit from the improvements. Software developers must track the
trends and adapt our software accordingly. Unfortunately, it is
hard work and it requires expertise. In the case of this
benchmark, the simdjson library is designed to benefit from better
processor features.

Not all software can easily run much faster on new processors, and
genuine progress is difficult.

Let us be ambitious. Let us move forward!
