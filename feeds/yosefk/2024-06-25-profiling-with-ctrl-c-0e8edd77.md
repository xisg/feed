---
title: Profiling with Ctrl-C
url: https://yosefk.com/blog/profiling-with-ctrl-c.html
published: "2024-06-25T00:00:00Z"
feed: yosefk
---

# Profiling with Ctrl-C

I once wrote about [how profiler\
output can be misleading](https://yosefk.com/blog/how-profilers-lie-the-cases-of-gprof-and-kcachegrind.html). Someone [commented](https://yosefk.com/cgi-bin/comments.cgi?post=blog/how-profilers-lie-the-cases-of-gprof-and-kcachegrind#comment-1847)
that you don’t need profilers - just Ctrl-C your program in a debugger instead, and you’ll see the call stack where your program
probably spends most of its time. I admit that I sneered at the idea at the time, because, despite those comments’ almost
aggressive enthusiasm, this method doesn’t actually work on the hard problems. But as my outlook on life worsened with age, I
came to think that Ctrl-C profiling deserves a shout-out, because it’s very effective against stupid problems encountered by
lazy people operating in unfriendly environments.

I mean, I’ve tended to dismiss the stupid problems and focus on the hard ones, but is this a good approach in the real world?
Today I’m quite ready to accept that most of life is stupid problems encountered by lazy people operating in unfriendly
environments. Certainly, one learning experience was becoming such a person myself, by stepping into a senior management role [1](#fn1) and then going back to programming after a few
years. Now I’m lazy because I got used to not doing anything myself, and I’m in an environment which is unfriendly to me,
because I forgot how things work, or they no longer work the way they used to. And while I’m a bit ashamed to admit this as
someone who’s developed several profilers himself, I’m often not really in the mood to figure out how to use a profiler in a
given setting.

But, here’s a program taking a minute to start up. Well, only in the debug build; this must be why nobody fixed it, but we
really should, it sucks to wait for a full minute every time you rebuild & rerun. So I Ctrl-C the thing, and what do you
know, there’s one billion stack frames from the [nlohmann JSON parser](https://github.com/nlohmann/json), I guess it
all gets inlined in the release build; must be what they call “zero-cost abstraction” [2](#fn2). Another Ctrl-C, another call stack, coming from a different place but again
ending up parsing JSON. And I don’t know what the fix was - a different JSON parser, or compiling some code with optimizations
even in the debug build - but someone fixed it after my Ctrl-C-based report.

Or let’s say I’m trying to switch to the LLD linker from gold, to speed up the linking. Why not the even faster [mold](https://github.com/rui314/mold)? \- because I’m on MIPS, and mold doesn’t support MIPS. But LLD is pretty fast,
too; the core was written by [the same person](https://github.com/rui314), after all. And then I open a core dump
from a binary linked with LLD, and gdb is _really_ slow. Hmm. It should have been _faster_, actually, because I’ve
also added `--gdb-index`, which tells the linker to create, I guess, some index for gdb, making gdb faster than its
slow default behavior, which is reserved for the unfortunate people who don’t know the cool flags. But I’m not seeing faster,
I’m seeing slower. What gives?

So, I run gdb under gdb, and Ctrl-C it while it’s struggling with the core dump. There’s some callstack with
`dwarf_decode_macro_bytes`. Google quickly brings up some relevant issues, such as “ [Using -ggdb3 and linking with ld.lld leads to cpu/memory hog in\
gdb](https://sourceware.org/bugzilla/show_bug.cgi?id=24624)” (Status: **UNCONFIRMED**) and “ [lld doesn't generate\
DW\_MACRO\_import like ld.bfd does](https://bugs.llvm.org/show_bug.cgi?id=42030)” (Status: **RESOLVED WONTFIX**.)

Apparently gcc generates some DWARF data that gdb is slow to handle. The GNU linker fixes this data, so that gdb doesn’t end
up handling it slowly. LLD refuses to emulate this behavior of the GNU linker, because it’s gcc’s fault to have produced that
DWARF data in the first place. And gdb refuses to handle LLD’s output efficiently, because it’s LLD’s fault to not have handled
gcc’s output the way the GNU linker does. So I just remove `-ggdb3` \- it gives you a bit richer debug info, but it’s
not worth the slower linking with gold instead of LLD, nor the slowdown in gdb that you get with LLD. And everyone links happily
ever after.

Which goes to show that **Ctrl-C profiling is often enough to solve a simple problem, and it’s usually much easier than**
**learning how to use a profiler and how to properly read its output**. You can connect a debugger to almost anything, all
the way down to some chip with nothing like a standard OS that could work with a standard profiler. You can connect a debugger
to almost anything especially if it’s slow - for example, maybe it’s hard to actually invoke the program under gdb because its
invocation is buried somewhere very deep, but if it’s slow, you can `gdb /proc/$pid/exe $pid` after it was
started.

A debugger also needs less to work with than a profiler. Unlike [perf](https://perf.wiki.kernel.org/index.php/Tutorial), gdb will give you a callstack even if [the program was compiled without frame\
pointer support](https://www.brendangregg.com/blog/2024-03-17/the-return-of-the-frame-pointers.html). And you certainly don’t need a special build, like [gprof](https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_mono/gprof.html)’s `-pg`, or to run on a slow
simulator, like [callgrind](https://valgrind.org/docs/manual/cl-manual.html) / [KCachegrind](https://kcachegrind.github.io/html/Home.html). And then the output of a profiler might be easy to
misinterpret - and I’ve only scratched the surface [the last time I wrote about it](https://yosefk.com/blog/how-profilers-lie-the-cases-of-gprof-and-kcachegrind.html).
Eyeballing a few callstacks is more straightforward.

Why then do we need profilers at all? Here is a very partial list of reasons, in no particular order.

Let’s say, completely hypothetically, that you’ve switched to the LLD linker, and your program is now 2-3% slower. If you
Ctrl-C it, you’ll see the same callstacks as with the version linked with gold. But if you have a profiler running on a
simulator, similarly to callgrind, then you can find the functions with the most slowdown - and they might not be the ones
taking the most time overall, they just have the most slowdown relatively to the old version - and then **you can look at**
**the assembly listings and see how much time was spent running each instruction**. And then you’ll see that the new
version has branch-to-address-from-register instructions where the old version had branch-to-constant-offset instructions.

Then you will learn about MIPS “relocation relaxation” (used also in RISC-V AFAIK.) The compiler “assumes the worst” and
generates code loading a function address into a register, and then jumping to the address stored in that register. Then, if
you’re lucky, the linker realizes that it has actually placed the function close enough to the caller for that caller to branch
to the function using a constant offset. (Fixed-sized RISC branch instructions cannot encode constant offsets larger than a
certain value, so the function needs to be close enough to the caller for the distance to fit into the offset encoding.) And
then the linker “relaxes” the expensive branch-from-register instruction into a cheaper branch-to-constant-offset instruction.
And it turns out that the LLD version you’re using doesn’t implement relocation relaxation.

Of course you, or should I say me, wouldn’t need that very, very fancy simulator-based profiler if you weren’t the idiot
using LLD 9 when LLD 14 was already available, with relocation relaxation implemented back in LLD 10. (I wish I’d saved the
discussion in the mailing list around this patch; now I can’t find it anywhere. There was nobody confident enough in their MIPS
knowledge to review the patch, but you don’t merge patches without a review, do you? There was even a message saying “Happy
anniversary to the relocation relaxation patch!” a year after it was submitted without having been merged. Eventually someone
said something like “we have to either merge or reject it, or we’re being rude” and someone else said “well, the patch author
knows MIPS better than any of us, so let’s just merge it.”)

But, despite having been an idiot here, I maintain that you don’t have to be an idiot to have this sort of problem, which a
profiler will help solve, and Ctrl-C profiling will not.

The broader issue is that **Ctrl-C is essentially a sampling profiler** \- one with an unusually low sampling
frequency, but a sampling profiler nonetheless. Very small changes spread across a program are obviously invisible to a sampling
profiler. Also, **[sampling profilers are bad at tail latency](https://danluu.com/perf-tracing/)** \- if
something is usually fast but occasionally slow, you won’t be there to Ctrl-C it when it’s slow. (Of course, if “slow” means 100
ms instead of the usual 25 ms, you wouldn’t manage to Ctrl-C it in time even if you were there - that low sampling frequency
comes with some downsides.)

Systems involving many threads, processes or machines… our esteemed “random pausing” technique, aka Ctrl-C profiling, is
often not great to use with these. And at this point I feel that the idea of replacing all of the various profilers with Ctrl-C
is too ridiculous to bother with more counterarguments.

**But, there are many various kinds of profilers, making it a question which kind to use, and how much legwork finding**
**the problem will take on top of using it**. Simulation-based profilers don’t have the problem of losing data to a low
sampling frequency - they analyze full instruction traces - but they’re too slow for anything like a production environment. So
you might need some measurements that you can run in production, and then a way to rerun the program on the simulator using
inputs that were observed to cause a slowdown in production based on these measurements. Tracing profilers like [ftrace](https://www.kernel.org/doc/html/v5.0/trace/ftrace.html) / [KernelShark](https://kernelshark.org/Documentation.html) are great for looking at examples of tail latency, but they
will not reliably take you to the places in the code where the time is spent. Sampling profilers can run in production and take
you to the right place in the code, but they’re a poor match for code that runs slowly but only occasionally, and even worse for
code that occasionally gets stuck waiting for something. And most of these tools have a bunch of non-trivial prerequisites,
config knobs and likely ways to misread their output.

Conversely, Ctrl-C in a debugger is easy, makes you look very effective when it actually works, and costs almost nothing to
try even when it doesn’t really help in the end. What’s not to like?

I often find myself recommending something primitive or ugly, which [might\
actually do better than the “proper” approach](https://yosefk.com/blog/refix-fast-debuggable-reproducible-builds.html#anti-thesis-sed---nasty-brutish-and-short), or it might have [less risky\
failure modes in the hands of typical users](https://yosefk.com/blog/dont-ask-if-a-monorepo-is-good-for-you-ask-if-youre-good-enough-for-a-monorepo.html), or it might be [easier to tailor to your needs than a more elaborate\
solution](https://yosefk.com/blog/how-to-make-a-heap-profiler.html). “Profile with Ctrl-C” fits right in - certainly very primitive, yet often compares surprisingly favorably with
more sophisticated alternatives. And therefore, I must give Ctrl-C profiling my warmest endorsement!

_Thanks to Dan Luu for reviewing a draft of this post._

* * *

1. In my Russian-speaking mind, “stepping into” is strongly associated with “stepping into shit.” I’m not sure
   there’s an idiomatic English synonym for stepping into something strongly implying that this something is shit; there should be
   \- it’s a very useful thing to have in a language. [↩︎](#fnref1)

2. “Zero-cost abstraction” is a figure of speech popular with people who don’t consider time spent compiling,
   deciphering compiler errors, debugging, or running the debug build as a “cost.” It would be more accurate to call it “zero cost
   in production machine resources,” though even that is quite often incorrect. [↩︎](#fnref2)
