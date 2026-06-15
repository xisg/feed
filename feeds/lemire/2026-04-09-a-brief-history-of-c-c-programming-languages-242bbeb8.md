---
title: A brief history of C/C++ programming languages
url: https://lemire.me/blog/2026/04/09/a-brief-history-of-c-c-programming-languages/
published: "2026-04-09T14:58:53Z"
feed: lemire
guid: https://lemire.me/blog/?p=22574
---

# A brief history of C/C++ programming languages

![](https://lemire.me/blog/wp-content/uploads/2026/04/Capture-decran-le-2026-04-09-a-10.54.35-150x150.png)

Initially, we had languages like Fortran (1957), Pascal (1970), and C (1972). Fortran was designed for number crunching and scientific computing. Pascal was restrictive with respect to low-level access (it was deliberately “safe”, as meant for teaching structured programming). So C won out as a language that allowed low-level/unsafe programming (pointer arithmetic, direct memory access) while remaining general-purpose enough for systems work like Unix. To be fair, Pascal had descendants that are still around, but C clearly dominated.

Object-oriented programming became viewed as the future in the 1980s and 1990s. It turned into some kind of sect.

But C was not object-oriented.

So we got C++, which began as “C with Classes”. C++ had templates, enabling generic programming and compile-time metaprogramming. This part of the language makes C++ quite powerful, but somewhat difficult to master (with crazy error messages).

Both C and C++ became wildly successful, but writing portable applications remained difficult — you often had to target Windows or a specific Unix variant. This was a problem for a company like Sun Microsystems that sold Unix boxes and wanted to compete against the juggernaut that Microsoft was becoming.

So Java came along in 1995. It was positioned as a safe, portable alternative to C++: it eliminated raw pointer arithmetic, added mandatory garbage collection, array bounds checking everywhere, and ran on a virtual machine (JVM) with just-in-time compilation for performance.

The “write once, run anywhere” promise addressed C/C++ portability pain points directly. To this day, Java remains a strong solution for writing portable enterprise and server-side code.

We also got JavaScript in 1995. Despite the name, it has almost nothing in common with Java semantically. It is best viewed as separate from the C/C++ branch. Python is similarly quite different.

Microsoft would eventually come up with C# in 2000. It belongs to the same C-family syntax tradition as C++ and Java, but with support for ahead-of-time compilation in modern .NET. It also allows guarded pointer access within explicitly marked unsafe scopes. At this point, C# can be seen as “C++ with garbage collection” in spirit. It even competes against C++ in the game industry thanks to Unity.

Google came up with Go. It is much like a simpler, modern C: garbage-collected, with built-in bounds checking on slices/arrays, and pointers allowed but without arbitrary arithmetic in safe code (the unsafe package exists for low-level needs).

Later, Apple came up with Swift. It has C++-like performance and syntax goals but adds modern safety features (bounds checking by default, integer overflow panics in debug mode) and uses Automatic Reference Counting (ARC) for memory management. Swift replaced Objective-C but I still view it as a C++ successor.

At about the same time, we got Rust. Like Swift, it drops the generational garbage collection from Java, C# and Go. It relies instead on compile-time ownership and borrowing rules, with the tradeoff that you can leak memory with reference cycles. We also got Zig  which makes memory usage fully explicit.

I think that it is fairer to describe Rust and Zig as descendants of C rather than C++. Both are much more powerful than C, of course… and the evolution of programming languages is complex. Still. They are C-like programming languages.

To this day, in much of the industry, the dominant programming languages for performance-critical, systems, enterprise, and infrastructure work remain C, C++, Java, and C#. By the Lindy effect (the longer something has survived, the longer it is likely to continue surviving), these languages, especially C, now over 50 years old, are still going to be around for a long time.
