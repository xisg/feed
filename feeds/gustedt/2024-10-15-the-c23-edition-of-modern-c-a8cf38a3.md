---
title: The C23 edition of Modern C
url: https://gustedt.wordpress.com/2024/10/15/the-c23-edition-of-modern-c/
published: "2024-10-15T15:38:17Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=4215
---

# The C23 edition of Modern C

The C23 edition of Modern C is now available for free download from

[https://hal.inria.fr/hal-02383654](https://hal.inria.fr/hal-02383654)

And as before a dedicated page for the book may be found at

[https://gustedt.gitlabpages.inria.fr/modern-c/](https://gustedt.gitlabpages.inria.fr/modern-c/)

where you also may find a link to download the code examples that come with the book.

This new edition has been the occasion to overhaul the presentation in many places, but its main purpose is the update to the new C standard, [C23](https://www.iso.org/standard/82075.html). The goal was to publish this new edition of Modern C at the same time as the new C standard goes through the procedure of ISO publication. The closest approximation of the contents of the new standard in a publically available document can be found [here](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3220.pdf). New releases of major compilers already implement most of the new features that it brings.

Among the most noticeable changes and additions that we handle are those for integers: there are new bit-precise types coined `_BitInt(N)`, new C library headers `(for arithmetic with overflow check) and` (for bit manipulation), possibilities for 128 bit types on modern architectures, and substantial improvements for enumeration types. Other new concepts in C23 include a `nullptr` constant and its underlying type, syntactic annotation with attributes, more tools for type generic programming such as type inference with `auto` and `typeof`, default initialization with `{}`, even for variable length arrays, and `constexpr` for named constants of any type. Furthermore, new material has been added, discussing compound expressions and lambdas, so-called “internationalization”, a comprehensive approach for program failure.

Also added has been an appendix and a temporary include header for an easy transition to C23 on existing platforms, that will allow you to start off with C23 right away.

Manning’s early access program (MEAP) for the new edition is still open at

[https://www.manning.com/books/modern-c-third-edition](https://www.manning.com/books/modern-c-third-edition)

Unfortunately they were not yet able to tell me when their version of the C23 edition will finally be published.
