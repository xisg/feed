---
title: Hiring (self-driving algos, HLL compiler research)
url: https://yosefk.com/blog/hiring-self-driving-algos-hll-compiler-research.html
published: "2016-09-11T00:00:00Z"
feed: yosefk
---

# Hiring (self-driving algos, HLL compiler research)

OK, so 2 things:

1\. If you send me a CV and they're hired to work on self-driving algos – machine vision/learning/mapping/navigation, I'll pay
you a shitton of money. (Details over email.) These teams want CS/math/physics/similar degree with great grades, and they want
programming ability. They'll hire quite a lot of people.

2\. The position below is for my team and if you refer a CV, I cannot pay you a shitton of money. But:

**We're developing an array language that we want to efficiently compile to our in-house accelerators (multiple target**
**architectures, you can think of it as "compiling to a DSP/GPU/FPGA.")**

Of recent public efforts, perhaps [Halide](http://halide-lang.org/) is the closest relative (we're compiling AOT
instead of processing a graph of C++ objects constructed at run time, but I'm guessing the work done at the back-end is somewhat
similar.) What we have now is already beating hand-optimized code in our C dialects on some programs, but it's still a "blue
sky" effort in that we're not sure exactly how far it will go (in terms of the share of production programs where it can replace
our C dialects.)

As usual, we aren't looking for someone with experience in exactly this sort of thing (here especially it'd be hopeless since
there are few compiler writers and most of them work on lower-level languages.) Historically, the people who enjoy this kind of
work have a background in what I broadly call (mislabel?) "discrete math" -  formal methods, theory of computation, board game
AI, even cryptography, basically anywhere where you have clever algorithms in a discrete space that can be shown to work every
time. (Heavyweight counter-examples missing one of "clever", "discrete" or "every time" – OSes, rendering, or NNs. This of
course is not to say that experience in any of these is disqualifying, just that they're different.)

I think of it as a gig combining depth that people expect from academic work with compensation that people expect from
industry work. If you're interested, email me (Yossi.Kreinin@gmail.com).

All positions are in Jerusalem.
