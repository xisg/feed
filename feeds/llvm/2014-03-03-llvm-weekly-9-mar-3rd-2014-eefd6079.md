---
title: 'LLVM Weekly - #9, Mar 3rd 2014'
url: https://blog.llvm.org/2014/03/llvm-weekly-9-mar-3rd-2014.html
published: "2014-03-03T05:43:00Z"
feed: llvm
guid: https://blog.llvm.org/2014/03/llvm-weekly-9-mar-3rd-2014.html
---

# LLVM Weekly - #9, Mar 3rd 2014

Welcome to the ninth issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

As well as growing another year older last week, I've also started publicising the book I authored with [Ben Everard](https://twitter.com/ben_everard), [Learning Python with Raspberry Pi](http://www.amazon.co.uk/Learning-Python-Raspberry-Alex-Bradbury/dp/1118717058) ( [Amazon US](http://www.amazon.com/Learning-Python-Raspberry-Alex-Bradbury/dp/1118717058/)) which should ship soon in paperback or is available right now for Kindle. Hopefully it should be available soon in DRM-free digital formats on oreilly.com. I will be putting more of my Raspberry Pi exploits and tutorials on [muxup.com](http://muxup.com), so if that interests you follow [@muxup](https://twitter.com/muxup).

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/9).

### News and articles from around the web

