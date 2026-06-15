---
title: retrobootstrapping rust for some reason
url: https://graydon2.dreamwidth.org/317484.html
published: "2025-06-16T19:13:06Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:317484
---

# retrobootstrapping rust for some reason

Elsewhere I've been asked about the task of replaying the bootstrap process for rust. I figured it would be fairly straightforward, if slow. But as we got into it, there were _just_ enough tricky / non-obvious bits in the process that it's worth making some notes here for posterity.

UPDATE: someone has [also scripted many of the subsequent snapshot builds](https://github.com/LegionMammal978/rust-from-ocaml) covering many years of rust's post-bootstrap development. Consider the rest of this post just a verbose primer for interpreting their work.

### context

Rust started its life as a compiler written in ocaml, called **rustboot**. This compiler did _not_ use LLVM, it just emitted 32-bit i386 machine code in 3 object file formats (Linux PE, macOS Mach-O, and Windows PE).

We then wrote a _second_ compiler _in Rust_ called **rustc** that _did_ use LLVM as its backend (and which, yes, is the genesis of today's rustc) and ran rustboot on rustc to produce a so-called "stage0 rustc". Then stage0 rustc was fed the sources of rustc again, producing a stage1 rustc. Successfully executing this stage0 -> stage1 step (rather than just crashing mid-compilation) is what we're going to call "bootstrapping". There's also a third step: running stage1 rustc on rustc's sources again to get a stage2 rustc and checking that it is bit-identical to the stage1 rustc. Successfully doing _that_ we're going to call "fixpoint".

Shortly after we reached the fixpoint we [discarded rustboot](https://github.com/rust-lang/rust/commit/6997adf76342b7a6fe03c4bc370ce5fc5082a869). We stored stage1 rustc binaries as snapshots on a shared download server and all subsequent rust builds were based on downloading and running that. Any time there was an incompatible language change made, we'd add support and re-snapshot the resulting stage1, gradually growing a long list of snapshots marking the progress of rust over time.

### time travel and bit rot

Each snapshot can typically only compile rust code in the rust repository written between its birth and the next snapshot. This makes replaying the entire history awkward ( [see above](https://github.com/LegionMammal978/rust-from-ocaml)). We're not going to do that here. This post is just about replaying the initial bootstrap and fixpoint, which happened back in April 2011, 14 years ago.

Unfortunately all the tools involved -- from the host OS and system libraries involved to compilers and compiler-components -- were and are moving targets. Everything bitrots. Some examples discovered along the way:

- Modern clang and gcc won't compile the LLVM used back then (C++ has changed too much -- and I tried several CXXFLAGS=-std=c++NN variants!)

- Modern gcc won't even compile the gcc used back then (apparently C as well!)

- Modern ocaml won't compile rustboot (ditto)

- 14-year-old git won't even connect to modern github (ssh and ssl have changed too much)

### debian

We're in a certain amount of luck though:

- Debian has maintained both EOL'ed docker images and still-functioning fetchable package archives at the same URLs as 14 years ago. So we can time-travel using that. A VM image would also do, and if you have old install media you could presumably build one up again if you are patient.

- It is easier to use i386 since that's all rustboot emitted. There's some indication in the Makefile of support for multilib-based builds from x86-64 (I honestly don't remember if my desktop was 64 bit at the time) but 32bit is much more straightforward.

- So: `docker pull --platform linux/386 debian/eol:squeeze` gets you an environment that works.

- You'll need to install rust's prerequisites also: g++, make, ocaml, ocaml-native-compilers, python.

### rust

The next problem is figuring out the code to build. Not totally trivial but not _too_ hard. The best resource for tracking this period of time in rust's history is actually the rust-dev mailing list archive. There's [a copy online at mail-archive.com](https://www.mail-archive.com/rust-dev@mozilla.org/info.html) (and Brian keeps [a public backup of the mbox file](https://github.com/brson/rust-dev-archives) in case that goes away). Here's [the announcement that we hit a fixpoint](https://www.mail-archive.com/rust-dev@mozilla.org/msg00329.html) in April 2011. You kinda have to just know that's what to look for. So that's the rust commit to use: 6daf440037cb10baab332fde2b471712a3a42c76. This commit still exists in the rust-lang/rust repo, no problem getting it (besides having to copy it into the container since the container can't contact github, haha).

### LLVM

Unfortunately we only started pinning LLVM to specific versions, using submodules, _after_ bootstrap, closer to the initial "0.1 release". So we have to guess at the LLVM version to use. To add some difficulty: LLVM at the time was developed on subversion, and we were developing rust against [a fork of a git mirror of their SVN](https://github.com/brson/llvm). Fishing around in that repo at least finds a version that builds -- [45e1a53efd40a594fa8bb59aee75bb0984770d29](https://github.com/brson/llvm/commit/45e1a53efd40a594fa8bb59aee75bb0984770d29), which is "the commit that exposed `LLVMAddEarlyCSEPass`", a symbol used in the rustc LLVM interface. I bootstrapped with that (brson/llvm) commit but subversion also numbers all commits, and they were preserved in the conversion to the modern LLVM repo, so you can see the same svn id 129087 as [e4e4e3758097d7967fa6edf4ff878ba430f84f6e](https://github.com/llvm/llvm-project/commit/e4e4e3758097d7967fa6edf4ff878ba430f84f6e) over in the official LLVM git repo, in case brson/llvm goes away in the future.

Configuring LLVM for this build is also a little bit subtle. The best bet is to actually read the rust 0.1 configure script -- when it was managing the LLVM build itself -- and work out what it would have done. But I have done that and can now save you the effort: `./configure --enable-targets=x86 --build=i686-unknown-linux-gnu --host=i686-unknown-linux-gnu --target=i686-unknown-linux-gnu --disable-docs --disable-jit --enable-bindings=none --disable-threads --disable-pthreads --enable-optimized`

So: configure and build that, stick the resulting bin dir in your path, and configure and make rust, and you're good to go!

```
root@65b73ba6edcc:/src/rust# sha1sum stage*/rustc
639f3ab8351d839ede644b090dae90ec2245dfff  stage0/rustc
81e8f14fcf155e1946f4b7bf88cefc20dba32bb9  stage1/rustc
81e8f14fcf155e1946f4b7bf88cefc20dba32bb9  stage2/rustc

```

### Observations

On my machine I get: 1m50s to build stage0, 3m40s to build stage1, 2m2s to build stage2. Also stage0/rustc is a 4.4mb binary whereas stage1/rustc and stage2/rustc are (identical) 13mb binaries.

While this is somewhat congruent with my recollections -- rustboot produced code faster, but its code ran slower -- the effect size is actually much less than I remember. I'd convinced myself retroactively that rustboot was produced _abysmally_ worse code than rustc-with-LLVM. But out-of-the-gate LLVM only boosted performance by 2x (and cost of 3x the code size)! Of course I also have a faster machine now. At the time bootstrap cycles took about a half hour each ( [according to this: 15 minutes for the 2nd stage](https://www.mail-archive.com/rust-dev@mozilla.org/msg00331.html)).

Of course you can still see this as a condemnation of the entire "super slow dynamic polymorphism" model of rust-at-the-time, either way. It may seem funny that this version of rustc bootstraps faster than today's rustc, but this "can barely bootstrap" version was a mere 25kloc. Today's rustc is 600kloc. It's really comparing apples to oranges.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=317484) comments
