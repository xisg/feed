---
title: Failure 0.1.1 released
url: https://without.boats/blog/failure-0-1-1/
published: "2017-11-30T00:00:00Z"
feed: boats
guid: https://without.boats/blog/failure-0-1-1/
---

# Failure 0.1.1 released

I’ve just published failure 0.1.1 to crates.io. It’s mostly some incremental improvements to failure that have been suggested since the first release two weeks ago.
Improvements to the derive A big change in version 0.1.1 is that the derive can be used without depending on the failure\_derive crate separately. All that needs to be done is tagging the extern crate statement with #\[macro\_use\]:
// No direct dependency on \`failure\_derive\` #\[macro\_use\] extern crate failure; #\[derive(Fail, Debug)\] #\[fail(display = "An error occurred.
