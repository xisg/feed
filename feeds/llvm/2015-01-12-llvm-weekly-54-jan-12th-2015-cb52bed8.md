---
title: 'LLVM Weekly - #54, Jan 12th 2015'
url: https://blog.llvm.org/2015/01/llvm-weekly-54-jan-12th-2015.html
published: "2015-01-12T18:10:00Z"
feed: llvm
guid: https://blog.llvm.org/2015/01/llvm-weekly-54-jan-12th-2015.html
---

# LLVM Weekly - #54, Jan 12th 2015

Welcome to the fifty-fourth issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

As you receive this week's issue, I should be on my way to California where I'll be presenting [lowRISC](http://www.lowrisc.org) at the RISC-V workshop in Monterey and having a few mother meetings. I'm in SF Fri-Sun and somewhat free on the Saturday if anyone wants to meet and chat LLVM or lowRISC/RISC-V.

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/54).

### News and articles from around the web

Euro LLVM 2015 will be held on April 13th-14th in London, UK. The [call for papers](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80291) is now open with a deadline of 16th Feb.

Talks for the [LLVM devroom at FOSDEM](https://fosdem.org/2015/schedule/track/llvm_toolchain/) have been announced. The LLVM devroom is on Sunday 1st Feb. Readers will be pleased to know this doesn't clash with [my talk on lowRISC](https://fosdem.org/2015/schedule/event/lowrisc/) which is on the Saturday.

Google now use [Clang for production Chrome builds on Linux](http://blog.llvm.org/2015/01/using-clang-for-chrome-production.html). They were previously using GCC 4.6. Compared to that baseline, performance stayed roughly the same while binary size decreased by 8%. It would certainly have been interesting to compare to a more recent GCC baseline. The blog post indicates they're hopeful to use Clang in the future for building Chrome for Windows.

Philip Reames did an interesting back of the envelope calculation about the [cost of maintaining LLVM](http://www.philipreames.com/Blog/2015/01/10/how-much-does-it-cost-to-maintain-llvm/). He picked out commits which seems like they could be trivially automated and guesstimated a cost based on developer time. The figure he arrives at is $1400 per month.

The next LLVM social for Cambridge, UK will be [on Wed 21st Jan at 7:30pm](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80341).

### On the mailing lists

- LLVM 3.6 will be [branching soon](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80447), on 14th January.

- Philip Reames asks [whether address space 1 is reserved on any architecture](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80355). It seems the answer is no, though the thread resulted in some discussion on the use of address spaces and the ability to reserve some. Philip [had a strawman proposal](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80370) for meanings of different address space numbers.

- Chandler Carruth has suggested [new IR features are needed to represent the cases global metadata is currently used for](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80430). Metadata was intended to be used to hold information that can be safely dropped, though this isn't true for e.g. module flags.

- Arch Robinson kicked off a discussion about [floating point range checks in LLVM](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80381). This isn't currently supported, though there's agreement it could be useful as well as a fair amount of discussion on some of the expected subtleties.

- If you're wondering about alias instructions, Bruce Hoult has a [good explanation](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80402).

- Note to out-of-tree backend maintainers, [get/setLoadExtAction now takes another parameter](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80385).

- Right now, LLDB will compile entered expressions in C++ mode. As [noted on the lldb mailing list](http://article.gmane.org/gmane.comp.debugging.lldb.devel/6141) this can be problematic when e.g. debugging a C function which has a local variable called 'this'. Greg Clayton [points out how helpful supporting C++ expressions can be](http://article.gmane.org/gmane.comp.debugging.lldb.devel/6142), even when debugging C code.

- A thread about the [design of the new pass manager](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80418) has been revived. Both Chandler Carruth and Philip Reames suggest [BasicBlockPasses should die](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80424).

- Philip Reames is seeking feedback on [a transformation which would convert a loop to a loop nest](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80386) if it contains infrequently executed slow paths. There's some interesting discussion in the thread, and it's also worth reading Duncan P.N. Exon Smith's [clarification of branch weight, branch probability, block frequency, and block bias](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80472).


### LLVM commits

- An option `hoist-cheap-insts` has been added to the machine loop invariant code motion pass to enable hosting even cheap instructions (as long as register pressure is low). This is disabled by default. [r225470](http://reviews.llvm.org/rL225470).

- The calculation of the unrolled loop size has been fixed. Targets may want to re-tune their default threshold. [r225565](http://reviews.llvm.org/rL225565), [r225566](http://reviews.llvm.org/rL225566).

- DIE.h (datastructures for DWARF info entries) is now a public CodeGen header rather than being private to the AsmPrinter implementation. dsymutil will make use of it. [r225208](http://reviews.llvm.org/rL225208).

- The new pass manager now has a handy utility for generating a no-op pass that forces a usually lazy analysis to be run. [r225236](http://reviews.llvm.org/rL225236).

- There's been a minor change to the .ll syntax for comdats. [r225302](http://reviews.llvm.org/rL225302).

- There have been some minor improvements to the emacs packages for LLVM and tablegen mode. [r225356](http://reviews.llvm.org/rL225356).

- An example GCStrategy using the new statepoint infrastructure has been added. [r225365](http://reviews.llvm.org/rL225365), [r225366](http://reviews.llvm.org/rL225366).


### Clang commits

- A `Wself-move` warning has been introduced. Similar to `-Wself-assign`, it will warn you when your code tries to move a value to itself. [r225581](http://reviews.llvm.org/rL225581).

- The I, J, K, M, N, O inline assembly constraints are now checked. [r225244](http://reviews.llvm.org/rL225244).


### Other project commits

- The libcxx test infrastructure has been refactored into separate modules. [r225532](http://reviews.llvm.org/rL225532).

- The effort to retire InputElement in lld continues. Linker script files are no longer represented as an InputElement. [r225330](http://reviews.llvm.org/rL225330).

- Polly has gained a [changelog](http://polly.llvm.org/changelog.html) in preparation of the next release. [r225264](http://reviews.llvm.org/rL225264).

- Polly has also gained a [TODO list](http://polly.llvm.org/todo.html) for its next phase of development. [r225388](http://reviews.llvm.org/rL225388).
