---
title: Multiplying the Shuffle Speed in Go with Batched Shuffling
url: https://lemire.me/blog/2025/12/15/multiplying-the-shuffle-speed-in-go-with-batched-shuffling/
published: "2025-12-15T01:42:10Z"
feed: lemire
guid: https://lemire.me/blog/?p=22369
---

# Multiplying the Shuffle Speed in Go with Batched Shuffling

![](https://lemire.me/blog/wp-content/uploads/2025/12/pcgshuffle_benchmark-150x150.png)

Programmers often want to randomly shuffle arrays. Evidently, we want to do so as efficiently as possible. Maybe surprisingly, I found that the performance of random shuffling was not limited by memory bandwidth or latency, but rather by computation. Specifically, it is the computation of the random indexes itself that is slow.

[Earlier in 2025](https://lemire.me/blog/2025/04/06/faster-shuffling-in-go-with-batching/), I reported how you could more than double the speed of a random shuffle in Go using a new algorithm ( [Brackett-Rozinsky and Lemire, 2025](https://arxiv.org/pdf/2408.06213)). However, I was using custom code that could not serve as a drop-in replacement for the standard Go Shuffle function. I decided to write a [proper library called `batchedrand`](https://github.com/lemire/batchedrand). You can use it just like the `math/rand/v2` standard library.

```
rng := batchedrand.Rand{rand.New(rand.NewPCG(1, 2))}

data := []int{1, 2, 3, 4, 5}
rng.Shuffle(len(data), func(i, j int) {
    data[i], data[j] = data[j], data[i]
})

```

How fast is it? The standard library provides two generators, PCG and ChaCha8. ChaCha8 should be slower than PCG, because it has better cryptographic guarantees. However, both have somewhat comparable speeds because ChaCha8 is heavily optimized with assembly code in the Go runtime while the PCG implementation is conservative and not focused on speed.

On my Apple M4 processor with Go 1.25, I get the following results. I report the time per input element, not the total time.

BenchmarkSizeBatched (ns/item)Standard (ns/item)speedupChaChaShuffle301.84.62.6ChaChaShuffle1001.84.72.5ChaChaShuffle5000002.65.11.9PCGShuffle301.53.92.6PCGShuffle1001.54.22.8PCGShuffle5000001.93.82.0

Thus, from tiny to large arrays, the batched approach is two to three times faster. Not bad for a drop-in replacement!

Get the Go library at [https://github.com/lemire/batchedrand](https://github.com/lemire/batchedrand)

[![](http://lemire.me/blog/wp-content/uploads/2025/12/pcgshuffle_benchmark.png)](http://lemire.me/blog/wp-content/uploads/2025/12/pcgshuffle_benchmark.png)

**Further reading:**

- Nevin Brackett-Rozinsky, Daniel Lemire, [Batched Ranged Random Integer Generation](https://arxiv.org/pdf/2408.06213), Software: Practice and Experience 55 (1), 2025
- Daniel Lemire, Fast Random Integer Generation in an Interval, ACM Transactions on Modeling and Computer Simulation, Volume 29 Issue 1, February 2019
