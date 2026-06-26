---
title: New @bitCast Semantics and LLVM Backend Improvements
url: https://ziglang.org/devlog/2026/#2026-06-25
published: "2026-06-25T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-06-25
---

# New @bitCast Semantics and LLVM Backend Improvements

Author: Matthew Lugg

(Quite long devlog coming up, apologies—I got a little carried away with this one!)

A few weeks ago, I began working on a branch implementing an improvement to the LLVM backend which had been planned for a long time. This ended up snowballing into a bigger change which implemented a few language proposals you might be interested to hear about.

## LLVM Backend Integer Lowering

Zig has always lowered arbitrary bit-width integer types (e.g. `u4`, `i13`, `u40`) directly to LLVM IR’s bit-int types ( `i4`, `i13`, `i40`). However, we’ve known for a long time that this lowering is not optimal, because LLVM’s documented semantics for representing these types in memory are unnecessarily restrictive to the optimizer. Perhaps more importantly, because Clang never emits LLVM IR like this, these code paths in LLVM have never been properly tested, and so are poorly supported in practice—over the past few years, we have observed many instances of [trivial optimizations being missed](https://github.com/ziglang/zig/issues/17768) and even straight-up [miscompilations](https://codeberg.org/ziglang/zig/issues/35560).

So, the original goal of the PR was to only use these bit-int types when manipulating values in SSA form, and to zero- or sign-extend them to ABI-sized types ( `i8`, `i16`, `i32`, etc) when storing them in memory. This should be well-supported, not least because it matches how Clang lowers C’s `_BitInt(N)`!

That change was actually fairly straightforward, but I hit one issue which led me down a bit of a rabbit-hole.

## The Problem with `@bitCast`

`@bitCast` is an interesting builtin. In the past, it was defined as being equivalent to the following sequence of operations:

- Take a pointer to the operand value
- Cast it to a pointer to the destination type
- Load from that pointer

In other words, it was essentially syntax sugar for reinterpreting bytes of memory. However, over time, we diverged from this definition—for instance, it became allowed to use `@bitCast` to reinterpret a `[3]u8` as a `u24`, even though on most targets `@sizeOf(u24)` is greater than `@sizeOf([3]u8)` so the above definition would invoke Illegal Behavior.

Up to now, the LLVM backend had implemented these underspecified semantics for the `@bitCast` builtin. However, because that definition involved reinterpreting memory, changing how we store integer types in memory ended up impacting the implementation of `@bitCast`, and introducing Illegal Behavior which led to crashes in the compiler test suite.

The easiest solution to this would probably have been to implement logic in the LLVM backend to approximately match the old behavior. I instead opted for a better solution—implement a new definition of `@bitCast`.

## Redefining `@bitCast`

In 2024, Jacob Young wrote up language proposal [#19755](https://github.com/ziglang/zig/issues/19755) which aimed to solve the problems with `@bitCast` by precisely specifying a new set of semantics for it. This proposal was accepted shortly after it was submitted, and in fact, the semantics it details are already implemented by the self-hosted x86\_64 backend! So to solve the LLVM backend’s problems, I didn’t necessarily need to match the old `@bitCast` semantics—instead, this seemed like a good time to finally get the new semantics implemented *everywhere*.

As an aside, another advantage to doing this is that we could take advantage of the compiler’s `Legalize` pass, which takes difficult-to-lower operations and rewrites them in terms of simpler operations, so that compiler backends only need to support those simple operations. `Legalize` already had functionality, used by the self-hosted x86\_64 backend, which converted complex `@bitCast` operations into simpler ones, and it could be easily adapted to aid the other compiler backends too (mainly the LLVM and C backends)—but only if they implemented the new semantics.

Regardless, the point is, I set out on a side quest (which ended up being harder than the *original* quest) to implement these new semantics throughout the compiler. This includes not only the LLVM and C backends, but also `comptime` execution—after all, Zig allows you to do almost any operation at comptime, `@bitCast` included! Because the new semantics are meaningfully different from the old (more on this later), I also had to audit a lot of uses of `@bitCast` across the standard library, compiler, and supporting libraries (e.g. `compiler_rt`). But after a few mostly-painless fixes for CI failures, I was able to finally get [my PR](https://codeberg.org/ziglang/zig/pulls/35711) green, and landed it in master yesterday (closing a good few issues in the process!).

## The New `@bitCast` Semantics

Now that we’ve gotten through all of the background, it’s finally time for me to actually explain new `@bitCast` behavior. Instead of being based on reinterpreting bytes in memory like before, the builtin is now defined in terms of the bits which *logically* represent a type.

Every type which supports `@bitCast` has a “logical bit layout”—a representation of that type as an ordered sequence of bits. For instance, `u5` is composed of 5 logical bits, which we order from least-significant to most-significant. `[2]u5` is composed of 10 logical bits—the 5 from the first element, followed by the 5 from the second element. The new definition of `@bitCast` is that it reinterprets the logical bits of one type as the logical bits of a different type.

The simplest example is to take an unsigned integer, say a `u8`, and convert it to a signed integer of the same size, in this case `i8`. This operation does exactly what you’d expect—the bits are unchanged, and we just reinterpret the most-significant bit as a sign bit. Also unchanged are the semantics of `@bitCast` between an integer type and a `packed struct`/ `packed union` type.

The place where the new semantics differ from the old is when you get aggregate types (arrays and vectors) involved.

Consider, for instance, bitcasting a `[2]u8` to a `u16`. Under the old semantics, the result of this operation depends on the target endian: on big-endian targets, the first array element became the 8 *most* significant bits, whereas on little-endian targets, the first array element became the 8 *least* significant bits. Under the new semantics, because we only care about *logical* bit representation (which is endian-agnostic), the operation behaves identically on every target: the first array element becomes the 8 *least* significant bits. As a general rule, the new semantics tend to match the behavior of the old semantics on little-endian targets.

This definition also allows for some weirder operations, such as converting `[2]u3` to `@Vector(3, u2)`:

```zig
test "bitcast [2]u3 to @Vector(3, u2)" {
    const arr: [2]u3 = .{ 0b001, 0b011 };
    const vec: @Vector(3, u2) = @bitCast(arr);

    // Concatenate all bits of `arr` starting with the least-significant bit of `arr[0]` to find the
    // logical bit sequence, then read off 2-bit chunks from it to get the elements of the resulting
    // vector value `vec`.
    //
    //     arr[0]         arr[1]
    //     0b001          0b011
    // -------------  -------------
    //  1    0    0    1    1    0
    // --------  --------  --------
    //   0b01      0b10      0b01
    //  vec[0]    vec[1]    vec[2]

    try expect(vec[0] == 0b01);
    try expect(vec[1] == 0b10);
    try expect(vec[2] == 0b01);
}
const expect = @import("std").testing.expect;

```

This kind of operation isn’t very useful most of the time, but it’s there if you need it! For instance, perhaps you want to deconstruct an integer into a vector of individual bits to operate on—that can now be done by a `@bitCast` to `@Vector(n, u1)`.

While doing all of this stuff, I also implemented a couple of smaller accepted proposals—I won’t detail them here, but you can take a look at the issues if you’re interested:

- Disallow `@bitCast` to/from vectors of pointers ( [#18936](https://github.com/ziglang/zig/issues/18936))
- Allow `@bitCast` on enums (part of [#35602](https://codeberg.org/ziglang/zig/issues/35602))

Of course, all of these changed semantics will be explained in the 0.17.0 release notes (hopefully a bit more concisely than what I managed here!), and suggested migration steps outlined.

## LLVM Backend Performance

On a final note, I just wanted to mention that the *original* motivation for this branch—changing how the LLVM backend lowers non-ABI integer types—was [demonstrably successful](https://github.com/ziglang/zig/issues/17768#issuecomment-4787726124) at restoring missed optimizations. In fact, the Zig compiler itself—despite not making heavy use of arbitrary bit width integers internally!—saw around 5% performance improvements from the better optimization. This means you might have some minor runtime performance gains to look forward to in 0.17.0!

Thanks for reading, I hope this was interesting to some of you. Happy hacking!
