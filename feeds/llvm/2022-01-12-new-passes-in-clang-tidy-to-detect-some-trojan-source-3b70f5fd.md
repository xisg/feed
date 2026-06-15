---
title: New passes in clang-tidy to detect (some) Trojan Source
url: https://blog.llvm.org/posts/2022-01-12-trojan-source/
published: "2022-01-12T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2022-01-12-trojan-source/
---

# New passes in clang-tidy to detect (some) Trojan Source

The original [Trojan Source paper](https://trojansource.codes/) encompasses afamily of attacks that rely on Unicode properties to make code _look_ differentfrom how a compiler _processes_ it. For instance the following code taken fromthe paper:

```c
#include <stdio.h>#include <stdbool.h>int main() { bool isAdmin = false; /*‮ } ⁦if (isAdmin)⁩ ⁦begin admins only */ printf("You are an admin.\n"); /* end admins only ‮ { ⁦*/ return 0;}
```

looks like there is a guard on `isAdmin` while the compiler actually reads thefollowing byte stream

```
/* <U+0x202E> } <U+0x2066> if (isAdmin) <U+0x2069> <U+0x2066> begin admins only */
```

This issue got submitted before the official release to the LLVM Security group,and while we agreed this was more of a display issue than an actualcompiler-related issue, we also agreed having a `clang-tidy` check for eachflaws described in the paper could not hurt.

# Using clang-tidy

The tool named `clang-tidy` can run a bunch of extra passes on a codebase,detecting coding convention issues, API misuses, security flaws etc. We havebeen adding three new checkers:

## Detecting misleading bidirectional characters

The new check `misc-misleading-bidirectional` parses each comment and stringliteral from the codebase and looks for unterminated bidirectional sequence,i.e. sequence that leak past the end of comment or string literal, makingregular code being displayed right-to-left instead of the usual left-to-right.In the case of the example above we get a warning close to:

```
5:3: warning: comment contains misleading bidirectional Unicode characters [misc-misleading-bidirectional]
```

## Detecting misleading identifiers

C++ allows for some Unicode codepoints within identifiers, including identifiersthat have a strong right-to-left direction, which can lead to misleadingstatements. For instance in the following,

```c++
int א = ג;
```

Are we assigining to `א` or to `ג`? We are actually doing the latter, andthat is confusing. The pass `misc-misleading-identifier` detect thatconfiguration and outputs a warning similar to

```
10:3: warning: identifier has right-to-left codepoints [misc-misleading-identifier]
```

## Detecting confusing identifiers

Who never received a spam using unicode characters that look alike asciicharacters to bypass some hypothetical anti-spam scanning? C like language donot escape the trend, and it is perfeclty valid and confusing to define

```c++
int foo;
```

at some point of the program, and

```c++
int 𝐟oo;
```

elsewhere. The `misc-homoglyph` checker detects such confusable identifiers(a.k.a. _homoglyph_) based on a list of [confusables.txt](https://www.unicode.org/Public/security/latest/confusables.txt) maintained by the Unicode consortium. In the case above, one would get a warningsimilar to

```
7:5: warning: 𝐟oo is confusable with foo [misc-homoglyph]
```

# Concluding Words

As described in this post, we chose to implement Trojan Source counter-measureas several `clang-tidy` checkers. Doing so instead of implementing them as Clang warningis a trade-off on parse time.

The interested reader can discover the alternative GCC aproach in this [dedicated blogpost](https://developers.redhat.com/articles/2022/01/12/prevent-trojan-source-attacks-gcc-12)!
