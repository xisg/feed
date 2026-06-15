---
title: Profiling in production with function call traces
url: https://yosefk.com/blog/profiling-in-production-with-function-call-traces.html
published: "2025-02-05T00:00:00Z"
feed: yosefk
---

# Profiling in production with function call traces

**A timeline showing function call and return events is a great way to debug performance problems, especially in**
**production**. In particular, it's often much more effective than traditional sampling profilers, for reasons we’ll
discuss. However, the adoption of function tracing in the industry remains uneven because of a chicken-and-egg problem.

To best use a tracing profiler, you need some adaptations to your code and your workflow (as opposed to sampling profilers,
which “just work” with your code.) So to _make_ a tracing profiler, one needs people wishing to change their code &
workflow in order to use it. That said, as we’ll see, **it’s gotten fairly easy to develop a tracing profiler**
**today,** and integrating it into your work is very doable as well – which I hope might encourage people to both make and
use tracing profilers.

“Our main contribution,” as they say in papers, is a new function tracing profiler for C++ imaginatively named [funtrace](https://github.com/yosefk/funtrace). It’s ready for serious use – it works out of the box on large,
complicated programs. For example, here it is peeking into the inner workings of the awesome painting program [Krita](https://krita.org/), showing how the call stack changes over time, the thread state transitions, and the source
code of some selected function:

![image8.png](https://yosefk.com/img/funtrace/image8.png)

Funtrace has the following attractive qualities, which I hope I am not overselling:

- AFAIK, has **the lowest-overhead tracing** (<10 ns per instrumented call or return in my measurements)
- Supports **threads, shared libraries and exceptions**
- Supports [ftrace events](https://www.kernel.org/doc/html/v5.0/trace/events.html), showing **thread**
  **scheduling states** alongside function calls & returns, so you see when time is spent waiting as opposed to
  computing
- Works with **stock gcc or clang** \- no custom compilers or compiler passes
- Easy to integrate into a build system, and even easier to **try _without_ touching the build system**
  using [tiny compiler-wrapping scripts](https://github.com/yosefk/funtrace/tree/master/compiler-wrappers) “passing all
  the right flags”
- **Small** (just **~1K LOC** for the runtime) and thus:

  - easy to port (currently **x86/Linux** is supported)
  - easy to extend (say, to support some variant of “green threads”/fibers)
  - easy to audit in case you’re reluctant to add something intrusive like this into your system without understanding it well
    (as I personally would be!)
- **Relatively comprehensive** – it comes with its own tool for finding and cutting instrumentation overhead in
  test runs too large to fully trace; support for remapping file paths to locate debug information and source code; a way to
  extract trace data from core dumps; and other such “ways to address real-world concerns.”

I’ve worked on several kinds of profilers during the last 20 years, and this one is easily my favorite. Perhaps it’s the
colorful stalactites?.. Anyway, that’s “our main contribution”; we’ll see how funtrace works, and how you could use similar
methods and existing components to build your own tracing profiler.

But we’ll cover more than the proverbial design and implementation of funtrace; in fact, we’ll leave a few of its
particularly “hardcore” bits for their own followup. We’ll start with [viztracer](https://github.com/gaogaotiantian/viztracer), a great tracing profiler for Python, and discuss how to
introduce a profiler like that into your workflow. We’ll also talk about [LLVM\
XRay](https://llvm.org/docs/XRay.html), AFAIK “the” open-source function tracing C++ profiler today, and how its design is influenced by what the workflow
needs to be in a place like Google, where XRay comes from.

We’ll also get to the awesome [magic-trace](https://github.com/janestreet/magic-trace), a non-intrusive tracing
profiler built on top of [Intel Performance Trace](https://perfwiki.github.io/main/). Unfortunately, its authors say
that magic-trace is [hard or\
impossible to port to many popular platforms](https://github.com/janestreet/magic-trace/wiki/How-could-magic-trace-be-made-to-work-on...). This will bring us to **what CPU makers could do to make**
**hardware-accelerated function tracing very cheap** in _both_ hardware and software – and usable much more widely
than today, including in dynamic and JITted languages.

- [viztracer: how to use a great tracing profiler](#viztracer-how-to-use-a-great-tracing-profiler)
- [Funtrace: making a tracing profiler for native code](#funtrace-making-a-tracing-profiler-for-native-code)
  - [Compiler instrumentation](#compiler-instrumentation)
  - [Runtime code](#runtime-code)
  - [Decoded trace viewer](#decoded-trace-viewer)
  - [Offline trace decoder](#offline-trace-decoder)
  - [ftrace: tracing thread state changes](#ftrace-tracing-thread-state-changes)
  - [Getting traces from core dumps](#getting-traces-from-core-dumps)
  - [`funcount`: culling overhead](#funcount-culling-overhead)
- [Antithesis: LLVM XRay](#antithesis-llvm-xray)
- [Synthesis: funtrace with XRay characteristics](#synthesis-funtrace-with-xray-characteristics)
- [Hardware-assisted tracing](#hardware-assisted-tracing)
- [Conclusion and future work](#conclusion-and-future-work)

## viztracer: how to use a great tracing profiler

I’m working on a small animation program, and I’ve found out that there’s such a thing as insufficient [yak shaving](https://en.wiktionary.org/wiki/yak_shaving) \- it turns out that the care and feeding of an unshaved yak
gets tiresome quickly. For example, I’ve avoided figuring out how to use a profiler for way too long, and in hindsight, wasted a
lot of time manually putting timers into the code.

I mean, you start putting timers into your code, and the first thing you print out is the average runtimes of things. The
averages tell you which code the program spends most of its time running, helping you to speed up some of this code. But then
you run into the occasional lags, and of course, **[averages can’t explain why\**
**you have unusual lags](https://danluu.com/perf-tracing/) \- because by definition, _on average_, you don’t have _unusual_ lags.** So
whatever is taking time when you _do_ have unusual lags doesn’t move the averages.

(Incidentally, this is why sampling profilers like [perf](https://perfwiki.github.io/main/), which periodically
check what the program is doing and then show you what it was doing most of the time, can help with saving CPU cycles, but
mostly **can’t help with worst case latency**.)

So you print out worst case runtimes, but of course the worst case runtimes of each function don’t help you _by_
_themselves._ What you’re really after is how long _each part_ of your code took when _an entire flow,_ like
your mouse-down event handling, was unusually slow. That slowness wasn’t due to every function taking the most time ever, but
due to _some_ of them taking a lot of time, and the sum of\* everything\* running _in this flow_ taking too
much.

So you start printing some sort of tables with timer values - a row for every time some flow ran, and a table like that for
every flow, and then you look at the rows with the most total cycles. The tables grow, and now you want something easier to look
at than tables of numbers. So you start having thoughts like “I could decorate my functions with @trace or something, to trace calls and returns, and if only there was a nice way to display this
trace with the function calls nested inside each other...”

And at this point you say – you know, _I would be building a tracing profiler!_ There has got to be a tracing profiler
for Python - I should find one and use it! Where’s my yak shaving machine?! Should have reached out for that long ago, before
getting all tangled up in all this yak fur!

Then you discover viztracer, which looks great, and works like a charm if you run it on some script with
`viztracer ./script.py`. Works fine for a short program run: you get the last 10 million function calls at the end of
the run. But your program is an interactive GUI, which means an “infinite” program run. You don’t want to quit the program to
get the trace. Well, you can Ctrl-C the program - more accurately, you can Ctrl-C viztracer which is running the program - to
get the last 10M calls before you Ctrl-Cd. But [how do you know when\
to Ctrl-C](https://yosefk.com/blog/profiling-with-ctrl-c.html)?

**This right here is the major problem with tracing profilers: _you need to figure out what to trace._** With sampling profilers like perf, no such problem: you sample _all the time_, and then you get a summary – of a
size _not dependent on how long the program ran._ And this is important not only because there are only so many terabytes
in a disk, but first and foremost because _there’s only so much time to scroll horizontally_, trying to find the part of
a giant timeline that you care about.

**Therefore, a tracing profiler usually comes with an API for triggering tracing, which the program must call**.
And this is our chicken-and-egg problem: when perf came out, it was immediately usable for all the natively compiled programs
out there – and everyone looking into performance could use it, and wanted to make it better. But with a tracing profiler, most
programs must be changed for it to be usable, if only a little bit, and who wants to risk developing a tool that nobody can use
on day one?

(There is, of course, _the other_ major problem with tracing profilers - their larger overhead compared to sampling
profilers. This IMO explains why **dynamic languages are more likely to have a tracing profiler than static languages at**
**this time**. Not only are these languages designed for things like intercepting function calls without the language maker
having to add support for this, making things easier for “community tool makers,” but **the execution is so slow to begin**
**with that the overhead of a tracing profiler is _relatively_ smaller than in static languages,** and thus doesn’t
deter tool makers and users alike as much. We’ll talk about the overhead later, when we get to compiled languages.)

Anyway, you use the tracing API:

```
tracer = viztracer.VizTracer()
tracer.start()

init_flow()

tracer.stop()
tracer.save('trace.json')
```

You start with your init flow, if only because there’s just one such flow, as opposed to the many runtime event handling
flows. You get your trace.json file, and you run `vizviewer trace.json`. A browser tab pops up, and in that tab, you
see this message:

![image1.png](https://yosefk.com/img/funtrace/image1.png)

At this point, I hope you brought your big yak shaving machine. If, like me, you’ve only brought the small one, this is where
it breaks (“great, I knew it was a waste of time to look for a tracing profiler, of course this stuff comes broken out of the
box”) and you go back to looking at tables of numbers for a while, until this irritates you enough to try again. Then you find
out that your init flow was long enough to trigger a known bug that [likely won’t be fixed](https://github.com/gaogaotiantian/viztracer/issues/139), but there’s a fine workaround –
**all you need to do if vizviewer gives you “Error: RPC framing error” is to reopen trace.json from the web UI [1](#fn1).**

Happy happy, joy joy, you think to yourself, and get busy profiling your runtime flows. You open the traces – wow, so much
better than tables of numbers!

![image2.png](https://yosefk.com/img/funtrace/image2.png)

There it is, my [wonderfully optimized resizing\
function in “unsafe Python”](https://yosefk.com/blog/a-100x-speedup-with-unsafe-python.html) \- and to its left, a bunch of short calls, looks like it’s drawing lots of buttons in a loop -
maybe it’ll be faster to draw them as one larger image?.. _Way_ nicer than the tables!

The colorful stalactites are reminiscent of [flamegraphs](https://github.com/brendangregg/FlameGraph) [2](#fn2), though they _aren’t_ flamegraphs - they
represent _the execution timeline,_ not _the share of time spent per callstack_. Vizviewer can show actual
flamegraphs, too - pass `--flamegraph`. In our example, instead of the many little calls on the left in the
screenshot above, you will get the following succinct summary (with the functions colored differently – done by different code,
I guess?..):

![image3.png](https://yosefk.com/img/funtrace/image3.png)

Note that this is the _exact_ flamegraph of a _short_ period of time captured in a trace – while a sampling
profiler shows you an _approximate_ flamegraph of a _long_ period of time, a very different thing.

In any case, now that tracing basically works, you have a simple playbook:

- When you start handling an event, create a tracer object, and **start()** tracing
- When you’re done, **stop()** tracing, and check how long it took to handle the event. Only keep the tracer
  objects upon the slowest measurements
- Eventually, **save()** the traces you kept

As you follow this playbook, you run into some issues:

- After creating the 1022nd VizTracer object (whether the previous 1021 were destroyed or not), the process terminates with
  the somewhat paradoxical error message `Failed to create Tss_Key: Success`. Some resource must be leaking - so let’s
  keep a pool of VizTracer objects and reuse them.
- One of the flows you trace invokes another flow you trace, and then you get a
  `Warning! Overwrite tracer! You should not have two VizTracer recording at the same time!` So you stop tracing when
  entering a nested flow, and restart it when that flow is done.

But, it’s not that many issues. **You do need to write some code around a tracing profiler to make it work for you –**
**but not a lot, and it’s well worth your trouble**, certainly with viztracer, which is absolutely great [3](#fn3). [Here’s my code for this](https://github.com/yosefk/Tinymation/blob/master/pyside/trace.py), presented mainly to show
that it’s <150 LOC – there’s not much to it.

And now that we’ve seen how useful they are, let’s make our own tracing profiler!

## Funtrace: making a tracing profiler for native code

To trace function calls in a compiled language, you need 4 main things:

- **Compiler instrumentation** for running code upon function entry & exit [4](#fn4)
- **Runtime code** collecting some sort of function IDs and timestamps when functions are called and return
- **Offline trace decoder** for converting the traced function IDs into symbolic names, and producing some format
  for the…
- … **Decoded trace viewer**– a UI for looking at the traced timeline

Actually, you also need a 5th thing, which one might call the first – namely, **assumptions about the user’s**
**workflow**: what the user needs to do, is willing to do, and is _not_ willing to do. For funtrace, these
assumptions are:

- **The user either _wants_ or _agrees_ to trace in**
  **production.** The user might _want_ this because performance problems happen in production, can be hard to
  reproduce, and you want to debug them. If the user isn’t interested in debugging problems in production, we still hope they
  _agree_ to trace in production, or at least in acceptance tests run before releasing production versions. That’s because
  _tracing overhead can become unacceptable unless continuously monitored and culled as needed_. Tracing during acceptance
  testing guarantees that the overhead is acceptable, by the definition of “acceptance testing.” And then you can look at trace
  data any time you want, without worrying that this data is irrelevant due to high tracing overhead. Conversely, _not_ tracing during acceptance testing virtually guarantees that _when you’ll actually enable tracing, the overhead will be_
  _unacceptable, and you won’t have time to cull it,_ making tracing unusable when you need it.
- Therefore, as a corollary of tracing in production, **the user agrees to continuously monitor and cull tracing**
  **overhead** by manually specifying things like “never trace these several functions,” “don’t trace when we’re loading
  files - we know this flow is always slow, and tracing it does nothing except making it even slower,” etc.
- **The user knows when to collect trace data and which collected data to save**, and will use our API to do it.
  For example, “start tracing when event handling begins, keep the trace of the slowest processing of every type of event, and
  save all these slow traces upon program exit.”

I like these assumptions for two reasons: **this is the workflow I want as a user**, and **you get a small**
**and fast runtime with these assumptions**. However, this is not the only possible set of sensible assumptions, and we’ll
see below how very different assumptions influence the design of LLVM XRay.

And now with our workflow assumptions in mind, let’s think step by step, as we tell LLMs when we want to guide their
boundless creativity away from complete bullshit, and work our way through the list of key components in a tracer.

### Compiler instrumentation

With any native language using LLVM, you can write a compiler pass calling some event handlers upon function entry & exit
– and with GCC as well, though most would prefer LLVM’s APIs for this.

Specifically in funtrace, however, my goal was **to use existing compiler flags for instrumentation**.
Specifically with C++, g++ and clang++ make this possible, and compiler flags are more consistent across compiler versions than
the internal APIs for writing a compiler pass. And even if my pass supported multiple compiler versions, who’d want to build it
for their specific compiler version so as to try funtrace?..

g++ and clang++ have the following flags, **all supported by funtrace**, and each having its own pros and
cons:

- Both compilers support `-finstrument-functions`, which makes the compiler generate calls to
  \_\_cyg\_profile\_func\_enter/exit when functions are, well, entered and exited. Very clean – you can write your tracing handlers in
  portable C code. For better or worse, this instruments functions _before inlining,_ and C++ code is famously full of tiny
  inline functions [5](#fn5). It can be neat to trace a
  program with -finstrument-functions to inspect the flow – might be easier to follow than either in a debugger or an IDE – but
  it’s impractical for tracing in production. Can we lower the overhead?

  - With g++ you can pass something like `-finstrument-functions-exclude-file-list=.h,.hpp,/usr/include` to ignore
    all the functions in the header files – close to, but not quite what I’d want.
  - With clang++ you can simply use `-finstrument-functions-after-inlining`, often exactly what you want in
    production.
- g++ supports the not-too-obvious flag combination `-pg -mfentry -minstrument-return=call`. This is similar to
  clang’s -finstrument-functions-after-inlining in that it, well, instruments functions after inlining. But this doesn’t call
  \_\_cyg\_profile\_func\_enter/exit – it calls \_\_fentry\_\_ and \_\_return\_\_, and it calls them in a different way, pretty much forcing
  them to be assembly functions – though this is actually a plus, since it lowers the overhead somewhat; bringing us to the run
  time code implementing all these functions called by compiler instrumentation.

### Runtime code

Our trace entries keep a timestamp, and a pointer into the code of a function. The highest bit of the code pointer is used to
mark an entry as “call” or “return” (no machine actually uses all the 64 bits of a pointer in userspace.)

Getting a cycle-accurate timestamp is fairly cheap; x86 has the so-called TSC (timestamp counter), which you read with the
RDTSC instruction. We’ll discuss alternatives to TSC on x86 and elsewhere in our “hardcore followup.”

We keep thread-local cyclic buffers of these entries. The user can dump them in full in one of 2 ways:

- **Call `funtrace_pause_and_write_current_snapshot()`.** We pause tracing while writing the snapshot,
  to not overwrite the data with events logged after the call.
- **Run `kill -SIGTRAP <pid>`** (similarly to viztracer’s and magic-trace’s Ctrl-C/SIGINT; I
  prefer SIGTRAP, since many programs handle SIGINT themselves.)

As we’ve seen above, "suddenly dumping the whole trace" like this works for peeking into programs you know nothing about, but
it’s not great. Dumping the full content of the buffers is costly in time & space, and then _looking_ at this data
gets annoying, too. For example, some threads are idler than others, and keep very old events in their buffers thanks to this
idleness. These old events cause the timeline UI to zoom out so much that you can’t see anything.

You can pass flags to the funtrace decoder asking to ignore some threads, or to ignore events past a certain age. But it’s
actually much easier to know _at the time you’re taking the snapshot_ what that age should be:

```
uint64_t start_time = funtrace_time(); //wraps RDTSC

do_stuff();

uint64_t latency = funtrace_time() - start_time;

if(latency > _slowest) {
  _slowest = latency;

  funtrace_free_snapshot(_snapshot);
  _snapshot = funtrace_pause_and_get_snapshot_starting_at_time(start_time);

  //eventually we’ll write _snapshot out with
  //funtrace_write_snapshot()
}
//else (if latency <= _slowest, the typical case),
//the only overhead added by our tracing logic
//is the 2 funtrace_time() calls

```

Now the snapshot only keeps “the interesting part” – small, and easy to look at. Tracing is always on, so the program can
decide at any moment that “something interesting” happened, and save the recent events according to the appropriate definition
of “recent.” [6](#fn6)

Appending to a cyclic buffer is easy; the ANDing of the current _pointer_(not _index_) with a mask is the only
slightly tricky thing (see Hardcore Follow-up.)

```
void trace(uint64_t code_ptr, uint64_t flags)
{
    //trace_buf is declared thread_local
    uint64_t buf_ptr = (uint64_t)trace_buf.pos;
    buf_ptr &= trace_buf.wraparound_mask;
    event* entry = (event*)buf_ptr;
    if(!entry) {
        return;
    }
    entry->func = code_ptr | flags;
    entry->cycle = __rdtsc();
    trace_buf.pos = entry + 1;
}

```

The flags argument is 0 for calls and 1<<63 for returns [7](#fn7). To pause tracing, we set wraparound\_mask to 0. Since we must have the “if(!entry)” for
pausing, we might as well also set the mask to 0 to support **disabling tracing at runtime, which cuts ~85% of the**
**overhead in my tests**.

(Note that **the code above is racy** \- it might take some time until a thread reads the zero from
wraparound\_mask which another thread pausing the tracing wrote; we don’t particularly mind - at worst, we’ll overwrite a few old
events. Likewise, some recent writes might not be visible to the snapshotting code reading the buffers - we don’t particularly
mind, either [8](#fn8).)

The “nice” C callbacks \_\_cyg\_profile\_func\_enter/exit simply call the trace() function above. Things are harder for the
\_\_fentry\_\_ & \_\_return\_\_ callbacks. Firstly, they aren’t passed the address of the function calling them as an argument - but
we could get that with \_\_builtin\_return\_address(0) [9](#fn9). More importantly, **they aren’t called according to the C calling convention**
\- the compiler “just calls them,” without bothering to save registers where their caller’s arguments might be kept, for
example.

I don’t know how to tell gcc or clang, “please don’t use registers where arguments are passed - please only use temporary
caller-saved registers.” And if you implement \_\_fentry\_\_ in C, and the compiler clobbers a register where arguments are passed,
and \_\_fentry\_\_ was called from a function which gets arguments, that function’s argument will have been clobbered.

So I wrote these functions in x86 assembly, basically by taking what the compiler produces from
`trace(__builtin_return_address(0), flags)` and then changing the code to only use those registers that I am allowed
to in this “non-standard calling convention” – or saving those registers that I can’t help using, but shouldn’t clobber.

(RDX and RAX are the annoying ones. RDTSC is hardwired to clobber them, but they’re also used for an argument and the return
value, respectively, so it works out to \_\_fentry\_\_ having to save RDX, and \_\_return\_\_ having to save both. Many more appetising
details of this sort await in the Hardcore Followup; generally, **writing a tracing profiler today involves figuring out**
**many small and simple, if somewhat arcane things, but not a lot of code** \- the perfect job for myself, I find.)

Of course, just the content of the buffers is not enough to make sense of the snapshot once it was dumped. We also need to
know:

- **Where the code was loaded to** \- the executable and each shared library get loaded to a different offset,
  potentially in every run if [ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization) is enabled, and
  we have to subtract this offset to convert code pointers to function names using symbol table lookup. We can get this info from
  `/proc/self/maps`, but it’s faster to get it from [dl\_iterate\_phdr](https://man7.org/linux/man-pages/man3/dl_iterate_phdr.3.html) (and _maybe_ more portable, eg
  Android reportedly won’t let you access files under /proc)
- **The TSC frequency** to convert cycles to nanoseconds. There are at least 3 methods to find it out - using the
  CPUID instruction (specifically “leaf 15H” - don’t ask), grepping the output of `dmesg`, and simply sleeping for some
  time and checking by how much TSC was incremented during that time. Funtrace tries all these methods, in that order, in case the
  first two fail (the 3rd kinda can’t fail, but it’s not very accurate.)

With that, we can decode the code pointers and the timestamps, respectively. One last nice thing to save is **thread**
**names**. The OS spends lavishly to let us userspace peasants name our threads - 15 (!!) characters per thread (16 with
the null byte), and funtrace reads these names with `pthread_getname_np` [10](#fn10).

That’s it - we have our snapshot.

### Decoded trace viewer

We need to decode the trace before viewing it. But first, we need to decide what the viewer will be, to make our decoder emit
the trace in the viewer’s format. Vizviewer, for example, is based on **[Perfetto](https://ui.perfetto.dev/)**. In fact, it turns out that **every tracing profiler mentioned in**
**this post (viztracer, magic-trace, XRay) uses Perfetto for the viewer**.

I assume that Perfetto owes its popularity to its quick rendering of very large traces at arbitrary zoom, its beautiful look
& feel, and its [simple JSON trace\
format](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/) \- just a bunch of records with a name, start timestamp, duration, and thread ID:

```
{
"traceEvents": [
{"args":{"name":"my_program"},"name":"process_name",
    "tid":1,"pid":1,"ph":"M"},
{"args":{"name":"main_thread"},"name":"thread_name",
    "tid":1,"pid":1,"ph":"M"},
{"name":"run()","ts":2,"dur":10,"tid":1,"pid":1,"ph":"X"},
{"name":"parse()","ts":5,"dur":5,"tid":1,"pid":1,"ph":"X"},
{"name":"open()","ts":6,"dur":3,"tid":1,"pid":1,"ph":"X"},
{"args":{"name":"worker_thread"},"name":"thread_name",
    "tid":2,"pid":1,"ph":"M"},
{"name":"config()","ts":3,"dur":4,"tid":2,"pid":1,"ph":"X"},
{"name":"open()","ts":4,"dur":2,"tid":2,"pid":1,"ph":"X"}
]
}

```

Note how we didn’t tell which function called which - Perfetto just finds which time ranges are nested within other time
ranges, and stacks them accordingly:

![image6.png](https://yosefk.com/img/funtrace/image6.png)

Today, however, I don’t recommend following vizviewer’s example and using Perfetto. Much better to do what vizviewer couldn’t
do - **use vizviewer itself!**

A big reason is that **vizviewer extends the JSON format to include the source code of functions** (and unlike
_every damned debugging and profiling tool_, here’s a program finally doing the right thing and putting _the source_
_code_ into the JSON instead of _file names_– so that you actually look at the source code _that was traced_,
and not the code appearing in those files _right now,_ possibly with some newer changes! And you can also send these JSON
files to someone and they’re self-contained, and they’ll open on their machine – unlike typical tool reports referencing source
code.)

So `funtrace2viz`, our trace decoder, simply produces a vizviewer JSON; to view it, install viztracver with
`pip install viztracer`, and you’ll get vizviewer in your $PATH. And now all we need is an…

### Offline trace decoder

Our trace entries have 2 fields – a code pointer and a cycle – so decoding involves 2 jobs:

- Converting code pointers to function names and source line numbers.
- Converting cycles to microseconds.

Therefore, we should do this in Rust, which has excellent libraries for both tasks.

_Libraries_ for _both_ tasks, you think; the 2nd task being multiplication of numbers. I guess this guy’s rabid
hatred of C++ metastasized into rabid Rust fandom, you think; sad, if unsurprising – a textbook example of mental illness.

Well, ackchyually, I’m a less rabid Rust fan than I’d like; I’m afraid that if you’re into stuff involving a mix of GUI and
number crunching, a combination of C++ and Python is your best bet today, if only because these are the two widely popular
languages which people use for this stuff, and where most of the libraries and tools are [11](#fn11).

That said, yes, converting cycles to microseconds _is_”a task,” as evidenced by the following comment in XRay’s
code:

```
// Chrome trace event format always wants data in micros.
// CyclesPerMicro = CycleHertz / 10^6
// TSC / CyclesPerMicro == TSC * 10^6 / CycleHertz == MicroTimestamp
//
// Could lose some precision here by converting the TSC to
// a double to multiply by the period in micros. 52 bit
// mantissa is a good start though.
//
// TODO: Make feature request to Chrome Trace viewer to
// accept ticks and a frequency or do some more involved
// calculation to avoid dangers of conversion.

```

You see, TSC is a 64b number, and if your machine has been running for a while, it will have more than 52 significant bits,
and you will start losing the low bits, because they won’t fit into a double’s mantissa. Now, in Rust, all I had to do to avoid
this precision loss was \`cargo add [num](https://docs.rs/num/latest/num/) \`, and then use
`Ratio<BigInt>` for the conversion.

But if it was C++, while you can find a library for this, you would want to avoid the dependency – because without a standard
build & packaging system, dependencies are a major PITA. So I’d just leave a TODO like they did in XRay.

**Rust is the fastest popular language with a standard package manager**. _This alone_ will make you
extremely productive in areas it has good libraries for, if you’re looking to minimize the product of machine time x developer
time. **Mature support and widespread use of binary packages for C++ code would greatly boost Rust’s**
**applicability!** For instance, `pip install` ing Python bindings wrapping C++ libraries is way easier than
managing these libraries as source dependencies. If Rust could become “a packaging system for C++” like Python effectively is,
it would immediately become very tempting to use just for this reason! [12](#fn12)

Of course, our bigger task is parsing ELF (with [goblin::elf](https://docs.rs/goblin/latest/goblin/elf/index.html)) and DWARF (with [addr2line](https://docs.rs/addr2line/latest/addr2line/)) \- and we need to parse both. Only DWARF has line info, but
only ELF has symbols containing some of the code pointers – for example, gcc doesn’t bother to produce DWARF debug info for
“thunks” it generates. What’s a thunk? Well, there are “virtual thunks” and “non-virtual thunks” according to the C++ demangler
(cargo add [cpp\_demangle](https://docs.rs/cpp_demangle/latest/cpp_demangle/)); I’m sure this means something, but I
don’t care exactly what it is – I just want some name related to the source code at least somewhat instead of bare hex
garbage.

Which reminds me – and I’m sure experienced programmers have seen it coming – we actually have _three_ jobs:
converting code pointers to names, converting cycles to microseconds, and dealing with random shit. Examples of the latter:

- Detecting and ignoring “virtual override thunks,” which for some reason call \_\_return\_\_ but not \_\_fentry\_\_
- Handling “orphan returns” (when the function call was overwritten in the cyclic buffer and we only see the return
  event)
- Handling exceptions, its own section in the Hardcore Followup
- Handling “strange missing returns” when f called g which called h, but h returns straight to f (eg because setjmp/longjmp
  were used)
- Remapping pathnames according to substitution rules provided by the user, in case the files aren’t where the debug info says
  they should be (which happens way more often than it should - see [an easy way to avoid such issues](https://yosefk.com/blog/refix-fast-debuggable-reproducible-builds.html))

But, that’s about it. I dwell on the details in part to show that **it’s not that much work**, even if you’re
aiming to cover enough ground for “serious uses” - threads, exceptions, shared objects, multiple compiler instrumentation
options, etc. etc. **The decoding is about 1K LOC**, same as the runtime (but with way more library
dependencies!)

One last item to file under “random shit” is converting _ftrace timestamps_(similar to, but uglier than converting
our trace entry timestamps), bringing us to…

### ftrace: tracing thread state changes

A function tracer traces function calls, and since we just did, we could use this tautology to declare that our job is done.
However, you’ll wonder if a function taking a long time was actually _computing_ something or _waiting_ for
something – so you need to know whether the thread was in a running state or not.

Linux can trace many kernel events, including scheduling events. A userspace peasant with the right permissions (eg
`sudo chown -R $USER /sys/kernel/tracing`) can configure ftrace to log thread scheduling events. You get the latest
events from the kernel buffer with `cat /sys/kernel/tracing/trace` (and no, you don’t need to actually understand the
example log below to follow what’s next - just showing it for those curious about tracing on Linux):

```
#                   _-----=> irqs-off
#                  / _----=> need-resched
#                 | / _---=> hardirq/softirq
#                 || / _--=> preempt-depth
#                 ||| / _-=> migrate-disable
#                 |||| /     delay
#  TASK-PID CPU#  |||||  TIMESTAMP  FUNCTION
#     | |     |   |||||     |         |
  <...>-78  [003] ..... 30625460: task_newtask: pid=81 comm=main clone_flags=3d0f00 oom_score_adj=0
 <idle>-0   [004] d.... 30701644: sched_switch: prev_comm=swapper/4 prev_pid=0 prev_prio=120 prev_state=R ==> next_comm=main next_pid=81 next_prio=120
  <...>-78  [003] d.... 30747413: sched_switch: prev_comm=main prev_pid=78 prev_prio=120 prev_state=D ==> next_comm=swapper/3 next_pid=0 next_prio=120
  <...>-81  [004] d.... 30750940: sched_waking: comm=main pid=78 prio=120 target_cpu=003
 <idle>-0   [003] d.... 30780026: sched_switch: prev_comm=swapper/3 prev_pid=0 prev_prio=120 prev_state=R ==> next_comm=main next_pid=78 next_prio=120
  <...>-81  [004] ..... 30810996: task_rename: pid=81 oldcomm=main newcomm=worker oom_score_adj=0
  <...>-78  [003] d.s.. 37939679: sched_waking: comm=rcu_sched pid=15 prio=120 target_cpu=035
  <...>-81  [004] d.h.. 38466542: sched_waking: comm=code pid=9974 prio=120 target_cpu=004

```

It turns out that we don’t even need to parse this format – **Perfetto can simply read ftrace data from a**
**`systemTraceEvents` JSON key, and has special support for visualizing scheduling events.** Here’s how it looks
like:

![image7.png](https://yosefk.com/img/funtrace/image7.png)

Perfetto shows us which thread each CPU core is running at any given moment (where “swapper” is Linux-speak for “nothing”.)
To each thread, Perfetto adds a special lane showing whether its state is Running, Runnable (light green), or waiting for
something (white/blank.)

So all we have to do is collect and log ftrace events (~200 lines of funtrace’s ~1200 LOC runtime.) We configure
`trace_clock` to `x86-tsc` to synchronize timestamps with our function call/return events. We read the
scheduling events supported by Perfetto from `trace_pipe` (only listening to events from our process and its
children, or we’ll be flooded with data, including too many CPU lanes to vertically scroll through.) And we maintain a circular
buffer of events, so that we can get a snapshot of all events after some time threshold at any moment.

That’s it - the offline decoder converts ftrace timestamps from TSC to milliseconds (a bit ugly to have to massage text like
this, but no biggie; weird that you have to do this - the JSON evidently wasn’t designed for TSC, the fastest timestamping
method, but whatever.) And we’re done.

Note that there’s WAY more info that we could get from ftrace. For example, we could show how threads wait for each other
because of taking the same locks, etc. etc. I just picked the low-hanging fruit, which was enough for a certain “completeness” –
you can know both what functions the CPU was running and when it was waiting.

I don’t know why other function tracers using Perfetto don’t collect ftrace data, with Perfetto making it so easy. I think
some come from a larger system having another part doing this – and perhaps others don’t bother because users have trouble
getting permissions to access ftrace?!.. **Why do I need permissions to know when my own threads were**
**scheduled?..** (I confess up front that if there’s an answer, I might refuse to understand it. I hate permissions!)

### Getting traces from core dumps

I firmly believe that in addition to compile-time and runtime support, proper developer tools come with _coretime_
_support_. I have a morbid fascination with **coredump-oriented design: make your data structures easy to extract from**
**core dumps, and write the code to do so**.

Function traces might come handy when performing an autopsy on a core dump – it helps to see what the program was doing
before it crashed. Looking at when your threads were doing what might help understand a race condition. A null dereferencing
might become obvious once you see that the wrong flow was running. And then some core dumps might be due to performance being
too bad (eg a real time program purposely crashing during test cycles), and traces are perfect for that.

So funtrace comes with a gdb.Python extension command, imaginatively named `funtrace`. It reads the thread-local
trace buffers, as well as the ftrace events [13](#fn13). We also save the addresses where shared libraries were loaded to, which we get from gdb’s
`info proc mappings`. You get a funtrace.raw file in the same format you’d get from
`funtrace_save_snapshot()`, and you can decode it with `funtrace2viz`. At **~120 LOC**, our
“coretime” costs us just 10% of our runtime in terms of lines of code.

### funcount: culling overhead

In a microbenchmark, it takes 8-9 ns to log one trace entry in my tests, and every function call takes 2 entries. This can be
a lot or a little, depending on how much work a function does on average. From my experience, **you’ll probably want to**
**disable tracing for a bunch of short functions** to get the overall overhead down to single-digit percentage points –
what I’d call a fair price for being able to debug performance issues (it _costs_ money because it _saves_ money,
like a plumber said in some movie; up to a point, slower code that you can optimize thanks to the extra visibility ends up being
_faster,_ because you will have used more optimization opportunities.)

You can exclude a function from tracing using the `NOFUNTRACE` macro, which adds an `__attribute__` to
the function disabling compiler instrumentation. The question is, **which functions to exclude?** You could check
some traces and find often-called short functions. But this brings back the first problem with tracing profilers: what to trace?
Traces are short - perfect for understanding interesting outlier events, but not for understanding the overhead _on average_ like we want here.

The obvious approach is, **let’s sort the functions by the number of times they’re called, and consider disabling**
**tracing on those called most often**. However, what is actually not obvious is _how to count function calls_. A
sampling profiler can’t tell the difference between a long function called once and a shorter function called many times [14](#fn14). gprof collects accurate call counts – for
single-threaded programs; in multithreaded programs, the call counts are garbage. And callgrind is slow, and won’t run in many
environments.

So, funtrace ships its own tool for counting calls, `funcount` – which is nice, in particular, because **it**
**counts the exact same calls that funtrace would instrument** under whatever compiler flags you chose. It works like
this:

- **the function entry hook increments an atomic counter in a 2-level page table**. We assume that 48 bits out of
  a pointer’s 64 are actually used; given an address, we increment the counter at
  `pages[high_bits][mid_bits][low_bits]`, where each index is made of 16 out of these 48 bits, and the arrays are
  sparse (most of the page pointers are null.)
- We could allocate pages thread-safely on demand using some compare-and-swappery, but it slows things down A LOT. Instead,
  **we use dl\_iterate\_phdr upon program start and upon calls to dlopen** to find executable address space segments,
  and allocate pages only for those segments [15](#fn15).
- **We print the non-zero counts at the end of the program run**. The resulting report, funcount.txt, can be
  decoded to function names & source line numbers using funcount2sym. You can then sort by call count with `sort`,
  and combine reports from multiple runs with awk or such [16](#fn16).

There’s also a knob to tune a time/space tradeoff: if you compile with `-DFUNCOUNT_PAGE_TABLES=16`, 16 page tables
will be kept instead of one, with threads indexing into the page table cpu\_core\_number % 16 (we get the core number from
RDTSCP.) If you have many threads calling the same functions, more page tables means less fighting for the cache lines keeping
these functions’ counters – at the cost of using more memory. The final report is the sum of the counts from all the page
tables.

This thing is about as fast as funtrace, modest in memory use, takes **~250 LOC for the runtime + ~60 LOC for the**
**offline decoding** – and like funtrace itself, is easy to understand and port.

While we’re on the subject of tracing overhead, one might ask – isn’t _opt-in_ tracing better? Instead of tracing by
default and using tools to find when to opt out of tracing, why not let people trace whatever they want, without putting tracing
in behind their backs?

This approach can work well for some teams. I believe that tracing by default is better for most projects, if only because
**the code with the most surprising performance artifacts is both what you want traced the most _in hindsight,_ and**
**what was most likely _not_ traced satisfactorily up front** in an opt-in tracing regime, since nobody expected the
surprise by definition.

Another point is that an “opt-in tracing backlog,” where many people just _don’t_ opt in for a long time, can easily
grow so much that you will lose all hope to use tracing when you need it. The “opting-out” backlog _cannot_ grow so
badly, because too much tracing results in performance problems that you will _have_ to fix long before they’ll get too
daunting to even try. This is similar to the argument for tracing in production vs flipping a switch when you need tracing –
**a switch that’s off by default is unlikely to really work when you need it**.

Last but not least, **opting out is typically less work than opting in, making tracing cost less in development**
**time**. You can definitely create a culture where people carefully put tracing statements into their code; the endlessly
growing “tracing backlog” is likely, but it’s not destiny. But then changing code becomes more costly, so you’ll tend to avoid
it more often – a problem in its own right. The same thing happens with other “good things,” like documentation and tests – they
make your system better, but you take more time making it, and then balk at making major changes. The nice thing about tracing
is that unlike documentation and tests, you can have it done mostly automatically with a bit of manual intervention.

## Antithesis: LLVM XRay

The thesis behind funtrace is that the user:

- traces in production,
- monitors & culls tracing overhead, and
- decides when to obtain and save trace data.

Now let’s look at XRay, built on assumptions we could call the antithesis of ours:

- We’re a huge company (in the specific case of XRay’s developers, it’s called Google.)
- We have a million programmers maintaining a billion lines of code who can’t be bothered to integrate a tracing profiler into
  their code. These programmers aren’t our users.
- We have, however, a performance team looking for issues across our millions of servers. _These_ are our users.
- If this performance team looks for issues completely randomly in a small share of our machines, it’s still a ton of
  machines, so they’ll accidentally bump into some unusual slowdowns.
- All of our code is compiled by the same build system. We can change the way code is compiled, as long as the overhead _as_
  _experienced by the teams owning the code_ is not large.

In short, the programmers who wrote the traced code are patients signing a consent form for periodic checkups – and the X-Ray
machine operators are trained doctors on a separate team. [For a big company, these are very sensible\
assumptions](https://yosefk.com/blog/advantages-of-incompetent-management.html). With this in mind, let’s look at how XRay works:

- **XRay instrumentation inserts NOP instruction sequences upon function entry/exit**. The XRay runtime can then
  change these instructions to make the functions jump to function entry/exit handlers _while the program is running_, and
  this patching is done without pausing execution - quite the engineering feat. This makes the overhead very low _when tracing_
  _is disabled,_ which is what code owners care about. It also lets the performance team pick subsets of code where tracing is
  enabled at runtime.
- **XRay culls the overhead of tracing automatically**, so that code owners needn’t bother. In fact, if you run
  XRay in a small test, you might get an almost empty trace and assume it didn’t work, unless you lower the following thresholds:

  - `-fxray-instruction-threshold=<N>` (default: 200) - the compiler only instruments functions with at least
    this many instructions, or those with loops which it thinks are likely enough to take a long time to stray from this rule.
  - `XRAY_BASIC_OPTIONS=func_duration_threshold_us=<N>` (default: 5 microseconds; undocumented at the time of
    writing) - a function call that took less than the value set by this env var is omitted from the trace [17](#fn17).
- **XRay makes an effort to compress trace events** \- instead of logging 64b function pointers, the compiler
  produces 28b integer IDs for the functions, and the runtime stores 32b TSC deltas instead of 64b TSC values. (If the delta
  doesn’t fit into 32 bits, the runtime logs a special record with the full 64b TSC value.)

**Thus an XRay trace entry is half the size of funtrace’s**(and it’s furthermore very easy to keep short calls
out of the trace) **, but XRay’s runtime overhead per instrumented function call is 6 times as large**(as measured
in a microbenchmark, FWIW.)

This is _the sensible tradeoff_ for a big company with a big code base, a big performance team outside the teams
owning the code, and a big tooling team implementing the tracing profiler:

- **It’s extremely important to minimize overhead with tracing disabled**, or people will push back against
  compiling with instrumentation.
- **Always-on function tracing is prohibitively expensive**. With a giant fleet of machines, even a 3% overhead
  costs a fortune.
- **…But the overhead of _occasionally_ enabled tracing is no big deal**. A rare 20-40% slowdown (the
  number reported in [the XRay\
  whitepaper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45287.pdf)) experienced by some service when the performance team is tracing it won’t register as a problem.
- **Traced data size must be minimal**. Since nobody tells us when something interesting happens, we must collect
  data covering A LOT of time to find interesting things ourselves – so we better filter the data well.
- **The complexity of the tool is not an issue** – the cost of developing and maintaining it is dwarfed by the
  cost savings it can provide across a giant server fleet.

Thus it is the right thing, for a big company, to invest effort into thread-safe runtime code patching, a system for creating
and decoding small function IDs, runtime mechanisms for filtering the trace which are slow and make traces harder to understand
\- but the performance team will manage to understand them, and this is how you get data covering a sufficiently large time range
to this team.

The adverse effects of this approach manifest in “smaller” contexts:

- **You can’t afford a high runtime overhead in many cases,** eg a realtime application, or even just a GUI on an
  end-user device. If you have a huge server fleet, slowing down some services considerably for a few seconds is fine, in part
  because in any case there are network lags all the time, and the end user can’t tell the difference. This isn’t as true when
  you’re running on a small “edge device” rather than a big server farm. Overhead is also worse for a _smaller_ server farm
  – the more machines you have, the smaller share of them you can slow down for tracing (and still find stuff), so the
  _relative_ cost shrinks.
- **Runtime code patching is impossible in some small, “embedded” environments,** either because you’re running
  from ROM, or because you’re running from a read-only image in RAM, and have no room for copies of the code pages changed by the
  runtime patching.
- The open source XRay version (not what Google uses internally AFAIK) **still can’t decode symbols from shared**
  **libraries.** Actually, it didn’t even _log_ function calls made by shared libraries for almost a decade, since the
  runtime patching didn’t cover DSOs. But the latest LLVM version _does_ log them thanks to a patch submitted in 2024 by [people using XRay for academic work](https://arxiv.org/pdf/2303.11110). They [plan to make decoding work, too, but the current implementation only supports the\
  “basic logging format”](https://github.com/tudasc/CaPI/issues/2) which is so bloated that it’s unusable in production - the runtime overhead will be
  **15-18** times as large as funtrace’s… Eventually, the OSS XRay might fully support shared libraries, but it will
  have taken at least a decade after the initial version was released.

A big tooling team can deal with the complexity of runtime patching and function ID decoding in the presence of DSOs [18](#fn18); evidently it’s harder for a smaller
project. Using code pointers as IDs spares you these complications, but you pay in trace entry size – which is OK for a smaller
software system, where you can put in logic for tracing just what you want.

**Bigness begets bigness and works well with bigness, and vice versa**. Neither big nor small are “better;” both
are fine as long as you “go big” or small consistently, and when the environment calls for it.

## Synthesis: funtrace with XRay characteristics

You can use funtrace with XRay instrumentation – a straightforward kind of synthesis. This uses almost the same assembly
call/return handlers that we have for gcc with the -pg… instrumentation flags – these get called instead of XRay’s
callbacks.

Why would you use `-fxray-instrument` instead of `-finstrument-functions-after-inlining`, the other way
to use funtrace under clang?

- You might like to be able to automatically exclude short functions by tuning the threshold set by
  -fxray-instruction-threshold=N.
- You can patch the code at runtime to enable tracing and disable it again, lowering the overhead further relatively to
  funtrace’s way to disable tracing (which still has your code jumping to its handlers, which do less than they do when tracing –
  but a bit more than XRay-generated code not patched to jump to tracing callbacks.)

Note that you need a recent LLVM to be able to trace inside DSOs with XRay instrumentation – specifically, a version having
the `-fxray-shared` flag.

Note as well that **support for exceptions under -fxray-instrument has some limitations (same as with gcc under**
**-pg)**, though it’s pretty good and a big step up from XRay’s not supporting exceptions at all (the Google style guide
bans C++ exceptions, and I must say that I fully share their distaste for the feature – but many programs use exceptions, so
funtrace makes an effort to support them, as we’ll see in the followup.)

Now, what would be a deeper form of synthesis than just combining XRay instrumentation with funtrace runtime? What does a
**synthesis of assumptions** look like?

Let’s say we assume the developer is “the user” of tracing and “owns” it, rather than relying on a performance team. We can
still ask ourselves, when is the developer the closest to the position of such a performance team? The answer is, **when**
**adding tracing to their program for the first time!**

I mean, if you started out with tracing from day one, then you’re never in that position. But if you already have a biggish
system and you’re adding tracing to it, then it’s very tedious to manually exclude lots of small functions from tracing.
**How can we make this easier?**

- **Filtering by function size**: funtrace adds a compile time flag, `-funtrace-instr-thresh=N`, which
  works a lot like -fxray-instruction-threshold=N; it excludes short functions from tracing unless they have loops (though you can
  pass `-funtrace-ignore-loops` to have them excluded anyway)
- **Filtering using a list of mangled function names**: let’s say you want to disable the tracing of lots of
  functions reported by funcount as “frequent callees”, and check what this does to tracing overhead, and to the traces you get.
  Going thru 100 source files to add NOFUNTRACE to each function gets old quickly – especially if you want to try many different
  experiments, excluding different subsets of functions every time. Instead, you can use `-funtrace-no-trace=file` to
  exclude the functions listed in that file – way quicker than editing each function.

Sounds great, but you might be wondering, _how does funtrace “add a compile time flag”_, if it uses stock gcc or clang
with no changes or compiler passes?.. If you had a feeling of something fishy coming up, you were very right. Funtrace adds
these trace filtering flags by **post-processing the assembly code generated by the compiler.** Some implications
of this:

- **Assembly post-processing removes most, but not all of the instrumentation overhead.** We’re removing
  instructions calling the function entry/exit hooks, but the code will still have been variously “scarred” by having those calls
  put in by the compiler in the first place. It shouldn’t cost more than 1ns per function, but it can add up.
- **Assembly post-processing, while tested on large programs, is less solid than the rest of funtrace.** It’s
  easy to make a compiler generating assembly code breaking this filtering – both the loop detection (which simply looks for
  branches to labels defined before the branch within a function) and the removal of calls to the hooks. It’s simple text
  processing making assumptions on how the text of the .s files looks like. It “works”, but these assumptions aren’t backed by any
  specification.

So one might decide that this assembly post-processing is more suitable for initial experimentation than a long-term
production deployment. It’s there; it’s your call in what scope to use it, and people will have different preferences for good
reasons. I personally don’t mind the risk of incorrect code generation that much, because I’m good at debugging such things, and
I count on tests to uncover it quickly. But this is the opposite of the right approach for many teams, so I’ve given the exact
opposite advice on some occasions.

Whether you want it for production or not, assembly post-processing should make first-time experiments with funtrace easier,
by providing features for “an encounter with a system for which tracing is alien” – the situation XRay is designed around.

## Hardware-assisted tracing

Most CPUs have some hardware tracing facilities, but the most basic & common of these are designed for people debugging
the hardware itself or very low-level software like kernels and boot loaders using something like a JTAG probe. For example,
when a branch is taken, the instruction address or a delta might be sent to a probe like that over a dedicated channel, or it
might be saved into a tiny circular SRAM buffer inside the chip that you can then read with the probe. This doesn’t help most
people debugging large systems though.

An exception to this is the Intel Performance Trace, which lets you trace native code with zero instrumentation. The awesome
magic-trace is built on top of it. You can try it with `magic-trace run ls` (for example); it works out of the box,
no recompilation required, and the overhead is low (they say 2-10%.)

**I’m unironically in shock that people deploying on x86, certainly to environments they control, don’t all insist on**
**using Intel hardware to always run the code under magic-trace in production.** (You can trigger tracing programmatically,
and with these traces, you’ll be able to debug any latency issue you could have in production.) Did Google develop the
high-overhead XRay when Intel Performance Trace already existed?. _._ (Not sure about the exact timeline; perhaps the two
matured at about the same time?) How can it be that a decade after Intel Performance Trace was made, AMD still hasn’t caught up,
and other platforms also lack equivalent features?

Seriously, it’s as if you had hardware floating point units for a decade and only a select few would be using them, or were
even aware of them, with a few more stuck on software emulation, and most just [using scaled integers](https://yosefk.com/blog/10x-more-selective.html), as in representing 1.05 with 105. How do we
explain this?

- **Programmers aren’t demanding tracing profilers**; they use sampling profilers, because it’s the tradition, it
  helps with the average case (and unlike with the big O notation, nobody drills it into programmers to look for the worst case
  when profiling, or how to do it), and it requires _zero_ work, unlike a tracing profiler where triggering tracing
  requires _slightly less than zero work._
- **The hardware solution is fairly complicated.** Many teams will not do it given weak demand; a lightweight
  feature with weak or uncertain demand might get greenlit, but a full-blown control flow trace like Intel’s isn’t a lightweight
  feature.
- **Machine instruction-level control flow tracing is useless for interpreters and hard-to-use for JITters,** and
  most people with control of their environment are server people running a lot of Java, JavaScript, Python, PHP etc. For an
  interpreter all you’d see is the interpreter’s loop spinning, with no clue what code it’s interpreting. For a JITter, _every_
  _JIT runtime_ would need to dump metadata telling a tool like magic-trace what source code each machine code snippet was
  generated from – you get this from compiler-generated symbol tables for statically compiled native code.

I would be pleasantly surprised if this writeup would cause programmers’ demand for tracing profilers to outright explode,
but I’m not counting on it. However, I have a suggestion for tracing support in the CPU hardware which is **simple enough**
**to risk implementing despite weak demand – and simple enough for interpreters and JITters to use.** So you can
realistically hope to put this thing into your CPU, have interpreters and JITters use it, and then programmers will love your
hardware for its tracing features.

Here’s how it could work:

- **You add an instruction, say TRACE**, which traces a 64b register value from a general-purpose register (or
  the instruction pointer as a special case). It also traces a few bits of static metadata from its encoding (so we can tag events
  as “function call,” “return,” “context switch” etc.
- **TRACE sends the data above via a special port to a trace writer module.** TRACE is _not_ another store
  instruction; it doesn’t go thru caches. Instead, data is sent to a module where it could be timestamped, compressed, and written
  from the module’s SRAM to a cyclic buffer in DRAM.

![image5.png](https://yosefk.com/img/funtrace/image5.png)

Thus we still rely on software to issue tracing instructions, but it’s just one instruction per event, we get timestamping,
compression and cyclic buffer management for free, and the only cost is another instruction and the DRAM bandwidth spent on
writing to the cyclic buffer (and we can write at a low priority - we don’t mind buffering these writes as long as we haven’t
run out of SRAM in the module; we would also lose very little bandwidth to precharging/activating DRAM rows, since our writes
are the opposite of random access.) We can also turn off the writing, and then the overhead shrinks to just fetching &
decoding a NOP - “like XRay with tracing disabled.”

Two refinements of this idea in two opposite directions:

- **If you’re “a perfectionist CPU maker”**, you can further lower the overhead to near-zero for statically
  generated code. For example, x86 already has the ENDBR64 instruction, which every recently compiled function starts with, in
  pursuit of the venerable goal of “ [control flow integrity](https://en.wikipedia.org/wiki/Control-flow_integrity)” [19](#fn19). This instruction could be an implicit TRACE
  PC, and we could have an ENDBR64\_UNTRACED for excluding functions from tracing to save bandwidth. The same thing could be done
  with RET.
- **If you’re “a pragmatic chip maker”** and the CPU IP vendor won’t add a TRACE instruction, you can instead
  have software store to a memory-mapped device your chip makes available at some address. You will spend more instructions to
  send the data, and you will interfere with the load-store unit, but you will still save the overhead of timestamping and cyclic
  buffer management, you won’t pollute caches with trace data, you will save space & bandwidth with compression, and you will
  get a single buffer for all the trace data instead of many thread-local buffers which probably take more memory than you need
  (for this, you will want to encode the thread ID or the CPU ID when storing trace data, either in the data itself or the address
  bits - eg each CPU can store to a different memory-mapped address, and the OS can trace context switches so you know which
  thread is currently running on each CPU.)

This scheme is very easy on the hardware; a “tracing device” with a RAM and a DMA is invisibly small in today’s hardware
designs, and this doesn’t interfere with the CPU logic and doesn’t risk bugs, either those breaking the CPU or those breaking
the trace.

As an added bonus, **an interpreter can use TRACE and pass it some data which isn’t a machine instruction address but**
**rather the ID of a function in the interpreted language**. And a JITter can emit TRACE instructions into its code. (I
feel that this would be bigger news for interpreters than JITters, but I think most JITters would benefit from it relatively to
Intel Performance Trace-like hardware-assisted tracing – would be happy to hear the thoughts of people understanding
JITters.)

I think it would be great to start seeing TRACE instructions in CPUs!

## Conclusion and future work

We (it’s always “we” in papers, isn’t it?) have presented a comprehensive solution for C++ function tracing, ready for
production use on x86/Linux and easy to port to many other platforms. We have also used the opportunity to discuss how to use a
function tracer in your workflow, how to implement your own function tracer for native code, and which existing tools can help
with the heavy lifting. Finally, we’ve seen how hardware could help making tracing more efficient and usable for both statically
and dynamically compiled languages, in a relatively cheap & simple way.

Here are some things “we” could add to funtrace (more likely _I_ than _we –_ though I’d be happy to work with
you on this!):

- **Porting to other native languages.** I’d expect the trace file format and the decoder to need no changes in
  most cases; the runtime you might well want to rewrite (eg a Rust runtime is easier to distribute with cargo than a C++ runtime,
  right?.. Though the C++ one would be functionally adequate in this case?..) For compile-time support, you could use existing
  LLVM features where relevant, or you could write an LLVM pass - or write code changing LLVM IR files or even compiler-generated
  assembly (the `-funtrace` compile time flags do this, and I've gone way further with that… in this context. See the
  hardcore followup!)
- **Support for performance counters**. We could log the output of RDPMC instead of, or in addition to RDTSC.
  This might be useful (you could learn things like which functions missed the cache a lot), though RDPMC is a PITA, as we’ll see
  in that followup of ours.
- **Support for custom events**. You might want to log things like “the resolution of the images we were
  processing was 1920x1080”. A “ [delayed printf](https://yosefk.com/blog/delayed-printf-for-real-time-logging.html)”
  approach could be a great fit for this, seeing how we need the executable files to decode the code pointers anyway; so we could
  easily extract format strings from the executables given the logged pointers while we’re at it, and do the formatting at
  decoding time.
- **Support for goroutines / async / other forms of “non-OS threads.”** Note that the relatively easy part is to
  decode each such “green thread” into its own Perfetto JSON thread. The harder part is to figure out how not to drown in all
  those threads when viewing them; these runtimes brag about millions of threads – that’s a lot of vertical scrolling. Ofc you can
  limit tracing to a subset of these; you could also show a lane per CPU instead of a lane per thread, but then the call stacks
  changing from one thread to another get very noisy (ask me how I know) and you might want to adapt the viewer to deal with this
  somehow.
- **Support for existing tracing frameworks** – for example, integrating with the [tracy](https://github.com/wolfpld/tracy) profiler. Note that any tracing system using the same timestamps as funtrace
  can theoretically coexist with it without any need to share buffers or data formats; what you’d want is to then view all this
  traced data in a single viewer.
- **Clever trace filtering** a-la XRay – eg detect at runtime that a function was called 1000 times and took less
  than a microsecond every time, and modify its code to no longer call the entry/return hooks. This is risky (what if this
  function locks something and will wait for a long time in some future call?), but only somewhat (we’ll still trace its caller
  and get some idea of where we waited), and it removes the overhead of RDTSC, which is big on x86.

That’s it – I hope you liked it! And if you’re really into this kind of stuff (evidently, I’m _really_ into this
stuff!), give [funtrace](https://github.com/yosefk/funtrace) a try, and stay tuned for the Hardcore Followup!

_Thanks to Dan Luu for reviewing a draft of this post._

## See also

[Sampling vs tracing](https://danluu.com/perf-tracing/) was the greatest inspiration for this work. It discusses
tracing in a distributed computing environment, and mentions some techniques different from what we’ve seen above – such as
having your function entry/exit handlers write to a “current code pointer” global variable, read in a busy loop by a CPU core
dedicated for tracing (YMMV, but this could lower the impact of tracing on latency at the cost of “burning” a CPU core, thus
trading some of the machine’s throughput for the latency gain.)

* * *

01. Note that, for every new trace.json, you want to run `vizviwer trace.json`, _even if you fully_
    _expect to have to then open trace.json again from the Web UI,_ to work around the “RPC” thing _._ That’s because
    vizviewer reads the source code from the JSON at the server side. So if you just load the JSON from the web UI in a tab
    previously opened by vizviewer, you won’t get the source code – you’ll get “No source code found” messages. [↩︎](#fnref1)

02. Traditionally, flamegraphs are displayed as stalagmites, growing upwards rather than downwards, but, like,
    whatever. [↩︎](#fnref2)

03. I list the issues I ran into _to encourage_ readers to use viztracer. If I just wrote how great it was,
    and then they tried it and ran into issues I didn’t mention, it would _discourage_ them _._ In fact, I start
    twitching when I look for something and only find posts showing the happy path with nothing going wrong, and telling how awesome
    the thing is – it is a strong predictor of running into issues with no help in sight. But if I see the rare writeup listing a
    few rakes the author stepped on – and their exact whereabouts to keep me from stepping on them myself – _that_ makes me
    very optimistic and eager to give the thing a try! [↩︎](#fnref3)

04. Strictly speaking, it’s enough to instrument function entry points; the callback can then change the return
    address to jump into an epilogue which traces the return, and then jumps to the original return address. For funtrace on
    x86/Linux, “this is neither good nor necessary,” to quote Capablanca, but it makes sense in some situations. [↩︎](#fnref4)

05. Zero overhead abstractions strike again! The billion-inline-wrappers style famously interferes with debugging,
    and profiling is a special case of that. But who cares? That’s overhead for developers, not machines. We’re here to serve
    machines! I mean, look how much we did for machines in quality and quantity in the last 80 years, compare that to the modest
    improvements they created for us, and weep. [↩︎](#fnref5)

06. Pausing can “spoil” some snapshots, but very rarely. Specifically, if each of 2 threads notices an unusual
    event, and they take snapshots concurrently, the 1st thread will get a fine snapshot, but the 2nd might see a “hole” in its
    snapshot, because the 1st one paused tracing while snapshotting. But 2 _overlapping_, _unusual_ events are, well,
    _very_ unusual, and the 1st is still fully captured. [↩︎](#fnref6)

07. Actually, it's [worse](https://github.com/yosefk/funtrace/blob/master/funtrace_flags.h) or it would
    have been called _flag_ rather than _flags_, but that stuff definitely belongs in the hardcore followup. [↩︎](#fnref7)

08. The races in our tracing code can probably be fixed without having a memory barrier upon every event (like XRay
    does, for example.) I _think_ we only need a barrier when we notice that tracing was paused, and we want whoever paused
    it to read up-to-date data. I felt that we can leave the races as is, and spare a bunch of extra complexity, on a rather naive
    theory of the memory system. Specifically, I believe that since we have one writer for the trace data (the thing written without
    locking – wraparound\_mask is _read_ without locking, but is written with a lock held), the worst that can happen is that
    a few recent writes will get stuck in the write buffer of the core running the traced thread. However, the bulk of the data will
    be in dirty cache lines, and the reader will get up-to-date data from these cache lines, because that’s how cache coherence
    works. I’m very willing to hear why things are worse than I think, especially with the funky binary search that the reader is
    running on the trace buffers – which will be discussed in The Hardcore Followup, but until then, feedback on the code is most
    welcome, too. I’m OK at this stuff but far from great, and much better at avoiding it than doing clever things with it. [↩︎](#fnref8)

09. Actually, \_\_cyg\_profile\_func\_enter gives you the function pointer for the caller, while \_\_builtin\_return\_address
    would give you an address of some instruction inside the caller. But both are fine for symbol table lookup – symbol table maps a
    function name to an address range containing both pointers; In fact, I don’t understand why \_\_cyg\_profile\_func\_enter bothers to
    give you the function pointer – \_\_builtin\_return\_address would give you an equally useful address, and would save instructions
    at the instrumented functions. [↩︎](#fnref9)

10. It should be a truth universally acknowledged that objects should have names – very useful for debugging.
    Awesome that threads have a whopping 15 characters for the name – can we also get this for locks, for example?.. This is a pet
    peeve of mine; I don’t understand why naming objects isn’t more common – the overhead is most often dwarfed by the utility. The
    overhead would be smaller still if this was something people cared about – eg you could variously assign 64b IDs mapped to
    names, instead of copying strings, if there was a standard way to manage these IDs. [↩︎](#fnref10)

11. There’s often a feedback loop where there are some basic reasons for a language not being that great for some
    area, and then it becomes a feedback cycle of, “nobody uses the language for this, so language developers don’t care to further
    hurt this use case, so they do, and so people have even less reason to use the language for that use case.” I’m not sure this is
    the case with Rust & numerics; I haven’t thought about it deeply. It seems that numerics isn’t a focus area for the
    language, but there’s also a lot of serious “numerics-adjacent work” happening in Rust. So I guess we’ll see where this goes. [↩︎](#fnref11)

12. In fact, to take our example, Rust’s Ratio<BigInt> is pretty slow – enough for me to have noticed &
    tried ratios with 64b numerators and denominators, which didn’t work for what I was doing, so back to BigInt I was. I’m pretty
    sure that MPFR would have been faster, but it’d also be a more painful dependency to deal with. So, Rust managing prebuilt
    packages a-la pip would have been great! [↩︎](#fnref12)

13. A nice side effect of our thread reading from the kernel ftrace buffer into our own userspace buffer is that
    you can then get the events from a core dump. You could of course save ftrace together with the core using some funky
    /proc/sys/kernel/core\_pattern instead – but who’s ever gonna do it, and then propagate the trace together with the core through
    whatever scary path it goes through until reaching the developer? [↩︎](#fnref13)

14. Actually, if we run the program _with funtrace instrumentation_, then a sampling profiler _could_ tell us where our \_\_fentry\_\_ callback or whatever gets called from, since a good sampling profiler samples _call_
    _stacks_, and so it could show us the callstacks with \_\_fentry\_\_ sorted by call count. But this suffers from artifacts due to
    low sampling rate, and then it’s just an unpleasant, heavyweight and flaky workflow IME – I tried running
    `perf record -g` and then running `hotspot` and going to the _Bottom Up_ or _Caller/Callee_
    tabs, and it sorta works, but I can’t recommend it. The nice thing about funcount is that it does this one job it’s named after,
    it provides a correct and straightforward answer to “the call count question,” and the reports are small & very easy to
    understand, combine and manipulate in whichever way you want. [↩︎](#fnref14)

15. Actually our approach misses function calls in constructors – eg if you interpose dlopen, libc’s dlopen will
    map executable segments, call the constructors, and only then return to you, giving you your first chance to run dl\_iterate\_phdr
    with the new segments mapped. In most programs, however, _for the purposes of lowering tracing overhead_, missing
    constructor calls has got to be a non-problem. [↩︎](#fnref15)

16. Decode the reports before combining them - function pointers are _not_ the same across runs thanks to
    ASLR and what-not [↩︎](#fnref16)

17. Actually, the so-called “basic logging format” (too slow for serious use, I’d say) can be said to be filtering
    functions by func\_duration\_threshold\_us; it keeps a callstack at runtime, and so it knows the time a function call took, so it
    can compare to the threshold. “FDR (flight data recorder) logging” also uses this threshold, but I don’t think it’s as
    straightforward as “filtering by function duration” and I didn’t dig enough to tell you exactly what it does. [↩︎](#fnref17)

18. It could be that Google’s internal version of XRay doesn’t support shared libraries, either, because Google
    links everything statically on the server – not sure they do, but if they do, I am here to say “you see, just like I told you –
    shared libraries are no good!” Anyway, my point is only that XRay’s effort to assign function IDs is a “big company thing” for
    various reasons I mentioned, and that it creates difficulties with shared library support that within a big company would be
    trivial, but evidently aren’t trivial outside it. [↩︎](#fnref18)

19. We’ve all heard something about “every ending is a new beginning,” and since we all know that A being B is a
    transitive relationship, we could all extrapolate from this that every new beginning is an ending. But I think few of us were
    emotionally prepared by this logic to seeing every compiler-generated function _beginning_ with an instruction called
    _END_\_WHATEVER. I’m just saying, at some point, you have got to start wondering, “am I maybe doing this whole thing
    wrong, and should I do some bigger changes than the crazy incremental ones I’m finding myself doing?” And if it requires a
    Worldwide Software Czar to shake things up in the right direction, I just want you all to know that I could be available, if the
    price is right. [↩︎](#fnref19)
