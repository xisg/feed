---
title: Bringing Stack Clash Protection to Clang / X86 — the Open Source Way
url: https://blog.llvm.org/posts/2021-01-05-stack-clash-protection/
published: "2021-01-30T10:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2021-01-05-stack-clash-protection/
---

# Bringing Stack Clash Protection to Clang / X86 — the Open Source Way

Stack clash is an attack that dates back to 2017, when the Qualys Research Teamreleased an advisory with a [joint blog post](https://blog.qualys.com/vulnerabilities-research/2017/06/19/the-stack-clash). It basicallyexploits large stack allocation (greater than `PAGE_SIZE`) that can lead tostack read/write _not_ triggering the [stack guard page](https://lkml.org/lkml/2017/6/22/345) allocated by the LinuxKernel.

Shortly after the advisory got released, GCC provided a [countermeasure](https://gcc.gnu.org/legacy-ml/gcc-patches/2017-07/msg00556.html) activated by `-fstack-clash-protection` thatbasically consists in splitting large allocation in chunks of `PAGE_SIZE`,with a probe in each chunk to trigger the kernel stack guard page.

This has been a major security difference between GCC and Clang since then. Ithas even been identified as a blocker by Fedora to move from GCC to Clang as thecompiler for some projects that already made the move upstream, leading to [extramaintenance](https://pagure.io/fesco/issue/2020) for packagers.

Support for this flag [landed in Clang in 2020](https://reviews.llvm.org/D68720),only for X86, SystemZ and PowerPC. Its implementation is a result of a fruitfulcollaboration between LLVM, Firefox and Rust developers.

Rust already had a countermeasure implemented in the form of a runtime call toperform the stack probing. With LLVM catching up, using a more lightweightapproach got [investigated in Rust](https://github.com/rust-lang/rust/pull/77885).

# Countermeasure Description

The Clang implementation for X86 is derived from the GCC implementation, with afew distinctions. The core ideas are:

1. thanks to X86 calling convention, we get a free probe at each call site,which means that each function starts with a probed stack

2. when probing the stack in the function prologue, we don’t probe the tail ofthe allocation. Stated otherwise, if the stack size is `PAGE_SIZE + PAGE_SIZE/2`,we want to probe only once. This is important to limit the numberof probes: if the stack size is lower than `PAGE_SIZE` no probe is needed

3. because a signal can interrupt the execution flow any time, at no pointshould we have two stack allocations (lower than `PAGE_SIZE`) without a probein between.


The probing strategy for stack allocation varies based on the size of the stackallocation. If it’s smaller than `PAGE_SIZE`, thanks to (2) no probing isneeding. If it’s below a small multiple of `PAGE_SIZE`, then the probing loopcan be unrolled. Otherwise a probing loop alternates stack allocation of `PAGE_SIZE` bytes and probe, starting with the allocation thanks to (1).

As side effect of (2) is that when performing a dynamic allocation, we need toprobe _before_ updating the stack, otherwise we got a hole in the protection.This probe cannot be done after the stack update, even with an offset, becauseof (3). Otherwise we end up with a bug as this one found in [GCC](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=97928)

The following scheme attempts to summarize the allocation and probinginteraction between static and dynamic allocations:

```
 + ----- <- ------------ <- ------------- <- ------------ + | |[free probe] -> [page alloc] -> [alloc probe] -> [tail alloc] + -> [dyn probe] -> [page alloc] -> [dyn probe] -> [tail alloc] + | | + <- ----------- <- ------------ <- ----------- <- ------------ +
```

# Validation with Firefox

Firefox provides an amazing test bench to evaluate the impact of compilerchanges. Indeed, with more than 12MLOC of C/C++ and 3MLOC of Rust built usingPGO/LTO and [XLTO](https://blog.llvm.org/2019/09/closing-gap-cross-language-lto-between.html), most of the important cases are covered.

Moreover, Firefox being supported on a large set of operating system andarchitectures, it was a great way to test the Stack Clash protection on variousset of configurations.

The work is detailed in the [bug 1588710](https://bugzilla.mozilla.org/show_bug.cgi?id=1588710).

## Functional Testing

To make sure that Firefox would perform as expected, we leveraged the huge testsuite to verify that the product would still work as expected with this option.

We used the [`try auto`](https://hacks.mozilla.org/2020/07/testing-firefox-more-efficiently-with-machine-learning/), a new command which will run the mostappropriate set of tests for such kind of changes during the development phase.Then, once the patch landed into Mozilla-central (Firefox nightly), the wholetest suite is executed, presenting about 29 days of machine time for about 9000tasks.

Thanks to this infrastructure, we have identified an [issue with `alloca(0)`](https://bugzilla.mozilla.org/show_bug.cgi?id=1588710#c40) generating buggy machine code.Fortunately, the [fix](https://bugs.llvm.org/show_bug.cgi?id=47657) was already in the trunk version of LLVM.We cherry-picked the fix in our custom Clang build which addressed our issue.

## Performance Testing

Over the years, Mozilla has developed a few tools to evaluate performanceimpact of changes, from micro-benchmark to page loads. These tools have been keyto improve Firefox overall performances but also evaluate the impact of the [moveto Clang](https://blog.mozilla.org/nfroyd/2018/05/29/when-implementation-monoculture-right-thing/) on all platforms done a couple years ago.

The usual procedure to evaluate performances improvements/regressions is to:

1. Run two builds with benchmarks. One without the patch, one with it.

2. Leverage the tooling to rerun the benchmark (usually 5 to 20 times) to limitthe noise.

3. Compare the various benchmark to see if significant regressions can beidentified.


In the context of this project, we run the usual benchmarks sensitive to C++changes and we haven’t identified any [regression](https://treeherder.mozilla.org/perfherder/comparesubtest?originalProject=try&newProject=try&newRevision=62108fa48bd15fe01f1a0f1ffab133af9b4207cc&originalSignature=1904137&newSignature=1904137&framework=10&originalRevision=a47c98b909b61035dae2e1e00883f2ade0fef129) in term ofperformances.

## Current status

Firefox nightly on Linux is now compiled with the stack-clash-option fromJanuary 8th 2021. We have not detected any regressions since it landed.If everything goes well, this change should ship with Firefox 86 (planned formid February 2021).

# Validation With Rust

Rust has long supported the callback style of the LLVM `probe-stack` attribute,using the function `__rust_probestack` defined in its own compiler builtinslibrary. In Rust’s spirit of safety, this attribute is added to _all_ functions,letting LLVM sort out which actually need probing. However, forcing such a callinto every function with a large stack frame is not ideal for performance,especially for those cases that could use just a few unrolled probes inline.Furthermore, Rust only has this callback implemented for its Tier 1 (mostsupported) targets, namely i686 and x86\_64, leaving other architectures withoutprotection so far. Therefore, letting LLVM generate inline stack probes isbeneficial both for the performance of avoiding a call and for the increasedarchitecture support.

Since the Rust compiler is written in Rust itself, with stack probing enabled bydefault, it makes a great functional test for any new code generation feature.The compiler is bootstrapped in stages, first building with a prior version,then rebuilding with the result of that first stage. Codegen issues are oftenrevealed if the compiler crashes during that rebuild, and experiments withinline stack probes were no different, leading to fixes in [D82867](https://reviews.llvm.org/D82867) and [D90216](https://reviews.llvm.org/D90216). Both of these were simple errors thatwere not apparent in existing FileCheck tests, showing the importance ofactually executing generated code.

An [issue](https://github.com/rust-lang/rust/issues/70143) also led to the realization that there was a moregeneral bug impacting both GCC and LLVM implementation of `-fstack-clash-protector`, leading to a new patch set on the LLVM side.Essentially, the observed behavior is the following:

Alignment requirements behave similarly to allocation with respect to the stack:they (may) make it grow. For instance the stack allocation for an `char foo[4096] __attribute__((aligned(2048)));` is done through:

```
and rsp, -2048sub rsp, 6024
```

Both `and` and the `sub` actually update the stack! To take that effect intoaccount, the LLVM patch considers the `and rsp, -2048` as a `sub rsp, 2048` when computing the probing distance, which means considering the worstcase scenario.

For future work on the Rust side, inline stack probes will replace `__rust_probestack` on i686 and x86\_64 soon in [Rustpr77885](https://github.com/rust-lang/rust/pull/77885), and that will include [perf results](https://perf.rust-lang.org/) to monitor the effect. After that,additional architectures can be functionally tested and enabled for inline stackprobes as well, increasing the reach of Rust’s memory safety.

# Validation with a Binary Tracer

None of the above validation validates the security aspect of the protection. Tohave more confidence on the actual probing scheme implementation, we implementeda binary tracer based on the (awesome) [QBDI](https://github.com/QBDI/QBDI) Dynamic Binary Instrumentation framework. This Proof Of Concept (POC) isavailable on GitHub: [stack-clash-tracer](https://github.com/serge-sans-paille/stack-clash-tracer)

This tool instruments all stack allocation and memory access of a runningbinary, logs them and checks that no stack allocation is greater than `PAGE_SIZE` and that we get an actual probing between two allocations.

Here is a sample session that showcases large stack allocation issues:

```
$ cat main.c#include <alloca.h>#include <string.h>int main(int argc, char**argv) { char buffer[5000]; strcpy(buffer, argv[0]); char* dynbuffer = alloca(argc * 1000); strcpy(dynbuffer, argv[0]); return buffer[argc] + dynbuffer[argc];}$ gcc main.c -o main$ LD_PRELOAD=./libstack_clash_tracer.so ./main 1[sct][error] stack allocation is too big (5024)$ LD_PRELOAD=./libstack_clash_tracer.so ./main 1 2 3 4 5[sct][error] stack allocation is too big (5024)[sct][error] stack allocation is too big (6016)
```

The same code, compiled with `-fstack-clash-protection`, is safer (apart fromthe stupid use of `strcpy`, that is)

```
$ gcc main.c -fstack-clash-protection -o main$ LD_PRELOAD=./libstack_clash_tracer.so ./main 1$ LD_PRELOAD=./libstack_clash_tracer.so ./main 1 2 3 4 5
```

Small bonus of this compiler-independent approach: we can verify both GCC andClang implementation `:-)`

```
$ clang main.c -fstack-clash-protection -o main$ LD_PRELOAD=./libstack_clash_tracer.so ./main 1$ LD_PRELOAD=./libstack_clash_tracer.so ./main 1 2 3 4 5
```

To come back on the Firefox test case, before we landed the change, we couldsee:

```
$ LD_PRELOAD=./libstack_clash_tracer.so firefox-bin[sct][error] stack allocation is too big (4168)
```

Once Firefox nightly shipped with stack clash protection, this warningdisappears.

# Conclusion

Aside from the technical aspects of the countermeasure, it is interesting tonote that its Clang implementation was derived from the GCC implementation, butled to an issue being reported in the GCC codebase. The Clang-generated code gotvalidated by Firefox People, tested by Rust people who reported several bugs,some impacting both Clang and GCC implementation, the circle is complete!

# References
