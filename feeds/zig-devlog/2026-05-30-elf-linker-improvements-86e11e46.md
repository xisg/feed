---
title: ELF Linker Improvements
url: https://ziglang.org/devlog/2026/#2026-05-30
published: "2026-05-30T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-05-30
---

# [ELF Linker Improvements](\#2026-05-30)

Author: Matthew Lugg

I’ve spent the past few weeks working on our new ELF linker which debuted in Zig 0.16.0. At the time of the 0.16.0 release, this linker implementation was in its fairly early stages, and only really supported linking Zig-only code without any external libraries (even libc)—hence why it was (and still is) disabled by default (it can be enabled with `-fnew-linker`). However, quite a lot of progress has been made since that initial release!

Here’s a nice milestone—as of [my latest PR](https://codeberg.org/ziglang/zig/pulls/35533), the new ELF linker is capable of building the self-hosted Zig compiler with LLVM and LLD libraries enabled, a task which requires quite a few features under the hood.

```sh
[mlugg@nebula master]$ # Build the Zig compiler using the new linker:
[mlugg@nebula master]$ zig build -Dno-lib -Dnew-linker -Denable-llvm
[mlugg@nebula master]$ # Use that compiler to build something with LLVM and LLD:
[mlugg@nebula master]$ ./zig-out/bin/zig build-exe ~/hello.zig -fllvm -flld
[mlugg@nebula master]$ ./hello
Hello, World!
[mlugg@nebula master]$

```

Of course, an ELF linker isn’t necessarily the most exciting thing in the world, which is why the headline feature of this new linker is its support for fast incremental compilation. After the recent enhancements, it is now possible (on x86\_64 Linux) to perform incremental rebuilds while linking external libraries, C sources, etc—without any additional performance overhead! Here’s a clip of me trying it out on [Andrew’s Tetris clone](https://github.com/andrewrk/tetris):

A few silly changes to Andrew’s Tetris clone being built in around 30ms each.

Oh, and fast incremental rebuilds also work nicely on the Zig compiler itself:

```
[mlugg@nebula master]$ zig build -Dno-lib -Denable-llvm -fincremental --watch
Build Summary: 4/4 steps succeeded
install success
└─ install zig success
   └─ compile exe zig Debug native success 36s

Build Summary: 4/4 steps succeeded
install success
└─ install zig success
   └─ compile exe zig Debug native success 244ms

Build Summary: 4/4 steps succeeded
install success
└─ install zig success
   └─ compile exe zig Debug native success 228ms

Build Summary: 4/4 steps succeeded
install success
└─ install zig success
   └─ compile exe zig Debug native success 288ms

Build Summary: 4/4 steps succeeded
install success
└─ install zig success
   └─ compile exe zig Debug native success 283ms

```

The biggest missing feature of this linker implementation right now is that it still does not yet support generating DWARF debug information for Zig code—that’s definitely my next priority. But even without that support, it’s amazing just how useful instant rebuilds can be, for example in any situation where you’re doing a lot of print debugging.

If you’re using the master branch of Zig and you’re on x86\_64 Linux, consider trying out incremental compilation with the new ELF linker if it previously wasn’t working with your project! I expect many codebases to already work great with it, unlocking the ability to rebuild your project in milliseconds. Of course, if you come across any bugs, please do [open an issue](https://codeberg.org/ziglang/zig/issues).

And if you’re currently sticking to tagged releases of Zig, don’t worry—as Andrew mentioned in his last devlog, Zig 0.17.0 is just around the corner, so it won’t be long before you can try this too!
