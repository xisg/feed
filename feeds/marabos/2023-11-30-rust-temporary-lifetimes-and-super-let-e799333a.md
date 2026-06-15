---
title: Rust Temporary Lifetimes and "Super Let"
url: https://blog.m-ou.se/super-let/
published: "2023-11-30T00:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/super-let/
---

# Rust Temporary Lifetimes and "Super Let"

The lifetime of temporaries in Rust is a complicated but often ignored topic.
In simple cases, Rust keeps temporaries around for exactly long enough,
such that we don’t have to think about them.
However, there are plenty of cases were we might not get exactly what we want, right away.

In this post, we (re)discover the rules for the lifetime of temporaries,
go over a few use cases for _temporary lifetime extension_,
and explore a new language idea, `super let`, to give us more control.
