---
title: 'C++11: Rvalue references for *this'
url: https://blog.m-ou.se/rvalue-references-for-this/
published: "2013-07-21T18:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/rvalue-references-for-this/
---

# C++11: Rvalue references for *this

Recently, gcc added support rvalue references for `*this`.
(Clang has supported it for quite a while now.)
In this post, I show how to use this feature, and how it means we can finally
define accessors and a few other things like `operator=` correctly.
