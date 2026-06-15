---
title: 'LLVM Fortran Levels Up: Goodbye flang-new, Hello flang!'
url: https://blog.llvm.org/posts/2025-03-11-flang-new/
published: "2025-03-11T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2025-03-11-flang-new/
---

# LLVM Fortran Levels Up: Goodbye flang-new, Hello flang!

LLVM has included a Fortran compiler “Flang” since LLVM 11 in late 2020. However,until recently the Flang binary was not `flang` (like `clang`) but instead `flang-new`.

LLVM 20 ends the era of `flang-new`. The community has decided that Flang isworthy of a new name.

The “new” name? You guessed it, `flang`.

A simple change that represents a major milestone for Flang.

This article will cover the almost 10 year journey of Flang. The firstconcepts, multiple rewrites, the adoption of LLVM’s Multi Level IntermediateRepresentation (MLIR) and Flang entering the LLVM Project.

If you want to try `flang` right now, you can [download](https://github.com/llvm/llvm-project/releases/tag/llvmorg-20.1.0) it or try it in your browser using [Compiler Explorer](https://godbolt.org/z/3hhYM37Kh).

# Why Fortran?

Fortran was first created in the 1950s, and the name came from “Formula Translation”.Fortran focused on the mathematics use case and freed programmers from writingassembly code that could only run on specific machines.

Instead they could write code that looked like a formula. You expect this todaybut for the time it was a revolution. This feature led to heavy use in scientificcomputing: weather modelling, fluid dynamics and computational chemistry, justto name a few.

> Whilst many alternative programming languages have comeand gone, it \[Fortran\] has regained its popularity for writing highperformance codes. Indeed, over 80% of the applicationsrunning on ARCHER2, a 750,000 core Cray EX which isthe UK national supercomputer, are written in Fortran.

- [Fortran High-Level Synthesis: Reducing the barriersto accelerating High Performance Computing (HPC) codes on FPGAs](https://arxiv.org/pdf/2308.13274) (Gabriel Rodriguez-Canal et al., 2023)

Fortran has had a [resurgence](https://ondrejcertik.com/blog/2021/03/resurrecting-fortran/) in recent years, gaining a [package manager](https://fpm.fortran-lang.org/), an unofficial [standard library](https://github.com/fortran-lang/stdlib) and [LFortran](https://lfortran.org/),a compiler that supports interactive programming (LFortran also uses LLVM).

For the full history of Fortran, IBM has an excellent [article](https://www.ibm.com/history/fortran) on the topic and I encourage you to look at the [“Programmer’s Primer for Fortran”](https://archive.computerhistory.org/resources/text/Fortran/102653987.05.01.acc.pdf) if you want to see the early form of Fortran.

If you want to learn the language, [fortran-lang.org](https://fortran-lang.org/) is a great place to start.

# Why Would You Make Another Fortran Compiler?

There are many Fortran compilers. Some are vendor specific such as the [Intel Fortran Compiler](https://www.intel.com/content/www/us/en/developer/tools/oneapi/fortran-compiler.html) or NVIDIA’s [HPC compilers](https://developer.nvidia.com/hpc-compilers). Thenthere are open source options like [GFortran](https://gcc.gnu.org/fortran/), whichsupports many platforms.

Why build one more?

The two partners in the early days of Flang were the US National Labs and NVIDIA.

For Pat McCormick (Flang project lead at Los Alamos National Laboratory) preservingthe utility of Fortran code was imperative:

> These \[Fortran\] codes represent an essential capability that supports manyelements of our \[The United States’\] scientific mission and will continue to doso for the foreseeable future. A fundamental risk facing these codes is theabsence of a long-term, non-proprietary support path for Fortran.

GFortran might seem to counter that statement, but remember that a single projectis a single point of failures, incompatibilities and disagreements. Having multipleimplementations reduces that risk.

NVIDIA’s Gary Klimowicz [laid out](https://www.youtube.com/watch?v=Fy68k5hHgLk) their goals for Flang in a presentation to FortranCon in 2020:

- Use a permissive license like that of [LLVM](https://llvm.org/LICENSE.txt),which is more palatable to commercial users and contributors.
- Develop an active community of Fortran compiler developers that includescompanies and institutions.
- Support Fortran tool development by basing Flang on existing LLVM frameworks.
- Support Fortran language experimentation for future language standards proposals.

Intentions echoed by Pat McCormick:

> The overarching goal was to establish an open-source, modern implementation andsimultaneously grow a community that spanned industry, academia, and federalagencies at both the national and international levels.

Fortran as a language also benefits from having many implementations. For C++language features, it is common to implement them on top of Clang and GCC, toprove the feature is viable and get feedback.

Implementing the feature multiple times in different compilers uncoversassumptions that may be a problem for certain compilers, or certain groups ofcompiler users.

In the same way, Flang and GFortran can provide that diversity.

However, even when features are standardised, standards can be ambiguous andimplementations do make mistakes. A new compiler is a chance to uncover these.

Jeff Hammond (NVIDIA) is very familiar with this, having tested Flang with manyexisting applications. They had this to say on the motivations for Flangand how users have reacted to it:

> The Fortran language has changed quite a bit over the past 30 years. Modern Fortrandeserves a modern compiler ecosystem, that’s not only capable of compiling allthe old codes and all the code written for the current standard, but also supportsinnovation in the future.
>
> Because it’s a huge amount of work to build a feature-complete modern Fortran compiler,it’s useful to leverage the resources of the entire LLVM community for this effort.NVIDIA and ARM play leading roles right now, with important contributions from IBM,Fujitsu and LBNL \[Lawrence Berkeley National Laboratory\], e.g. related to testsuites and coarrays. We hope to see the developer community grow in the future.
>
> Another benefit from the LLVM Fortran compiler is that users are more likely toinvest in supporting a new compiler when it has full language support and runs onall the platforms. A broad developer base is critical to support all the platforms.
>
> What I have seen so far interacting with our Fortran users is that they are veryexcited about LLVM Flang and were willing to commit to supporting it in theirbuild systems and CI systems, which has driven quality improvements in both theFlang compiler and the applications.
>
> Like Clang did with C and C++ codes when it started to become popular, Flangis helping to identify bugs in Fortran code that weren’t noticed before, whichis making the Fortran software ecosystem better.

# PGI to LLVM: The Flang Timeline

The story of Flang really starts in 2015, but the Portland Group (PGI) collaboratedwith US National Labs prior to this. PGI would later become part of NVIDIA andbe instrumental to the Flang project.

- **1989** The [Portland Group](https://web.archive.org/web/19970628161656/http://www.pgroup.com/corp_home.html) is formed. To provide C, Fortran 77 and C++ compilers for the Intel i860 market.
- **1990** Intel bundles PGI compilers with its iPSC/860 supercomputer.
- **1996** PGI [works with](https://web.archive.org/web/20100528051556/http://www.sandia.gov/ASCI/Red/papers/Mattson/OVERVIEW.html) Sandia National Laboratories to provide compilers for the Accelerated Strategic Computing Initiative (ASCI) Option Redsupercomputer.
- **December 2000** PGI becomes a [wholly owned subsidiary](https://www.electronicsweekly.com/news/archived/resources-archived/stmicroelectronics-to-acquire-pgi-2000-12/) ofSTMicroElectronics.
- **August 2011** Away from PGI, Bill Wendling [starts](https://github.com/llvm-fortran/fort/commit/af352bf765ecf3e55da38c34cb480b269a157894) an LLVM based Fortran compiler called “Flang” (later known as [“Fort”](https://github.com/llvm-fortran/fort)).Bill is joined by several collaborators a few months later.
- **July 2013** PGI is [sold to NVIDIA](https://www.theregister.com/2013/07/30/nvidia_buys_the_portland_group/).

In late 2015 there were the first signs of what would become “Classic Flang”. Thoughat the time it was just “Flang”, I will use “Classic Flang” here for clarity.

Development of what was to become “Fort” continued under the “Flang” name,completely separate from the Classic Flang project.

- **November 2015** NVIDIA joins the US Department of EnergyExascale Computing Project. Including a commitment to create an open source [Fortran compiler](https://www.llnl.gov/article/41756/nnsa-national-labs-team-nvidia-develop-open-source-fortran-compiler-technology).


  > “The U.S. Department of Energy’s National Nuclear Security Administration and itsthree national labs \[Los Alamos, Lawrence Livermore and Sandia\] have reached anagreement with NVIDIA’s PGI division to adapt and open-source PGI’s Fortranfrontend, and associated Fortran runtime library, for contribution to the LLVM project.”


  (this news is also the first appearance of Flang in an issue of [LLVM Weekly](https://llvmweekly.org/issue/98))

- **May 2017** The first release of Classic Flang as a separaterepository, outside of the LLVM Project. Composed of a PGI compiler frontendand a new backend that generates LLVM Intermediate Representation (LLVM IR).

- **August 2017** The Classic Flang project is [announced officially](https://llvmweekly.org/issue/191)(according to LLVM Weekly’s report, the original mailing list is offline).


During this time, plans were formed to propose moving Classic Flang into the LLVMProject.

- **December 2017** The original “Flang” is renamed to [“Fort”](https://github.com/llvm-fortran/fort/commit/0585746476e3c1abe8ab4109b9dd98483cabdf09) so as not to compete with Classic Flang.

- **April 2018** Steve Scalpone (NVIDIA) [announces](https://www.youtube.com/watch?v=sFVRQDgKihY) at the European LLVM Developers’ Conference that the frontend of Classic Flang will be rewritten to addressfeedback from the LLVM community. This new front end became known as “F18”.

- **August 2018** Eric Schweitz (NVIDIA) begins work on what would become“Fortran Intermediate Representation”, otherwise known as “FIR”. This work wouldlater become the `fir-dev` branch.

- **February 2019** Steve Scalpone [proposes](https://lists.llvm.org/pipermail/llvm-dev/2019-February/130497.html) contributing F18 to the LLVM Project.

- **April 2019** F18 is [approved](https://discourse.llvm.org/t/f18-is-accepted-as-part-of-llvm-project/51719) for migration into the LLVM Project monorepo.

  At this point F18 was only the early parts of the compiler, it could not generatecode (later `fir-dev` work addressed this). Despite that, it moved into `flang/` in the monorepo, awaiting the completion of the rest of the work.

- **June 2019** Peter Waller (Arm) [proposes](https://discourse.llvm.org/t/rfc-adding-a-fortran-mode-to-the-clang-driver-for-flang/52307) adding a Fortran mode to the Clang compiler driver.

- **August 2019** The [first appearance](https://github.com/flang-compiler/f18/commit/b6c30284e7876f6ccd4bb024bd5f349128e99b7c) of the `flang.sh` driver wrapper script (more on this later).

- **December 2019** The [plan](https://discourse.llvm.org/t/flang-landing-in-the-monorepo/54022) for rewriting the F18 git history to fit into the LLVM project is announced.This effort was led by Arm, with Peter Waller going so far as to writea [custom tool](https://github.com/llvm/llvm-project/commit/55d5e6cbe2509a24132d056e1f361dc39312929b#diff-c389405236998090c7c8b9741506f01fb28abbd7da52e9566323c585ac0eb89cL910) to rewrite the history of F18.

  Kiran Chandramohan (Arm) [proposes](https://groups.google.com/a/tensorflow.org/g/mlir/c/SCerbBpoxng/m/bVqWTRY7BAAJ) an OpenMP dialect for MLIR, with the intention of using it in Flang (discussioncontinues on [Discourse](https://discourse.llvm.org/t/rfc-openmp-dialect-in-mlir/397) during the following January).

- **February 2020** The [plan](https://discourse.llvm.org/t/plan-for-landing-flang-in-monorepo/54546) for improvements to F18 to meet the standards required for inclusion in theLLVM monorepo is announced by Richard Barton (Arm).

- **April 2020** Upstreaming of F18 into the LLVM monorepo is [completed](https://github.com/llvm/llvm-project/commit/b98ad941a40c96c841bceb171725c925500fce6c).


At this point what was in the LLVM monorepo was F18, the rewritten frontend ofClassic Flang. Classic Flang remained unchanged, still using the PGI based frontend.

Around this time work started in the Classic Flang repo on the `fir-dev` branchthat would enable code generation when using F18.

For the following events remember that Classic Flang was still in use. The ClassicFlang binary is named `flang`, just like the folder F18 now occupies in the LLVM Project.

**Note:** Some LLVM changes referenced below will appear to have skipped an LLVM release.This is because they were done after the release branch was created, but beforethe first release from that branch was distributed.

- **April 2020** The first attempt at adding a new compiler driver for Flang is [posted](https://reviews.llvm.org/D79092) for review. It used the name `flang-tmp`. This change was later abandoned in favour of a different approach.

- **September 2020** Flang’s new compiler driver is [added](https://reviews.llvm.org/D86089) as an experimental option. This is the first appearance of the `flang-new` binary,instead of `flang-tmp` as proposed before.


  > The name was intended as temporary, but not the driver.


  - Andrzej Warzyński (Arm, Flang Driver Maintainer)
- **October 2020** Flang is included in an LLVM release for the first time inLLVM 11.0.0. There is an `f18` binary and the previously mentioned script `flang.sh`.

- **August 2021** `flang-new` is no longer experimental and [replaces](https://reviews.llvm.org/D105811) the previous Flang compiler driver binary `f18`.

- **October 2021** LLVM 13.0.0 is the first release to include a `flang-new` binary(alongside `f18`).

- **March 2022** LLVM 14.0.0 releases, with `flang-new` replacing `f18` as the Flangcompiler driver.

- **April 2022** NVIDIA [ceases development](https://discourse.llvm.org/t/nvidia-transition-from-fir-dev/61947) of the `fir-dev` branch in the Classic Flang project. Upstreaming of `fir-dev` to the LLVM Project begins around this date.

  `flang-new` can now do [code generation](https://reviews.llvm.org/D122008) if the `-flang-experimental-exec` option is used. This change used workoriginally done on the `fir-dev` branch.

- **May 2022** Kiran Chandramohan [announces](https://www.youtube.com/watch?v=FoIjafZGDdE) at the European LLVM Developers’ Meeting that Flang’s OpenMP 1.1 support is close to complete.

  The `flang.sh` compiler driver script becomes `flang-to-external-fc`. Itallows the user to use `flang-new` to parse Fortran source code, then write it backto a file to be compiled with an existing Fortran compiler.

  The script can be put in place of an existing compiler to test Flang’s parser onlarge projects.

- **June 2022** Brad Richardson (Berkeley Lab) [changes](https://reviews.llvm.org/D153379) `flang-new` to generate code by default, removing the `-flang-experimental-exec` option.

- **July 2022** Valentin Clément (NVIDIA) [announces](https://discourse.llvm.org/t/nvidia-transition-from-fir-dev/61947/5) that upstreaming of `fir-dev` to the LLVM Project is complete.

- **September 2022** LLVM 15.0.0 releases, including Flang’s experimental codegeneration option.

- **September 2023** LLVM 17.0.0 releases, with Flang’s code generation enabledby default.


At this point the LLVM Project contained Flang as it is known today. Sometimesreferred to as “LLVM Flang”.

“LLVM Flang” is the combination of the F18 frontend and MLIR-based code generationfrom `fir-dev`. As opposed to “Classic Flang” that combines a PGI based frontend andits own custom backend.

The initiative to upstream Classic Flang was in some sense complete. Thoughwith all of the compiler rewritten in the process, what landed in the LLVM Projectwas very different to Classic Flang.

- **April 2024** The `flang-to-external-fc` script is [removed](https://github.com/llvm/llvm-project/pull/88904).

- **September 2024** LLVM 19.1.0 releases. The first release of `flang-new` as a standalone compiler.

- **October 2024** The community deems that Flang has met the criteria to not be“new” and the name is changed. Goodbye `flang-new`, hello `flang`!

- **November 2024** AMD [announces](https://rocm.blogs.amd.com/ecosystems-and-partners/fortran-journey/README.html) its next generation Fortran compiler, based on LLVM Flang.

  Arm [releases](https://developer.arm.com/Tools%20and%20Software/Arm%20Compiler%20for%20Linux#Downloads) an experimental versionof its new Arm Toolchain for Linux product, which includes LLVM Flangas the Fortan compiler.

- **March 2025** LLVM 20.1.0 releases. The first time the `flang` binary has beenincluded in a release.


# Flang and the Definition of New

Renaming Flang was discussed a few times [before](https://discourse.llvm.org/t/reviving-rename-flang-new-to-flang/68130/1) the final proposal. It was always contentious, so for the final [proposal](https://discourse.llvm.org/t/proposal-rename-flang-new-to-flang/69462) Brad Richardson decided to use the [LLVM proposal process](https://github.com/llvm/llvm-www/blob/main/proposals/LP0001-LLVMDecisionMaking.md).Rarely used, but specifically designed for these situations.

> After several rounds of back and forth, I thought the discussion wasdevolving and there wasn’t much chance we’d come to a consensus without someoutside perspective.

- Brad Richardson

That outside perspective included Chris Lattner (co-founder of the LLVM Project),who quickly [identified](https://discourse.llvm.org/t/proposal-rename-flang-new-to-flang/69462/25) a unique problem:

> We have a bit of an unprecedented situation where an LLVM project is takingthe name of an already established compiler \[Classic Flang\]. Everyone seems towant the older flang \[Classic Flang\] to fade away, but flang-new is not asmature and it isn’t clear when and what the criteria should be for that.

Confusion about the `flang` name was a key motivation for Brad Richardson too:

> Part of my concern was that the name “flang-new” would get common usagebefore we were able to change it. I think it’s now been demonstrated that thatconcern was valid, because right now \[November 2024\] fpm \[ [Fortran Package Manager](https://fpm.fortran-lang.org/)\]recognizes the compiler by that name.
>
> My main goal at that point was just clear goals for when we wouldmake the name change.

No single list of goals was agreed, but some came up many times:

- Known limitations and supported features should be documented.
- As much as possible, work that was expected to fix knownbugs should be completed, to prevent duplicate bug reports.
- Unimplemented language features should fail with a message saying that they areunimplemented. Rather than with a confusing failure or by producing incorrectcode.
- LLVM Flang should perform relatively well when compared to other Fortrancompilers.
- LLVM Flang must have a reasonable pass rate with a large Fortran language testsuite, and results of that must be shown publicly.
- All reasonable steps should be taken to prevent anyone using a pre-packagedClassic Flang confusing it with LLVM Flang.

You will see a lot of relative language in those, like “reasonable”. Noone could say exactly what that meant, but everyone agreed that it wasinevitable that one day it would all be true.

Paul T Robinson summarised the dilemma [early](https://discourse.llvm.org/t/proposal-rename-flang-new-to-flang/69462/15) in the thread:

> > the plan is to replace Classic Flang with the new Flang in the future.
>
> I suppose one of the relevant questions here is: Has the future arrived?

After that Steve Scalpone (NVIDIA) gave [their perspective](https://discourse.llvm.org/t/proposal-rename-flang-new-to-flang/69462/16) that it was not yet time to change the name.

So the community got to work on those goals:

- Many performance and correctness issues were addressed by the “High LevelFortran Intermediate Representation” (HLFIR) (which this article will explain later).
- A cross-company team including Arm, Huawei, Linaro, Nvidia and Qualcomm [collaborated](https://github.com/orgs/llvm/projects/35/views/1) to make it possible to build the popular [SPEC 2017](https://www.spec.org/cpu2017/) benchmark with Flang.
- Flang gained support for OpenMP up to version 2.5, and was able to compile OpenMPspecific benchmarks like [SPEC OMP](https://www.spec.org/omp2012/) and the [NAS Parallel Benchmarks](https://www.nas.nasa.gov/software/npb.html).
- Linaro [showed that](https://www.youtube.com/watch?v=Gua80XRPhyY) the performanceof Flang compared favourably with Classic Flang and was not far behind GFortran.
- The GFortran test suite was added to the [LLVM Test Suite](https://github.com/llvm/llvm-test-suite/tree/main/Fortran/gfortran),and Flang achieved good results.
- Fujitsu’s [test suite](https://github.com/fujitsu/compiler-test-suite) was madepublic and tested with Flang. The process to make IBM’s Fortran test suite publicwas started.

With all that done, in October of 2024 `flang-new` [became](https://github.com/llvm/llvm-project/commit/06eb10dadfaeaadc5d0d95d38bea4bfb5253e077) `flang`. The future had arrived.

> And it’s merged! It’s been a long (and sometimes contentious) process, butthank you to everyone who contributed to the discussion.

- Brad Richardson, [closing out](https://discourse.llvm.org/t/proposal-rename-flang-new-to-flang/69462/86) the proposal.

The goals the community achieved have certainly been worth it for Flang as acompiler, but did Brad achieve their own goals?

> What did I hope to see as a result of the name change? I wanted it to beeasier for more people to try it out.

So once you have finished reading this article, [download](https://github.com/llvm/llvm-project/releases/tag/llvmorg-20.1.0) Flang or try it out on [Compiler Explorer](https://godbolt.org/z/3hhYM37Kh).You know at least one person will appreciate it!

# Fortran Intermediate Representation (FIR)

All compilers that use LLVM as a backend eventually produce code in the form ofthe [LLVM Intermediate Representation](https://llvm.org/docs/LangRef.html)(LLVM IR).

A drawback of this is that LLVM IR does not include language specific information.This means that for example, it cannot be used to optimise arrays in a wayspecific to Fortran programs.

One solution to this has been to build a higher level IR that represents theunique features of the language, optimise that, then convert the result into LLVM IR.

Eric Schweitz (NVIDIA) started to do that for Fortran in late 2018:

> FIR was originally conceived as a high-level IR that would interoperate withLLVM but have a representation more friendly and amenable to Fortranoptimizations.

Naming is hard but Eric did well here:

> FIR was a pun of sorts. Fortran IR and meant to be evocative of the trees(Abstract Syntax Trees).

We will not go into detail about this early FIR, because [MLIR](https://mlir.llvm.org/) was revealed soon after Eric started the project and they quickly adopted it.

> When MLIR was announced, I quickly switched gears from building datastructures for a new “intermediate IR” to porting my IR design to MLIR andusing that instead.
>
> I believe FIR was probably the first “serious project” outside of Google tostart using MLIR.

The FIR work continued to develop, with Jean Perier (NVIDIA) joining Eric onthe project. It became its own public branch `fir-dev`, which was later contributedto the LLVM Project.

The following sections will go into detail on the intermediate representationsthat Flang uses today.

# MLIR

The journey from Classic Flang to LLVM Flang involved a rewrite of theentire compiler. This provided an opportunity to pick up new things fromthe LLVM Project. Most notably MLIR.

“Multi-Level Intermediate Representation” (MLIR) was first [introduced](https://www.youtube.com/watch?v=qzljG6DKgic) to the LLVMcommunity in 2019, around the time that F18 was approved to move into the LLVM Project.

The problem that MLIR addresses is the same one that Eric Schweitz tackled with FIR:It is difficult to map high level details of programming languagesinto LLVM IR.

You either have to attach them to the IR as metadata, try to recover thelost details later, or fight an uphill battle to add the details toLLVM IR itself. These details are crucial for producing optimised code in certainlanguages. (Fortran array optimisations were one use case referenced).

This led languages such as Swift and Rust to create their own IRs that includeinformation relevant to their own optimisations. After that IR has been optimisedit is converted into LLVM IR and goes through the normal compilation pipeline.

To implement these IRs they have to build a lot of infrastructure, but it cannotbe shared between the compilers. This is where MLIR comes in.

> The MLIR project aims to directly tackle these programming language design andimplementation challenges—by making it very cheap to define and introduce newabstraction levels, and provide “in the box” infrastructure to solve commoncompiler engineering problems.

- [“MLIR: A Compiler Infrastructure for the End of Moore’s Law”](https://arxiv.org/abs/2002.11054)(Chris Lattner, Mehdi Amini et al., 2020)

## Flang and MLIR

The same year MLIR debuted, Eric Schweitz gave a talk at the later USLLVM Developers’ meeting titled [“An MLIR Dialect for High-Level Optimization of Fortran”](https://www.youtube.com/watch?v=ff3ngdvUang).FIR by that point was implemented as an MLIR dialect.

> That \[switching FIR to be based on MLIR\] happened very quickly and I neverlooked back.
>
> MLIR, even in its infancy, was clearly solving many of the exact same problemsthat we were facing building a new Fortran compiler.

- Eric Schweitz

The MLIR community were also happy to have Flang on board:

> It was fantastic to have very quickly in the early days of MLIR a non-ML \[Machine Learning\] frontendto exercise features we built in MLIR in anticipation. It led us to course-correctin some cases, and Flang was a motivating factor for many feature requests.It contributed significantly to establishing and validating that MLIR had the right foundations.

- Mehdi Amini

Flang did not stop there, later adding another dialect [“High Level Fortran Intermediate Representation”](https://flang.llvm.org/docs/HighLevelFIR.html)(HLFIR) which works at a higher level than FIR. A big target of HLFIRwas array optimisations, that were more complex to handle using FIR alone.

> FIR was a compromise on both ends to some degree. It wasn’t trying to capturesyntactic information from Fortran, and I assumed there would be work done onan Abstract Syntax Tree. That niche would later be filled by “High Level FIR”\[HLFIR\].

- Eric Schweitz

## IRs All the Way Down

The compilation process starts with Fortran source code.

```fortran
subroutine example(a, b) real :: a(:), b(:) a = bend subroutine
```

( [Compiler Explorer](https://godbolt.org/z/8j3W46j3j))

The [subroutine](https://fortran-lang.org/learn/quickstart/organising_code/) `example` assigns array `b` to array `a`.

It is tempting to think of the IRs in a “stack” where each one is convertedinto the next. However, MLIR allows multiple “dialects” of MLIR to exist in thesame file.

(The steps shown here are the most important ones for Flang. In reality thereare many more between Fortran and LLVM IR.)

In the first step, Flang produces a file that is a mixture of HLFIR, FIRand the built-in MLIR dialect `func` (function).

```mlir
module attributes {<...>} { func.func @_QPexample(%arg0: !fir.box<!fir.array<?xf32>> {fir.bindc_name = "a"}, %arg1: !fir.box<!fir.array<?xf32>> {fir.bindc_name = "b"}) { %0 = fir.dummy_scope : !fir.dscope %1:2 = hlfir.declare %arg0 dummy_scope %0 {uniq_name = "_QFexampleEa"} : (!fir.box<!fir.array<?xf32>>, !fir.dscope) -> (!fir.box<!fir.array<?xf32>>, !fir.box<!fir.array<?xf32>>) %2:2 = hlfir.declare %arg1 dummy_scope %0 {uniq_name = "_QFexampleEb"} : (!fir.box<!fir.array<?xf32>>, !fir.dscope) -> (!fir.box<!fir.array<?xf32>>, !fir.box<!fir.array<?xf32>>) hlfir.assign %2#0 to %1#0 : !fir.box<!fir.array<?xf32>>, !fir.box<!fir.array<?xf32>> return }}
```

For example, the “dummy arguments” (the [arguments of a subroutine](https://fortran-lang.org/en/learn/quickstart/organising_code/#subroutines))are declared with `hlfir.declare` but their type is specified with `fir.array`.

As MLIR allows multiple dialects to exist in the same file, there is no need forHLFIR to have a `hlfir.array` that duplicates `fir.array`, unless HLFIR wantedto handle that differently.

The next step is to convert HLFIR into FIR:

```mlir
module attributes {<...>} { func.func @_QPexample(<...>) { <...> %c3_i32 = arith.constant 3 : i32 %7 = fir.convert %0 : (!fir.ref<!fir.box<!fir.array<?xf32>>>) -> !fir.ref<!fir.box<none>> %8 = fir.convert %5 : (!fir.box<!fir.array<?xf32>>) -> !fir.box<none> %9 = fir.convert %6 : (!fir.ref<!fir.char<1,17>>) -> !fir.ref<i8> %10 = fir.call @_FortranAAssign(%7, %8, %9, %c3_i32) : (!fir.ref<!fir.box<none>>, !fir.box<none>, !fir.ref<i8>, i32) -> none return }<...>}
```

Then this bundle of MLIR dialects is converted into LLVM IR:

```mlir
define void @example_(ptr %0, ptr %1) { <...> store { ptr, i64, i32, i8, i8, i8, i8, [1 x [3 x i64]] } %37, ptr %3, align 8 call void @llvm.memcpy.p0.p0.i32(ptr %5, ptr %4, i32 48, i1 false) %38 = call {} @_FortranAAssign(ptr %5, ptr %3, ptr @_QQclX2F6170702F6578616D706C652E66393000, i32 3) ret void}<...>
```

This LLVM IR passes through the standard compilation pipeline that clang also uses.Eventually being converted into target specific [Machine IR](https://llvm.org/docs/MIRLangRef.html) (MIR), into assembly andfinally into a binary program.

- Fortran
- MLIR (including HLFIR and FIR)
- MLIR (including FIR)
- LLVM IR
- MIR
- Assembly
- Binary

At each stage, the optimisations most suited to that stage are done.For example, while you have HLFIR you could optimise array accesses because at thatpoint you have the most information about how the Fortran treats arrays.

If Flang were to do this later on, in LLVM IR, it would be much more difficult.Either the information would be lost or incomplete, or you would be at a stage inthe pipeline where you cannot assume that you started with a specific sourcelanguage.

# OpenMP to Everyone

**Note:** Most of the points made in this section also apply to [OpenACC](https://www.openacc.org/) support in Flang. In the interest of brevity, Iwill only describe OpenMP in this article. You can find more about OpenACCin this [presentation](https://www.youtube.com/watch?v=vVmCLdSboWc).

## OpenMP Basics

[OpenMP](https://www.openmp.org/) is a standardised API for addingparallelism to C, C++ and Fortran programs.

Programmers mark parts of their code with “directives”. These directivestell the compiler how the work of the program should be distributed.Based on this, the compiler transforms the code and inserts calls to anOpenMP runtime library for certain operations.

This is a Fortran example:

```Fortran
SUBROUTINE SIMPLE(N, A, B) INTEGER I, N REAL B(N), A(N)!$OMP PARALLEL DO DO I=2,N B(I) = (A(I) + A(I-1)) / 2.0 ENDDOEND SUBROUTINE SIMPLE
```

(from [“OpenMP Application Programming Interface Examples”](https://www.openmp.org/wp-content/uploads/openmp-examples-4.5.0.pdf), [Compiler Explorer](https://godbolt.org/z/chjzs3o6r))

**Note:** Fortran arrays are [one-based](https://fortran-lang.org/en/learn/quickstart/arrays_strings/) by default. So the first element is at index 1. This example reads the previous element as well, so it starts `I` at 2.

`!$OMP PARALLEL DO` is a directive in the form of a Fortran comment (Fortrancomments start with `!`). `PARALLEL DO` starts a parallel “region” thatincludes the code from `DO` to `ENDDO`.

This tells the compiler that the work in the `DO` loop should be shared amongstall the threads available to the program.

Clang has [supported OpenMP](https://blog.llvm.org/2015/05/openmp-support_22.html) for many years now. The equivalent C++ code is:

```C++
void simple(int n, float *a, float *b){ int i; #pragma omp parallel for for (i=1; i<n; i++) b[i] = (a[i] + a[i-1]) / 2.0;}
```

( [Compiler Explorer](https://godbolt.org/z/Yh9jb8rKe))

For C++, the directive is in the form of a `#pragma` and attachedto the `for` loop.

LLVM IR does not know anything about OpenMP specifically, so Clang does all thework of converting the intent of the directives into LLVM IR. The output fromClang looks like this:

```mlir
define dso_local void @simple(int, float*, float*) (i32 noundef %n, ptr noundef %a, ptr noundef %b) <...> {entry:<...> call void (<...>) @__kmpc_fork_call(@simple <...> (.omp_outlined) <...>) ret void}define internal void @simple(int, float*, float*) (.omp_outlined) (ptr <...> %.global_tid., ptr <...> %.bound_tid., ptr <...> %n, ptr <...> %b, ptr <...> %a) {entry:<...> call void @__kmpc_for_static_init_4(<...>)<...>omp.inner.for.body.i:<...>omp.loop.exit.i: call void @__kmpc_for_static_fini(<...>)<...> ret void}
```

(output edited for readability)

The body of `simple` no longer does all the work. Instead it uses `__kmpc_fork_call` to tell the OpenMP [runtime library](https://github.com/llvm/llvm-project/tree/main/openmp) to run another function, `simple (.omp_outlined)` to do the work.

This second function is referred to as a “micro task”. The runtime librarysplits the work across many instances of the micro task and each timethe micro task function is called, it gets a different slice of the work.

The number of instances is only known at runtime, and can be controlled withsettings such as [`OMP_NUM_THREADS`](https://www.openmp.org/spec-html/5.0/openmpse50.html).

The LLVM IR representation of `simple (.omp_outlined)` includes labels like `omp.loop.exit.i`, but these are not specific to OpenMP. They are just normal LLVM IRlabels whose name includes `omp`.

## Sharing Clang’s OpenMP Knowledge

Shortly after Flang was approved to join the LLVM Project, it was proposed thatFlang should share OpenMP support code with Clang.

> This is an RFC for the design of the OpenMP front-ends under the LLVMumbrella. It is necessary to talk about this now as Flang (aka. F18) ismaturing at a very promising rate and about to become a sub-project nextto Clang.
>
> TLDR;Keep AST nodes and Sema separated but unify LLVM-IR generation forOpenMP constructs based on the (almost) identical OpenMP directivelevel.

- “\[RFC\] Proposed interplay of Clang & Flang & LLVM wrt. OpenMP”,Johannes Doerfert (Lawrence Livermore National Laboratory), May 2019 (only one [part](https://discourse.llvm.org/t/rfc-proposed-interplay-of-clang-flang-llvm-wrt-openmp-flang-dev/51905) of this still exists online, this quote is from a copy of the other part, which was provided to me).

For our purposes, the “TLDR” means that although both compilers have differentinternal representations of the OpenMP directives, they both have to produceLLVM IR from that representation.

This proposal led to the creation of the `LLVMFrontendOpenMP` library in `llvm`. By using the same class `OpenMPIRBuilder`, there is no need to repeat work inboth compilers, at least for this part of the OpenMP pipeline.

As you will see in the following sections, Flang has diverged from Clang for otherparts of OpenMP processing.

## Bringing OpenMP to MLIR

Early in 2020, Kiran Chandramohan (Arm) [proposed](https://discourse.llvm.org/t/rfc-openmp-dialect-in-mlir/397) an MLIR dialect for OpenMP, for use by Flang.

> We started the work for the OpenMP MLIR dialect because of Flang.… So, MLIR has an OpenMP dialect because of Flang.

- Kiran Chandramohan

This dialect would represent OpenMP specifically, unlike the generic LLVM IRyou get from Clang.

If you compile the original Fortran OpenMP example without OpenMP enabled, youget this MLIR:

```mlir
module attributes {<...>} { func.func @_QPsimple(<...> { %1:2 = hlfir.declare %arg0 <...> {uniq_name = "_QFsimpleEn"} : <...> %3:2 = hlfir.declare %2 <...> {uniq_name = "_QFsimpleEi"} : <...> %10:2 = hlfir.declare %arg1(%9) <...> {uniq_name = "_QFsimpleEa"} : <...> %17:2 = hlfir.declare %arg2(%16) <...> {uniq_name = "_QFsimpleEb"} : <...> %22:2 = fir.do_loop <...> { <...> hlfir.assign %34 to %37 : f32, !fir.ref<f32> } fir.store %22#1 to %3#1 : !fir.ref<i32> return }}
```

(output edited for readability)

Notice that the `DO` loop has been converted into `fir.do_loop`. Now enableOpenMP and compile again:

```mlir
module attributes {<...>} { func.func @_QPsimple(<...> { %1:2 = hlfir.declare %arg0 <...> {uniq_name = "_QFsimpleEn"} : <...> %10:2 = hlfir.declare %arg1(%9) <...> {uniq_name = "_QFsimpleEa"} : <...> %17:2 = hlfir.declare %arg2(%16) <...> {uniq_name = "_QFsimpleEb"} : <...> omp.parallel { %19:2 = hlfir.declare %18 {uniq_name = "_QFsimpleEi"} : <...> omp.wsloop { omp.loop_nest (%arg3) : i32 = (%c2_i32) to (%20) inclusive step (%c1_i32) { hlfir.assign %32 to %35 : f32, !fir.ref<f32> omp.yield } } omp.terminator } return }}
```

(output edited for readability)

You will see that instead of `fir.do_loop` you have `omp.parallel`, `omp.wsloop` and `omp.loop_nest`. `omp` is an MLIR dialect that describes [OpenMP](https://mlir.llvm.org/docs/Dialects/OpenMPDialect/).

This translation of the `PARALLEL DO` directive is much more literal thanthe LLVM IR produced by Clang for `parallel for`.

As the `omp` dialect is specifically made for OpenMP, it can representit much more naturally. This makes it easier to understand the code and towrite optimisations.

Of course Flang needs to produce LLVM IR eventually, and to do that ituses the same `OpenMPIRBuilder` class that Clang does. From theMLIR shown previously, `OpenMPIRBuilder` produces the following LLVM IR:

```mlir
define void @simple_ <...> {entry: call void (<...>) @__kmpc_fork_call( <...> @simple_..omp_par <...>) ret void}define internal void @simple_..omp_par <...> {omp.par.entry: call void @__kmpc_for_static_init_4u <...>omp_loop.exit: call void @__kmpc_barrier(<...>) ret voidomp_loop.body: <...>}
```

The LLVM IR produced by Flang and Clang is superficially different, butstructurally very similar. Considering the differences in source languageand compiler passes, it is not surprising that they are not identical.

## ClangIR and the Future

It is surprising that a compiler for a language as old as Fortran got ahead ofClang (the most well known LLVM based compiler) when it came to adopting MLIR.

This is largely due to timing, MLIR is a recent invention and Clang existedbefore MLIR arrived. Clang also has a legacy to protect, so it is unlikely tomigrate to a new technology right away.

The [ClangIR](https://llvm.github.io/clangir/) project is working to changeClang to use a new MLIR dialect, “Clang Intermediate Representation” (“CIR”).Much like Flang and its HLFIR/FIR dialects, ClangIR will convert C and C++into the CIR dialect.

Work on OpenMP support for ClangIR has already [started](https://github.com/llvm/clangir/pull/382),using the `omp` dialect that was originally added for Flang.

Unfortunately at time of writing the `parallel` directive is not supported byClangIR. However, if you look at the CIR produced when OpenMP is disabled, you cansee the `cir.for` element that the OpenMP dialect might replace:

```mlir
module <...> attributes {<...>} { cir.func @_Z6simpleiPfS_( <...> { %1 = cir.alloca <...> ["a", init] <...> %2 = cir.alloca <...> ["b", init] <...> %3 = cir.alloca <...> ["i"] <...> cir.scope { cir.for : cond { <...> } body { <...> cir.yield loc(#loc13) } step { <...> cir.yield loc(#loc36) } loc(#loc36) } loc(#loc36) cir.return loc(#loc2) } loc(#loc31)} loc(#loc)
```

(on [Compiler Explorer](https://godbolt.org/z/Yj9EKK7ao))

# Flang Takes Driving Lessons

**Note:** This section paraphrases material from [“Flang Drivers”](https://github.com/llvm/llvm-project/blob/main/flang/docs/FlangDriver.md).If you want more detail please refer to that document, or [Driving Compilers](https://fabiensanglard.net/dc/index.php).

“Driver” in a compiler context means the part of the compiler that decideshow to handle a set of options. For instance, when you use the option `-march=armv8a+memtag`,something in Flang knows that you want to compile for Armv8.0-a with the MemoryTagging Extension enabled.

`-march=` is an example of a “compiler driver” option. These options are what usersgive to the compiler. There is actually a second driver after this, confusinglycalled the “frontend” driver, despite being behind the scenes.

In Flang’s case the “compiler driver” is `flang` and the “frontend driver” is `flang -fc1` (they are two separate tools, contained in the same binary).

They are separate tools so that the compiler driver can provide an interfacesuited to compiler users, with stable options that do not change over time.On the other hand, the frontend driver is suited to compiler developers, exposesinternal compiler details and does not have a stable set of options.

You can see the differences if you add `-###` to the compiler command:

```
$ ./bin/flang /tmp/test.f90 -march=armv8a+memtag -### "<...>/flang" "-fc1" "-triple" "aarch64-unknown-linux-gnu" "-target-feature" "+v8a" "-target-feature" "+mte" "/usr/bin/ld" \ "-o" "a.out" "-L/usr/lib/gcc/aarch64-linux-gnu/11"
```

(output edited for readability)

The compiler driver has split the compilation into a job for the frontend( `flang -fc1`) and the linker ( `ld`). `-march=` has been converted into manyarguments to `flang -fc1`. This means that if compiler developers decided tochange how `-march=` was converted, existing `flang` commands would still work.

Another responsibility of the compiler driver is to know where to find librariesand header files. This differs between operating systems and evendistributions of the same family of operating systems (for example Linuxdistributions).

This created a problem when implementing the compiler driver for Flang. All thesedetails would take a long time to get right.

Luckily, by this time Flang was in the LLVM Project alongside Clang.Clang already knew how to handle this and had been tested on all sorts ofsystems over many years.

> The intent is to mirror clang, for both the driver and CompilerInvocation, asmuch as makes sense to do so. The aim is to avoid re-inventing the wheel andto enable people who have worked with either the clang or flang entry points,drivers, and frontends to easily understand the other.

- [Peter Waller](https://discourse.llvm.org/t/rfc-adding-a-fortran-mode-to-the-clang-driver-for-flang/52307) (Arm)

Flang became the first in-tree project to use Clang’s compiler driverlibrary ( `clangDriver`) to implement its own compiler driver.

This meant that Flang was able to handle all the targets and tools that Clangcould, without duplicating large amounts of code.

# Reflections on Flang

We are almost 10 years from the first announcement of what would become LLVMFlang. In the LLVM monorepo alone there have been close to 10,000 commitsfrom around 400 different contributors. Undoubtedly more in Classic Flang beforethat.

So it is time to hear from users, contributors, and supporters, past andpresent, about their experiences with Flang.

> Collaborating with NVIDIA and PGI on Classic Flang was crucial in establishingArm in High Performance Computing. It has been an honour to continue investingin Flang, helping it become an integral part of the LLVM project and a solidfoundation for building HPC toolchains.
>
> We are delighted to see the project reach maturity, as this was the last step inallowing us to remove all downstream code from our compiler. Look out for ArmToolchain for Linux 20, which will be a fully open source, freely availablecompiler based on LLVM 20, available later this year.”

- Will Lovett, Director Technology Management at Arm.

(the following quote is presented in Japanese and English, in case of differences,Japanese is the authoritative version)

> 富士通は、我々の数十年にわたるHPCの経験を通じて培ったテストスイートを用いて、Flangの改善に貢献できたことを嬉しく思います。Flangの親切で協力的なコミュニティに大変感銘を受けました。
>
> 富士通は、より高いパフォーマンスと使いやすさを実現し、我々のプロセッサを最大限に活用するために、引き続きFlangに取り組んでいきます。Flangが改善を続け、ユーザーを増やしていくことを強く願っています。
>
> Fujitsu is pleased to have contributed to the improvement of Flang with ourtest suite, which we have developed through our decades of HPC experience.Flang’s helpful and collaborative community really impressed us.
>
> Fujitsu will continue to work on Flang to achieve higher performance andusability, to make the best of our processors. We hope that Flang will continueto improve and gain users.

- 富士通株式会社 コンパイラ開発担当 マネージャー 鎌塚　俊 (Shun Kamatsuka, Manager of the Compiler Development Team at Fujitsu).

> Collaboration between Linaro and Fujitsu on an active CI using Fujitsu’stestsuite helped find several issues and make Flang more robust, inaddition to detecting any regressions early.
>
> Linaro has been contributing to Flang development for two years now, fixing agreat number of issues found by the Fujitsu testsuite.

- Carlos Seo, Tech Lead at Linaro.

> [SciPy](https://scipy.org/) is a foundational Python package. It provides easyaccess to scientific algorithms, many of which are written in Fortran.
>
> This has caused a long stream of problems for packaging and shipping SciPy,especially because users expect first-class support for Windows;a platform that (prior to Flang) had no license-free Fortran compilersthat would work with the default platform runtime.
>
> As maintainers of SciPy and redistributors in the [conda-forge](https://conda-forge.org/) ecosystem, we hoped for a solution to this problem for many years. In the end,we switched to using Flang, and that [process](https://labs.quansight.org/blog/building-scipy-with-flang) was a minor miracle.
>
> Huge thanks to the Flang developers for removing a major source of pain for us!

- Axel Obermeier, Quantsight Labs.

> At the Barcelona Supercomputing Center, like many other HPC centers, we cannotignore Fortran.
>
> As part of our research activities, Flang has allowed us to apply our work inlong vectors for RISC-V to complex Fortran applications which we have been ableto run and analyze in our prototype systems. We have also used Flang to supportan in-house task-based directive-based programming model.
>
> These developments have proved to us that Flang is a powerful infrastructure.

- Roger Ferrer Ibáñez, Senior Research Engineer at the Barcelona Supercomputing Center (BSC).

> I am thrilled to see the LLVM Flang project achieve this milestone. It is a uniqueproject in that it marries state of the art compiler technologies like MLIR withthe venerable Fortran language and its large community of developers focused onhigh performance compute.
>
> Flang has set the standard for LLVM frontends by adopting MLIR and C++17 featuresearlier than others, and I am thrilled to see Clang and other frontends modernizebased on those experiences.
>
> Flang also continues something very precious to me: the LLVM Project’s abilityto enable collaboration by uniting people with shared interests even if theyspan organizations like academic institutions, companies, and other research groups.

- Chris Lattner, serving member of the LLVM Board of Directors, co-founder ofthe LLVM Project, the Clang C++ compiler and MLIR.

> The need for a more modern Fortran compiler motivated the creation of the LLVM Flangproject and AMD fully supports that path.
>
> In following with community trends, AMD’s Next-Gen Fortran Compiler will be adownstream flavor of LLVM Flang and will in time supplant the current AMD Flangcompiler, a downstream flavor of “Classic Flang”.
>
> Our mission is to allow anyone that is using and developing a Fortran HPC codebaseto directly leverage the power of AMD’s GPUs. AMD’s Next-Gen Fortran Compiler’s goalis fulfilling our vision by allowing you to deploy and accelerate your Fortran codeson AMD GPUs using OpenMP offloading, and to directly interface and invoke HIP andROCm kernels.

- AMD, [“Introducing AMD’s Next-Gen Fortran Compiler”](https://rocm.blogs.amd.com/ecosystems-and-partners/fortran-journey/README.html)

# Getting Involved

Flang might not be new anymore, but it is definitely still improving. If youwant to try Flang on your own projects, you can [download](https://github.com/llvm/llvm-project/releases/tag/llvmorg-20.1.0) it right now.

If you want to contribute, there are many ways to do so. Bug reports,code contributions, documentation improvements and so on. Flang follows the [LLVM contribution process](https://llvm.org/docs/Contributing.html) and youcan find links to the forums, community calls and anything else youmight need [here](https://flang.llvm.org/docs/GettingInvolved.html).

# Credits

Thank you to the following people for their contributions to this article:

- Alex Bradbury (Igalia)
- Andrzej Warzyński (Arm)
- Axel Obermeier (Quansight Labs)
- Brad Richardson (Lawrence Berkeley National Laboratory)
- Carlos Seo (Linaro)
- Daniel C Chen (IBM)
- Eric Schweitz (NVIDIA)
- Hao Jin
- Jeff Hammond (NVIDIA)
- Kiran Chandramohan (Arm)
- Leandro Lupori (Linaro)
- Luis Machado (Arm)
- Mehdi Amini
- Pat McCormick (Los Alamos National Laboratory)
- Peter Waller (Arm)
- Steve Scalpone (NVIDIA)
- Tarun Prabhu (Los Alamos National Laboratory)

# Further reading

- [Learn Fortran](https://fortran-lang.org/learn/)
- [The ’eu’ in eucatastrophe – Why SciPy builds for Python 3.12 on Windows are a minor miracle](https://labs.quansight.org/blog/building-scipy-with-flang)
- [Resurrecting Fortran](https://ondrejcertik.com/blog/2021/03/resurrecting-fortran/)
- [The Fortran Package Manager’s First Birthday](https://everythingfunctional.wordpress.com/2021/03/12/the-fortran-package-managers-first-birthday/)
- [How to write a new compiler driver? The LLVM Flang perspective](https://www.youtube.com/watch?v=OvTiKWfhaho)
- [Flang in the Exascale Supercomputing Project](https://www.exascaleproject.org/research-project/flang/)
