---
title: Use of Formal Methods at Amazon Web Services
url: http://brooker.co.za/blog/2014/08/09/formal-methods.html
published: "2014-08-09T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2014/08/09/formal-methods
---

# Use of Formal Methods at Amazon Web Services

How we're using TLA+ at AWS

Late last year, we published [Use of Formal Methods at Amazon Web Services](http://research.microsoft.com/en-us/um/people/lamport/tla/formal-methods-amazon.pdf) about our experiences with using formal methods at Amazon Web Services (AWS). The focus is on [TLA+](http://research.microsoft.com/en-us/um/people/lamport/tla/tla.html), and why we think it’s a great fit for the kind of work we do.

From the paper:

> In order to find subtle bugs in a system design, it is necessary to have a precise description of that design. There are at least two major benefits to writing a precise design; the author is forced to think more clearly, which helps eliminate ‘plausible hand-waving’, and tools can be applied to check for errors in
> the design, even while it is being written. In contrast, conventional design documents consist of prose, static diagrams, and perhaps pseudo-code in an adhoc untestable language. Such descriptions are far from precise; they are often ambiguous, or omit critical aspects such as partial failure or the granularity of concurrency (i.e. which constructs are assumed to be atomic). At the other end of the spectrum, the final executable code is unambiguous, but contains an overwhelming amount of detail. We needed to be able to capture the essence of a design in a few hundred lines of precise description.

The [full paper](http://research.microsoft.com/en-us/um/people/lamport/tla/formal-methods-amazon.pdf) is worth reading if you’re interested in formal methods.
