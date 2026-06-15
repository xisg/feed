---
title: Checking multiplication overflow
url: https://lemire.me/blog/2026/05/06/checking-multiplication-overflow/
published: "2026-05-06T20:15:20Z"
feed: lemire
guid: https://lemire.me/blog/?p=22629
---

# Checking multiplication overflow

![](https://lemire.me/blog/wp-content/uploads/2026/05/Capture-decran-le-2026-05-06-a-16.14.54-150x150.png)

Suppose that `x` is a variable of an unsigned type. In C/C++, it could be of type `size_t` for example.

You have an expression like `6 * x` and you want to know whether `6 * x` overflows. That is, you want to know if `6 * x` exceeds the range of values that can be represented by the type. In most cases, a variable of type `size_t` will be about to represent all values in the range `[0, 2^64-1]`. Instead of 64, let me use a variable for the number of bits: `[0, 2^L-1]`.

The easiest approach is to compare `x` with `(2^L-1) // 6` where I use the symbol `//` to denote the integer division (as opposed to `/`).

But can you do otherwise ?

If the value does not overflow, we know for sure that `(6 * x)//6 == x`. The interesting question is what happens when it overflows. We can answer this directly for an arbitrary non-zero constant `a` in the range `[1, 2^L-1]`.

Let `k = (a*x)//2^L` be the number of times the multiplication wraps around. The effective (wrapped) value computed by the machine is `r = a*x - k*2^L`, with `0 <= r < 2^L`. Overflow happens precisely when `k >= 1`. We have that `k <= a − 1` because `x<2^L`.

Performing the integer division of `r = a*x - k*2^L` by `a`, we get `x` plus `-k*2^L//a`. When `k` is non-zero, this last value ( `-k*2^L//a`) is one of `-2^L//a`, `-2* 2^L//a`, …, `-(a-1) * 2^L//a`.

- When `k = 0` (no overflow): `r // a = x`.
- When `k ≥ 1`: `r // a = x + (negative integer) ≠ x`.

Hence we have the following result.

_Theorem_ If `x` is of an unsigned type and `a` is a non-zero constant, then `a * x` overflows if and only if `(a * x)//a != x`.

In practice, a simple comparison x with `(2^L-1) // a`  is likely more efficient. Optimizing compilers might be able to convert `(a * x)//a != x` to a simple comparison. Unfortunately, the Go compiler (for example) cannot.

An open question is whether there is a more mathematically elegant check.
