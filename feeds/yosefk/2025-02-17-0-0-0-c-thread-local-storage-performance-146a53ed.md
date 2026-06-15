---
title: '0+0 > 0: C++ thread-local storage performance'
url: https://yosefk.com/blog/cxx-thread-local-storage-performance.html
published: "2025-02-17T00:00:00Z"
feed: yosefk
---

# 0+0 > 0: C++ thread-local storage performance

We'll discuss how to make sure that your access to TLS (thread-local storage) is fast. If you’re interested strictly in TLS
performance guidelines and don't care about the details, [skip right to the end](#summary-of-performance-guidelines)
— but be aware that you’ll be missing out on assembly listings of profound emotional depth, which can shake even a cynical,
battle-hardened programmer. If you don’t want to miss out on that — and who would?! — read on, and you shall learn the
computer-scientific insight behind the intriguing inequality 0+0 > 0.

I’ve recently [published](https://yosefk.com/blog/profiling-in-production-with-function-call-traces.html) a new
C++ profiler, [funtrace](https://github.com/yosefk/funtrace), which traces function calls & returns as well as
thread state changes, showing an execution timeline like this (the screenshot is from [Krita](https://krita.org/), a
“real-world,” complicated drawing program):

![image8.png](https://yosefk.com/img/funtrace/image8.png)

One thing a software-based tracing profiler needs is a per-thread buffer for traced data. Actually it would waste less memory
for all threads to share the same buffer, and this is how things “should” work in a system with some fairly minimal [hardware support\
for tracing, which I suggested in the funtrace writeup](https://yosefk.com/blog/profiling-in-production-with-function-call-traces.html#hardware-assisted-tracing), and which would look roughly like this:

![image5.png](https://yosefk.com/img/funtrace/image5.png)

But absent such trace data writing hardware, the data must be written using store instructions through the caches [1](#fn1). So many CPUs sharing a trace buffer results in
them constantly yanking lines from each other’s caches in order to append to the buffer, with a spectacular slowdown. And then
you'd need to synchronize updates to the current write position — still more slowdown. A shared buffer can be fine for [user-initiated printing](https://yosefk.com/blog/delayed-printf-for-real-time-logging.html), but it’s too slow for
tracing every call and return.

So per-thread buffers it is — bringing us to C++’s `thread_local` keyword, which gives each thread its own copy of
a variable in the global scope — perfect for our trace buffers, it would seem. But it turns out that **we need to be**
**careful with exactly how we use `thread_local` to keep our variable access time from exploding**, as explained
in the rest of this document.

The C toolchain — not the C++ compiler front-end, but assemblers, linkers and such — is generally quite ossified, with [decades-old\
linker bugs enshrined as a standard](https://stackoverflow.com/questions/76925604/why-is-the-constructor-of-a-global-variable-not-called-in-a-library) [2](#fn2). TLS
is an interesting case when this toolchain was actually given quite the facelift to support a new feature — with the result of
simple, convenient syntax potentially hiding fairly high overhead (contrary to the more typical case of inconvenient syntax, no
new work in the toolchain, and resource use being fairly explicit.)

At first glance, TLS looks wonderfully efficient, with a whole machine register dedicated to making access to these exotic
variables fast, and a whole scheme set up in the linker to use this register. Let’s take this code accessing a
`thread_local` object named `tls_obj`:

```
int get_first() {
  return tls_obj.first_member;
}
```

This compiles to the following assembly code:

```
  movl  %fs:tls_obj@tpoff, %eax

```

This loads data from the address of `tls_obj` into the `%eax` register where the return value should
go. The address of tls\_obj is computed by adding the value of the register `%fs` and the constant offset
`tls_obj@tpoff`. Here, `%fs` is the TLS base address register on x86; other machines similarly reserve a
register for this. `tls_obj@tpoff` is an offset from the base address of the TLS area allocated per thread, and it’s
assigned by the linker such that room is reserved within the TLS area for every `thread_local` object in the linked
binary. Is this awesome or what?!

## Constructors

If instead we access a `thread_local` object with a constructor — let's call it `tls_with_ctor` — we
get assembly code like this (and this is with `-O3` – you really don’t want to see the unoptimized version of
this):

```
  cmpb  $0, %fs:__tls_guard@tpoff
  je    .slow_path
  movl  %fs:tls_with_ctor@tpoff, %eax
  ret
.slow_path:
  // inlined call to __tls_init, which constructs
  // all the TLS variables in this translation unit…
  pushq %rbx
  movq  %fs:0, %rbx
  movb  $1, %fs:__tls_guard@tpoff
  leaq  tls_with_ctor@tpoff(%rbx), %rdi
  call  Class::Class()
  leaq  tls_with_ctor2@tpoff(%rbx), %rdi
  call  Class2::Class2()
  // …followed by our function’s code
  movl    %fs:tls_with_ctor@tpoff, %eax
  popq  %rbx
  ret

```

Our simple access to a register plus offset has evolved to first check a thread-local “guard variable”, and if it’s not yet
set to 1, it now calls the constructors for all of the thread-local objects in the translation unit. ( `__tls_guard`
is an implicitly generated `static`, per-translation-unit boolean.)

While funtrace’s call/return hooks, which get their trace buffer pointer from TLS, are called all the time, access to
`thread_local` s should be more rare in “normal” code — so not sure it’s fair to brand this `__tls_guard`
approach as having “unacceptable overhead.” Of course, the inlining only happens if your thread\_local is defined in the same
translation unit where you access it; **accessing an `extern thread_local` with a constructor involves a**
**function call**, with the function testing the guard variable of the translation unit where the thread\_local is defined.
But with inlining, the fast path is quite fast on a good processor (I come from an embedded background where you usually have
_cheap_ CPUs rather than _good_, so an extra load and a branch depending on the loaded value shock me more than
they should; a superscalar out-of-order branch-predicting speculatively-executing CPU will handle this just fine.)

What I don’t understand is why. Like, _why._ Generating this code must have taken a bunch of compiler work; it didn’t
“just happen for free.” Furthermore, the `varname@tpoff` thing must have involved some linker work; it’s not like
keeping the linker unchanged was a constraint. Why not arrange for the `__tls_init` function of every translation
unit (the one that got inlined into the slow path above) to be called before a thread’s entry point is called? Because it would
require a little bit of libc or libpthread work?..

I mean, this was done for _global constructors_. You don’t check whether you called the global constructors of a
translation unit before accessing a global with a constructor (and sure, _that_ would have been even slower than the TLS
init code checking `__tls_guard`, because it would need to have been a _thread-safe_ guard variable access;
though even _this_ was implemented for calling the constructors of _static variables declared inside functions,_ see also `-fno-threadsafe-statics`.) It’s not really harder to do this for TLS constructors than for global
constructors, except that we need `pthread_create` to call this code, which, why not?..

Is this a deliberate performance tradeoff, benefitting code with lots of thread\_locals and starting threads constantly, with
each thread using few of the thread\_locals, and some thread\_locals having slow constructors [3](#fn3)? But such code isn't great to begin with?.. Anyway, I don’t really get why the
ugly thing above is generated from `thread_local` s’ constructors. The way I handled it in my case is,
**funtrace sidesteps the TLS constructor problem by [interposing](https://docs.oracle.com/cd/E19683-01/816-1386/chapter3-26/index.html) `pthread_create`**, and initializing its `thread_local` s in its pthread\_create wrapper.

## Shared libraries

And now let’s see what happens when we put our thread-local variable, the one without a constructor, into a shared library
(compiling with `-fPIC` and linking with `-shared`):

```
push %rbp
mov  %rsp,%rbp
data16 lea tls_obj(%rip),%rdi
data16 data16 callq __tls_get_addr@plt
mov  (%rax),%eax
pop  %rbp
retq

```

All this colorful code is generated instead of what used to be a single **movl**
**%fs: _tls\_obj_@tpoff, %eax**. More code was generated than before,
forcing us to **spill and restore registers**. But the worst part is that our
TLS access now requires **a function call** — we need
`__tls_get_addr` to find the TLS area of the currently running shared library.

**Why don’t we just use the same code as before — the `movl` instruction — with the dynamic linker**
**substituting the right value for `tls_obj@tpoff`?** This is an honest question; I don’t understand why this
isn’t a job for the dynamic linker like any other kind of dynamic relocation. Is this to save work in libc again?.. Like, for
`tls_obj@tpoff` to be an offset _from the same base address_ no matter which shared library
`tls_obj` was linked into, you would need the TLS areas of all the shared libraries to be allocated contiguously:

- main executable at offset 0
- the first loaded .so at the offset `sizeof(main TLS)`
- the next one at the offset `sizeof(main TLS) + sizeof(first.so TLS)`
- …

But for this, libc would need to do this contiguous allocation, and of course you can’t move the TLS data once you’ve
allocated it, since someone might be keeping pointers into it [4](#fn4). So you need to carve out a chunk of the memory space — no biggie with a 64-bit or even
“just” a 48-bit address space, right?.. — and you need to put the executable’s TLS at some magic address with `mmap`
and then you keep `mmap` ing the TLS areas of newly loaded .so’s one next to another.

But this now becomes a part of the ABI (“these addresses are reserved for TLS”), and I guess nobody wanted to soil the ABI
this way “just” to make TLS fast for shared libraries?.. In any case, looks like TLS areas are allocated non-contiguously and so
you need a different base address every time and you can’t use an offset… but _still_, couldn’t the dynamic linker bake
this address into the code, instead of calling a function to get it?.. Feels to me that this was doable but deemed not worth the
trouble, more than it being impossible, though maybe I’m missing something.

A curious bit is those `data16` [5](#fn5)
in the code:

```
data16 lea tls_obj(%rip),%rdi
data16 data16 callq __tls_get_addr@plt

```

What is this for?.. Actually, the `data16` prefix does nothing in this context except padding the instructions to
take more space, making things slightly slower still, though it’s peanuts compared to the function call. Why does the compiler
put this padding in? Because if you compile with `-fPIC` but then link the code into an executable, without the
`-shared`, the function call gets replaced with faster code:

```
push %rbp
mov  %rsp,%rbp
mov  %fs:0x0,%rax
lea  -0x4(%rax),%rax
mov  (%rax),%eax
pop  %rbp
retq

```

The generated code is still scarred with the **register spilling** and
what-not, and we don’t get our simple **movl %fs: _tls\_obj_@tpoff, %eax** back, but still, we have to be very thankful for the compiler & linker
work here, done for the benefit of the _many_ people whose build system compiles everything with `-fPIC`,
including code that is then linked without `-shared` (because who knows if the .o will be linked into a shared
library or an executable? It’s not like the build system knows _the entire graph of build dependencies —_ wait, it
actually _does —_ but still, it obviously shouldn’t be _bothered_ to find out if -fPIC is needed — this type of
mundane concern would just distract it from its noble goal of Scheduling a Graph of Completely Generic Tasks. Seriously, no C++
build system out there stoops to this - not one, and goodness knows there are A LOT of them.)

In any case, the `data16` s are generated by the compiler to make the red instructions take enough space for the
green instructions to fit into, in case we link without `-shared` after all.

## Constructors in shared libraries

And now let’s see what happens if we put (1) a thread\_local object with (2) a constructor into a shared library, for a fine
example of how 2 of C++’s famously “zero-overhead” features compose. We’ve all heard how “the whole is greater than the sum of
its parts,” occasionally expressed by the peppier HRy people as “1 + 1 = 3.” I suggest a similarly inspiring expression “0 + 0
\> 0”, which quite often applies to “zero overhead”:

```
sub  $0x8,%rsp
callq TLS init function for tls_with_ctor@plt
data16 lea tls_with_ctor(%rip),%rdi
data16 data16 callq __tls_get_addr@plt
mov  (%rax),%eax
add  $0x8,%rsp
retq

```

So, now we have 2 function calls — one for calling the constructor in case it wasn’t called yet, and another to get the
address of the `thread_local` variable from its ID. Makes sense, except that I recall that under `-O3`,
this “TLS init function” business was inlined, and now it no longer is? Say, I wonder what code got generated for this “TLS init
function”?..

```
  subq  $8, %rsp
  leaq  __tls_guard@tlsld(%rip), %rdi
  call  __tls_get_addr@PLT
  cmpb  $0, __tls_guard@dtpoff(%rax)
  je    .slow_path
  addq  $8, %rsp
  ret
.slow_path:
  movb  $1, __tls_guard@dtpoff(%rax)
  data16  leaq  tls_with_ctor@tlsgd(%rip), %rdi
  data16 data16 call  __tls_get_addr@PLT
  movq  %rax, %rdi
  call  Class::Class()@PLT
  data16  leaq  tls_with_ctor2@tlsgd(%rip), %rdi
  data16 data16 call  __tls_get_addr@PLT
  addq  $8, %rsp
  movq  %rax, %rdi
  jmp   Class2::Class2()@PLT

```

Oh boy. So not only doesn’t this thing get inlined, but it calls `__tls_get_addr` _again, **even on the**_
_**fast path**._ And then you have the slow path, which calls \_\_tls\_get\_addr _again and again_…not that we care,
it runs just once, but it kinda shows that this \_\_tls\_get\_addr business doesn’t optimize very well. I mean, it’s not just the
slow path of the init code — here’s how a function accessing 2 thread\_local objects with constructors looks like:

```
pushq   %rbx
call    TLS init function for tls_with_ctor@PLT
data16 leaq tls_with_ctor@tlsgd(%rip), %rdi
data16 data16 call __tls_get_addr@PLT
movl    (%rax), %ebx
call    TLS init function for tls_with_ctor2@PLT
data16 leaq tls_with_ctor2@tlsgd(%rip), %rdi
data16 data16 call __tls_get_addr@PLT
addl    (%rax), %ebx
movl    %ebx, %eax
popq    %rbx

```

Like… man. This calls \_\_tls\_get\_addr **_4 times_**, twice per
accessed thread\_local (once directly, and once from the “TLS init functions”).

Why do we call _2_“TLS init function for whatever” when _both_ do the same thing — check the guard variable
and run the constructors of _all_ objects in the translation unit (and in this case the two objects are defined in the
same translation unit, the same one where the function is defined)? Is it because in the general case, the two objects come from
2 different translation units?

And what about the \_\_tls\_get\_addr calls to get the addresses of the objects themselves? Why call _that_ twice? Why not
call something just once that gives you the base address of the module’s TLS, and then add offsets to it? Is it because in the
general case, the two objects could come from 2 different shared libraries?

And BTW, with clang 20 (the latest version ATM), it’s seemingly enough for _one_ thread-local object in a translation
unit to have a constructor for the compiler to generate a “TLS init function” for _every_ thread-local object, and call
it when the object is accessed… so, seriously, **don’t** use `thread_local` with constructors, even if
you don’t care about the overhead, as long as there’s even one thread\_local object where you _do_ care about access
time.

On the other hand, clang has an optimization, where access to several thread\_locals _with hidden visibility [6](#fn6)_ is indeed optimized such that
`__tls_get_addr` is only called once (instead of twice the number of accessed thread\_locals), and then we add a
per-variable offset to access each thread\_local. It turns out that a big part of the answer to the question "why call
\_\_tls\_get\_addr per variable?" is that **with the default visibility, variables could be interposed at runtime,**
and so the compiler can't assume that they're defined by the same shared library, even if it's compiling a .cpp file that
defines all of the accessed variables.

Of course, the other part of the answer is that it takes work to implement this optimization; [according to the comment that I learned this\
from](https://lobste.rs/s/b5dnjh/0_0_0_c_thread_local_storage_performance#c_dkzbw3), this optimization is not available on all platforms in clang, and I'm not seeing it in g++ on x86. A smaller problem
is that as you can see in the code below, with the current code generation, there's lots of register spilling and restoring
going on which I can't really explain (even if I look at the slow path which I elided in the assembly listing below, since it's
hairy enough as it is):

```
pushq   %rbp
pushq   %r15
pushq   %r14
pushq   %rbx
pushq   %rax
leaq    __tls_guard@TLSLD(%rip), %rdi
callq   __tls_get_addr@PLT
movq    %rax, %rbx
cmpb    $0, __tls_guard@DTPOFF(%rax)
je      .slow_path
movl    tls_with_ctor@DTPOFF(%rbx), %ebp
addl    tls_with_ctor2@DTPOFF(%rbx), %ebp
movl    %ebp, %eax
addq    $8, %rsp
popq    %rbx
popq    %r14
popq    %r15
popq    %rbp
retq

```

Note that if you compile with `-fPIC` but then link without `-shared`, even the single call to
`__tls_get_addr` gets replaced with the much faster, if quite colorful instruction
`data16 data16 data16 mov %fs:0x0,%rax`. All in all, an impressive effort by clang to optimize TLS access from shared
objects; yet on net balance, I think it's fair to recommend putting data into a smaller number of thread\_locals and avoiding
constructors, rather than counting on visibility to improve the code generation.

So what does that famous `__tls_get_addr` function do? Here’s the fast path:

```
mov  %fs:DTV_OFFSET, %RDX_LP
mov  GL_TLS_GENERATION_OFFSET+_rtld_local(%rip), %RAX_LP
cmp  %RAX_LP, (%rdx)
jne  .slow_path
mov  TI_MODULE_OFFSET(%rdi), %RAX_LP
salq $4, %rax
movq (%rdx,%rax), %rax
cmp  $-1, %RAX_LP
je   .slow_path
add  TI_OFFSET_OFFSET(%rdi), %RAX_LP
ret

```

These 11 instructions on the fast path enable lazy allocation of a shared library’s TLS — every thread only allocates a TLS
for a given shared library upon its first attempt to access one of its thread-local variables. (Each “variable ID” passed to
`__tls_get_addr` is a pointer to a struct with module ID and an offset within that module’s TLS;
`__tls_get_addr` checks whether TLS was allocated for the module, and if it wasn’t, calls
`__tls_get_addr_slow` in order to allocate it.)

Is this lazy allocation the answer to why the whole thing is so slow? Do we _really_ want to only call constructors
for thread-local variables upon first use, and ideally to even allocate memory for them upon first use? Note that we allocate
memory _for all the thread\_locals **in a shared library**_ upon the first use of even one; but we call
constructors _for all the thread\_locals **in a translation unit**_ upon the first use of even one; which is
a bit random for the C++ standard to prescribe, not to mention that it doesn’t really concern itself with dynamic loading? So
it’s more, the standard gave implementations room to do this, rather than prescribed them to do this?.. I don’t know about you,
but I’d prefer a contiguous allocation for all the TLS areas of all the modules in all the threads, and fast access to the
variables over this lazy allocation and initialization; I wonder if this was a deliberate tradeoff or “just how things ended up
being.”

## Summary of performance guidelines

- Access to `thread_local` objects without constructors linked into an executable is _very_ efficient
- Constructors make this slower…
- Especially if you access an `extern thread_local` from another translation unit…
- Separately from constructors, compiling with `-fPIC` also makes TLS access slower…
- …and linking code compiled with `-fPIC` with the `-shared` flag makes it _seriously_ slower,
  worse than either constructors or compiling with `-fPIC`...
- …but constructors together with `-fPIC -shared` _really_ takes the cake and is the slowest by far!
- …and actually, a thread\_local variable x having a constructor might slow down access to a thread\_local variable y in the
  same translation unit
- Prefer putting the data into one thread\_local object rather than several when you can (true for globals, too, BTW.) It can’t
  hurt, and it can probably help a lot, by having fewer calls to `__tls_get_addr` if your code is linked into a shared
  library.
- Define your thread\_locals as having hidden visibility - it won't always help if they're compiled into a shared library, but
  sometimes it'll help a lot, and it can't hurt.

## Future work

It annoys me to no end that the funtrace runtime has to be linked into the executable to avoid the price of
`__tls_get_addr`. (This also means that funtrace must export its runtime functions from the executable, which
precludes shared libraries using the funtrace runtime API (for taking trace snapshots) from linking with
`-Wl,--no-undefined`.)

I just want a tiny thread-local struct. It can’t be that I can’t do that efficiently without modifying the executable, so
that for instance a Python extension module can be traced without recompiling the Python executable. Seriously, there’s a limit
to how idiotic things should be able to get.

I’m sure there’s some dirty trick or other, based on knowing the guts of libc and other such, which, while dirty, is going to
work for a long time, and where you can reasonably safely detect if it stopped working and upgrade it for whatever changes the
guts of libc will have undergone. If you have an idea, please share it! If not, I guess I’ll get to it one day; I released
funtrace before getting around to this bit, but generally, working around a large number of stupid things like this is a big
chunk of what I do.

## Knowing what you shouldn’t know

If I manage to stay out of trouble, it’s rarely because of knowing that much, but more because I’m relatively good at 2 other
things: knowing what I don’t know, and knowing what I shouldn’t know. To look at our example, you could argue that the above
explanations are shallower than they could be — I ask why something was done instead of looking up the history, and I only
briefly touch on what `TI_MODULE_OFFSET` and `TI_OFFSET_OFFSET` (yes, TI\_OFFSET\_OFFSET) are, and I don’t
say a word about GL\_TLS\_GENERATION\_OFFSET, for example, and I _could._

I claim that the kind of things we saw around \_\_tls\_get\_addr is an immediate red flag along the lines of, yes I am looking
into low-level stuff, but no, nothing good will come out of knowing this particular bit very well in the context that I’m in
right now; maybe I’ll be forced to learn it sometime, but right now this looks exactly like stuff I should avoid rather than
stuff I should learn.

I don’t know how to generalize the principle to make it explicit and easy to follow. All I can say right now is that the next
section has examples substantiating this feeling; you mainly want to avoid `__tls_get_addr`, because even people who
know it very well, because they maintain it and everything related to it, run into problems with it.

I’ve recently been seeing the expression “anti-intellectualism” used by people criticizing arguments along the lines of “this
is too complex for me to understand, so this can’t be good.” While I agree that we want some more concrete argument about why
something isn’t worth understanding than “I don’t get it, and I _would_ get it if it was any good,” I implore not to call
this “anti-intellectualism,” lest we implicitly crown ourselves as “intellectuals” over the fact that we understand what
TI\_OFFSET\_OFFSET is. It’s ridiculous enough that we’re called “knowledge workers,” when the “knowledge” referred to in this
expression is the knowledge of what TI\_OFFSET\_OFFSET is.

## Workarounds for shared libraries

Like I said, it annoys me to no end that TLS access is slow for variables defined in shared libraries. Readers suggested
quite a few workarounds, "dirty" to varying degrees:

### "Inlining" `pthread_getspecific`

There's a pthreads API for allocating "thread-specific keys" which is a form of TLS. Calling `pthread_getspecific`
upon every TLS access isn't any better than calling `__tls_get_addr`. But [we can "inline" the code of glibc's implementation](https://x.com/pskocik/status/1891494684863680663), and if we can
make sure that our key is the first one allocated, it will take just a couple of assembly instructions (loading a pointer from
`%fs` with a constant offset, and then loading our data from that pointer):

```
#define tlsReg_ (__extension__( \
  { char*r; __asm ("mov %%fs:0x10,%0":"=r"(r)); r; }))

inline void *pxTlsGetLt32_m(pthread_key_t Pk){
  assert(Pk<32);
  return *(void**)(tlsReg_+0x310+sizeof(void*[2])*Pk+8);
}
void* getKey0(void) {
  return pxTlsGetLt32_m(0);
}

```

`getKey0` compiles to:

```
  mov  %fs:0x10,%rax
  mov  0x318(%rax),%rax

```

### Compiling with `-ftls-model=initial-exec`

It [turns out](https://news.ycombinator.com/item?id=43078859) that there's something called the " [initial\
exec TLS model](https://maskray.me/blog/2021-02-14-all-about-thread-local-storage#initial-exec-tls-model-executable-preemptible)", where a TLS access costs you 2 instructions and no function calls:

```
movq tls_obj@GOTTPOFF(%rip), %rax
movl %fs:(%rax), %eax

```

You can also make just some variables use this model with `__attribute((tls_model("initial-exec")))`, instead of
compiling everything with `-ftls-model=initial-exec`, which might be very useful since the space for such variables
is a scarce resource as we'll see shortly.

This method is great if you can `LD_PRELOAD` your library, or link the executable against it so that it becomes
`DT_NEEDED`. Otherwise, this may or may not work at runtime:

> the shared object generally needs to be an immediately loaded shared object. The linker sets the DF\_STATIC\_TLS flag to
> annotate a shared object with initial-exec TLS relocations.
>
> glibc ld.so reserves some space in static TLS blocks and allows dlopen on such a shared object if its TLS size is small.
> There could be an obscure reason for using such an attribute: general dynamic and local dynamic TLS models are not
> async-signal-safe in glibc. However, other libc implementations may not reserve additional TLS space for dlopen'ed initial-exec
> shared objects, e.g. musl will error.

### Faster `__tls_get_addr` with `-mtls-dialect=gnu2`

It turns out there's a faster `__tls_get_addr` which you can opt into using. This is still too much code for my
taste; but if you're intereseted in the horrible details, you can read [the comment where I found out about this](https://news.ycombinator.com/item?id=43079061).

## See also

Various compiler and runtime issues make this slow stuff even slower, and it takes a while to get it fixed. If you stay
within the guidelines above, you should avoid such problems; if you don’t, you might have more problems than described above —
including both performance and correctness:

- [mulitple calls to \_\_tls\_get\_addr() with -fPIC](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=81501) (reported in
  2017, status: NEW as of 2025). Some highlights from 2022:

  - “We recently upgraded our toolchain from GCC9 to GCC11, and **we're seeing `__tls_get_addr` take up to 10%**
    **of total runtime** under some workloads, where it was 1-2% before. It seems that some changes to the optimization passes
    in 10 or 11 have significantly increased the impact of this problem.”
  - “I've shown a workaround I used, which might be useful until GCC handle `__tls_get_addr()` as returning a
    constant addresses that doesn't need to be looked up multiple times in a function.“
  - “Thanks for the patch! I wonder if it would handle coroutines correctly. **Clang has this open bug ["Compiler incorrectly caches thread\_local address across\**
    **suspend-points"](https://github.com/llvm/llvm-project/issues/47179) that is related to this optimization**.”
- [TLS performance degradation after dlopen](https://sourceware.org/bugzilla/show_bug.cgi?id=19924) (reported in
  2016; fixed in libc 2.39 in 2023, backported to older libcs up to 2.34 in 2025):

  - “we have noticed a performance degradation of TLS access in shared libraries. **If another shared library that uses TLS is**
    **loaded via dlopen, \_\_tls\_get\_addr takes significant more time**. Once that shared library accesses it's TLS, the performance
    normalizes. We do have a use-case where this is actually really significant.”
  - “elf: Fix slow tls access after dlopen \[BZ #19924\] In short: \_\_tls\_get\_addr checks the global generation counter and if the
    current dtv is older then \_dl\_update\_slotinfo updates dtv up to the generation of the accessed module. So if the global
    generation is newer than generation of the module then \_\_tls\_get\_addr keeps hitting the slow dtv update path. The dtv update
    path includes a number of checks to see if any update is needed and this already causes measurable tls access slow down after
    dlopen. It may be possible to detect up-to-date dtv faster. But if there are many modules loaded (> TLS\_SLOTINFO\_SURPLUS)
    then this requires at least walking the slotinfo list. This patch tries to update the dtv to the global generation instead, so
    after a dlopen the tls access slow path is only hit once. The modules with larger generation than the accessed one were not
    necessarily synchronized before, so additional synchronization is needed.”
  - “the fix for [bug 19924](https://sourceware.org/bugzilla/show_bug.cgi?id=19924) was to update DTV on tls access
    up to the global gen count so after an independent dlopen the next tls access updates the DTV gen count instead of falling into
    a slow code path over and over again. **this introduced some issues**: update happens now even if the accessed tls
    is in an early loaded library that use static tls (l\_tls\_offset is set), so such access is no longer as-safe and may alloc. some
    of this was mitigated by an ugly workaround: “elf: Support recursive use of dynamic TLS in interposed malloc.” a possible better
    approach is to expose the gen count of the accessed module directly in the tls\_get\_addr argument: this is possible on 64bit
    targets if we compress modid and offset into one GOT entry and use the other for the gen count when processing DTPMOD and DTPREL
    relocs. (then the original logic before the 19924 fix would not slow down after a global gencount bump: we can compare the DTV
    gen count to the accessed module gen count. btw we do this with TLSDESC today and thus aarch64 was imho not affected by the
    malloc interposition issue.) however i feel this is dancing around a bad design to use the generation count to deal with dlclose
    and reused modids. so here is a better approach…”

If you’re not quite following some of the above, this sort of makes my point about `__tls_get_addr` being
undesirable, though I am not sure how to defend this way of making a point in the general case.

_Thanks to Dan Luu for reviewing a draft of this post._

* * *

1. You could instead write the data straight to DRAM, by putting your trace buffer into memory mapped with the
   “uncached” attribute in the processor’s page table. But operating systems don’t make it very easy to allocate such memory, and
   for good reason — this can save you cache-related overheads, but writing 64-bit words to DRAM wastes 7/8ths of the bandwidth,
   since DRAM writes work in “bursts” of 64 bytes. That’s why CPUs use DRAM bandwidth efficiently when writing back full cache
   lines, but not when writing individual words to uncached memory. And of course none of this would help with maintaining a next
   write position variable shared between threads. [↩︎](#fnref1)

2. The nature of C++ combines with human nature in such a way that people often want to have some Registry of
   Plugins, with some libmyplugin.a implementing a MyPlugin derived from Plugin and calling Registry::add(this) in the constructor
   of MyPlugin g\_plugin. Of course this runs into the following wonderful provision in the standard: “It is implementation-defined
   whether initialization happens before main is entered or is delayed until the first non-initialization odr-use of a non-inline
   definition in the same translation unit as the one containing the variable definition.” Which of course is a bit of a
   chicken-and-egg problem, as nothing from libmyplugin.a ever gets executed unless the plugin is added to the registry — but it’s
   not going to be added to the registry “until something from the plugin gets executed” (though in practice it will happen
   earlier, before main — what actually happens is that the linker notices that a symbol from the translation unit gets used and
   then _doesn’t_ drop all the code from the translation unit defining the global constructor; the language in the standard
   just gives implementations the maximal possible leeway to make your life miserable.) I refer to the standard accepting this
   nonsense (where your code works if it’s kept in a .cpp that’s compiled into a .o and then linked, but not if the .o is put into
   a lib.a and then the lib.a is linked) and filing it under legitimate “implementation-defined” behavior as “enshrining
   decades-old bugs as a standard,” though you could say that it’s not a bug as much as the linker not bothering to implement an
   obviously correct behavior that was not a part of its original specification. I had other examples from my time of transitioning
   from gold to lld and mold, but thankfully they have largely faded from my memory. [↩︎](#fnref2)

3. Apparently we can say that it's trade-off made _by the implementation_ since the standard [permits](https://stackoverflow.com/questions/24253584/when-is-a-thread-local-global-variable-initialized)
   implementations to choose any time for calling the constructor of a thread\_local before the first use, same as it does for
   "normal" global variables, where implementations normally choose to call the constructor before main unconditionally, without
   having a guard variable for lazy initialization: "3.7.2/2 \[basic.stc.thread\]: A variable with thread storage duration shall be
   initialized before its first odr-use (3.2) and, if constructed, shall be destroyed on thread exit." [↩︎](#fnref3)

4. You also can’t really deallocate a TLS area when a shared library is unloaded if you allocate them all
   contiguously — you could theoretically reuse that area for the TLS of another shared library, but only if it fits into that
   address range; you could unmap the physical pages and have the OS use them for other allocation requests, but you might be stuck
   with the virtual address space range of the TLS area, with no uses for it in sight. Is this very unlikely case — lots of
   dlopen/dlclose and no dlopening of the same previously dlclosed library where it would fit perfectly into the “TLS address range
   gap” — one reason this kind of design would be rejected?.. Like, I’d still totally do it, and I did a bunch of work of this
   sort, but maybe it offends the sensibilities of most people doing these things?.. [↩︎](#fnref4)

5. Actually two `data16` prefixes are so pointless that while gdb _disassembles_ this instruction
   sequence just fine, the GNU assembler presumably refuses to _assemble_ it, so the compiler generates the delightfully
   self-explanatory code `.value 0x6666` (0x66 being the encoding of `data16`) followed by `rex64`
   and then the `call`. I don’t know why gdb’s disassembler doesn’t show us the `rex64`; I’m not by any
   measure an x86 guy. I’m just someone who thinks that this stuff has to work somehow and there must be a way to make it do the
   thing you want, and I try to learn enough to seem to be able to do it when I need it. [↩︎](#fnref5)

6. You set the visibility to "hidden" with `-fvisibility-hidden`,
   `__attribute__((visibility ("hidden")))`, or `[[gnu::visibility("hidden")]]` [↩︎](#fnref6)