The list of [mentoring organisations for Google Summer of Code 2014](https://www.google-melange.com/gsoc/org/list/public/google/gsoc2014) has been released. LLVM is one of them, so any budding compiler engineers who qualify may want to check out the [ideas page](http://llvm.org/OpenProjects.html). Other organisations I spotted advertising relevant project ideas are [the Linux Foundation](http://llvm.linuxfoundation.org/index.php/GSoC), [X.org](http://www.x.org/wiki/SummerOfCodeIdeas/) and of course [GCC](http://gcc.gnu.org/wiki/SummerOfCode).

At the end of last week, Broadcom made a major step forward in [announcing the release of full register level documentation for the VideoCore IV graphics engine](http://blog.broadcom.com/chip-design/android-for-all-broadcom-gives-developers-keys-to-the-videocore-kingdom/) as well as full graphics driver source. The device most well-known for featuring VideoCore IV is the [Raspberry Pi](http://www.raspberrypi.org/archives/6299). The released documentation opens the door to producing something similar to the [GPU-accelerated FFT library](http://www.raspberrypi.org/archives/5934) support that was recently released. Some readers of LLVM Weekly may of course be interested in using this information to produce an LLVM backend. Hopefully the following pointers will help. There are lots of resources linked to at the homepage of the [VideoCore IV reverse engineering project](https://github.com/hermanhermitage/videocoreiv). I'd draw particular attention to the [QPU reverse engineering effort](https://github.com/hermanhermitage/videocoreiv-qpu) which contains good information despite the reverse engineering part of the work being made unnecessary by the Broadcom release. You may want to check out the [raspi-internals mailing list](http://www.freelists.org/archive/raspi-internals/) and `#raspberrypi-internals` on Freenode. It's also worth looking at [the commented disassembly of the VideoCore FFT code](https://github.com/hermanhermitage/videocoreiv-qpu/tree/master/qpu-fft) and Herman Hermitage's work in progress [QPU tutorial](https://github.com/hermanhermitage/videocoreiv-qpu/tree/master/qpu-tutorial).

Code for [Fracture](https://github.com/draperlaboratory/fracture), an architecture-independent decompiler to LLVM IR has been released.

Olivier Goffart has written about [his proof of concept reimplementation of Qt's moc using libclang](http://woboq.com/blog/moc-with-clang.html). It's actually from last year, but it's new to me.

Alex Denisov has written a [guide to writing a clang plugin](http://railsware.com/blog/2014/02/28/creation-and-using-clang-plugin-with-xcode/). He gives an example of a minimal plugin that complains about lowercased Objective C class names.

Coursera are re-running [their compilers course](https://www.coursera.org/course/compilers) on March the 17th. See [Dirkjan Ochtman's impressions of the course from the previous run](http://dirkjan.ochtman.nl/writing/2012/07/21/compilers-on-coursera.html).

The Qualcomm LLVM team are [advertising for an intern](http://article.gmane.org/gmane.comp.compilers.llvm.devel/70655).

### On the mailing lists

- Denis Steckelmacher [kicked off a discussion about the suggested GSoC project to use LLVM for code generation](http://article.gmane.org/gmane.comp.compilers.llvm.devel/70671). There are particularly interesting replies from [Kirill Batuzov](http://article.gmane.org/gmane.comp.debugging.valgrind.devel/26122) and [Julian Seward](http://article.gmane.org/gmane.comp.debugging.valgrind.devel/26124) who elaborate on why it's not a case of 'just use LLVM and it will be fast' as well as suggesting some changes that could be made to increase Valgrind's speed.

- Chandler Carruth [announces he has flipped the switch for C++11 support on the build systems](http://article.gmane.org/gmane.comp.compilers.llvm.devel/70836). I, for one, welcome our new C++11 overloards.

- For those interested in the issues of supporting precise GC on LLVM, see the discussion surrounding Philip Reames' [question on representing a safepoint](http://article.gmane.org/gmane.comp.compilers.llvm.devel/70724).

- Alp Toker [proposed a way to move forward with the LLVM OpenMP runtime](http://article.gmane.org/gmane.comp.compilers.llvm.devel/70714). The main issue right now is there is no real test suite. Alp's suggestion is to organise a few days effort to adapt the libgomp test suite (to be hosted externally, due to licensing reasons).

- Jasper Neumann [suggested a cunning optimisation](http://article.gmane.org/gmane.comp.compilers.llvm.devel/70789) for checking the modulus. Benjamin Kramer explains the difficulty of integrating such an optimisation in to the peephole optimiser.

- Nico Weber [suggests a Clang warning that fires on unexpected indentation levels](http://article.gmane.org/gmane.comp.compilers.clang.devel/35267). He has [prototype implementation](http://llvm.org/bugs/show_bug.cgi?id=18938) up on the bugtracker.

- Saleem Abdulrasool has [suggested adding a macro that indicates the LLVM integrated assembler is being used](http://article.gmane.org/gmane.comp.compilers.clang.devel/35268). Respondents to the thread so far aren't keen on the idea, pointing out that projects should really check whether a certain syntax or feature they need works rather than marking themselves incompatible with the LLVM integrated assembler which may of course improve compatibility in the future.


### LLVM commits

- LLVM grew a big-endian AArch64 target [r202024](http://reviews.llvm.org/rL202024). Some might consider it a step back, but apparently there's a decent number of people interested in big-endian on AArch64. There's an interesting [presentation from ARM](http://www.linux-kvm.org/wiki/images/c/c4/Kvm-forum-2013-crossing-the-endianness-bridge.pdf) about running a virtualised BE guest on a LE host.

- The flipping of the C++11 switch has allowed a number of simplifications to start to make their way in to the LLVM codebase. For instance, turning simple functors in to lambdas. Like or loathe C++11 lambda syntax, they're certainly less verbose. [r202588](http://reviews.llvm.org/rL202588). `OwningPtr<T>` gained support for being converted to and from `std::unique_ptr<T>`, which lays the ground for LLVM moving to using `std::unique_ptr` in the future. [r202609](http://reviews.llvm.org/rL202609).

- The coding standards document was updated to reflect the C++11 features that can now be used in the LLVM/Clang codebase and to provide guidance on their use. [r202497](http://reviews.llvm.org/rL202497), [r202620](http://reviews.llvm.org/rL202620).

- The loop vectorizer is now included in the LTO optimisation pipeline by default. [r202051](http://reviews.llvm.org/rL202051).

- DataLayout has been converted to be a plain object rather than a pass. A DataLayoutPass which holds a DataLayout has been introduced. [r202168](http://reviews.llvm.org/rL202168).

- The PowerPC backend learned to track condition register bits, which produced measurable speedups (10-35%) for the POWER7 benchmark suite. [r202451](http://reviews.llvm.org/rL202451).

- X86 SSE-related instructions gained a scheduling model. Sadly there is no indication whether this makes any measurable difference to common benchmarks. [r202065](http://reviews.llvm.org/rL202065).

- The scalar replacement of aggregates pass (SROA) got a number of refactorings and bug fixes from Chandler Carruth, including some bug fixes for handling pointers from address spaces other than the default. [r202092](http://reviews.llvm.org/rL202092), [r202247](http://reviews.llvm.org/rL202247), and more.

- An experimental implementation of an invalid-pointer-pair detector was added as part of AddressSanitizer. This attempts to identify when two unrelated pointers are compared or subtracted. [r202389](http://reviews.llvm.org/rL202389).

- Shed a tear, for libtool has been removed from the LLVM build system. The commit says it was only being used to find the shared library extension and nm. The diffstat of 93 insertions and 35277 deletions speaks for itself. [r202524](http://reviews.llvm.org/rL202524).


### Clang commits

- The initial changes needed for `omp simd` directive support were landed. [r202360](http://reviews.llvm.org/rL202360).

- The `-Wabsolute-value` warning was committed, which will warn for several cases of misuse of absolute value functions. It will warn when using e.g. an int absolute value function on a float, or when using it one a type of the wrong size (e.g. using abs rather than llabs on a long long), or whn taking the absolute value of an unsigned value. [r202211](http://reviews.llvm.org/rL202211).

- An API was added to libclang to create a buffer with a JSON virtual file overlay description. [r202105](http://reviews.llvm.org/rL202105).

- The driver option `-ivfsoverlay` was added, which reads the description of a virtual filesystem from a file and overlays it over the real file system. [r202176](http://reviews.llvm.org/rL202176).

- CFG edges have been reworked to encode potentially unreachable edges. This involved adding the AdjacentBlock class this encodes whether the block is reachable or not. [r202325](http://reviews.llvm.org/rL202325).

- The 'remark' diagnostic type was added. This provides additional information to the user (e.g. information from the vectorizer about loops that have been vectorized). [r202475](http://reviews.llvm.org/rL202475).


### Other project commits

- The compiler-rt subproject now has a `CODE_OWNERS.txt` to indicate who is primarily responsible for each part of the project. [r202377](http://reviews.llvm.org/rL202377).

- A standalone deadlock detector was added to ThreadSanitizer. [r202505](http://reviews.llvm.org/rL202505).

- The OpenMP runtime has been ported to FreeBSD. [r202478](http://reviews.llvm.org/rL202478).
