---
title: Writing type parametric functions in Go
url: https://burntsushi.net/type-parametric-functions-golang/
published: "2013-04-06T17:54:00Z"
feed: burntsushi
guid: https://burntsushi.net/type-parametric-functions-golang/
---

# Writing type parametric functions in Go

Go’s only method of compile time safe polymorphism is structural subtyping, and
this article will do nothing to change that. Instead, I’m going to present a
package `ty` with facilities to write type parametric functions in Go that
maintain **run time** type safety, while also being convenient for the
caller to use.
