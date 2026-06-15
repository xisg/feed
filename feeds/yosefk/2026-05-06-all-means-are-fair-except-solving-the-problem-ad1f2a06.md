---
title: All means are fair except solving the problem
url: https://yosefk.com/blog/all-means-are-fair-except-solving-the-problem.html
published: "2026-05-06T00:00:00Z"
feed: yosefk
---

# All means are fair except solving the problem

An industry veteran in my circles has recently made the rookie mistake [1](#fn1) of printing a warning from his code upon misuse. Surprisingly to nobody experienced,
critical workflows soon came to a screeching halt.

It turned out that a program using his code prints something like “yay, done” upon exit, and scripts expect it to be the last
thing it says. But now those warnings occasionally got printed from destructors or such, _after_ the “yay, done”, making
the scripts think the program failed.

One might think that this prompted people to fix the reported misuse, and that thought would be another rookie mistake.
Instead, they were quick to point out that it’s hard to know where these warnings could come from, and we cannot risk all those
critical workflows failing when some case of misuse surfaces in a new context.

I mean, you could grep to get an upper bound, and if you did, not that many places would come up. But one could then say, as
some in fact _did_, that maybe you haven’t grepped everywhere you should have, and even the cases you did find are owned
by many different teams, so we won’t get the fixes quickly enough, etc.

Several solutions were suggested by helpful high-ranking people:

- You could add a destructor printing “yay, done” _again_ if a warning was printed during the destruction sequence
  (opening an interesting technical debate about the differences between a destructor, \_\_attribute\_\_((destructor)), an atexit
  handler and other unspeakable horrors). In fact, our industry veteran would later learn, and I swear that I’m not making this
  up, that _this was already implemented by someone else_ who printed something during the program termination sequence,
  and had to appease the scripts.
- You could suppress the warnings by default, and enable them upon request (opening a debate about the runtime method to
  enable them, and the appropriate circumstances to do this).
- You could write those warnings to their own file, and…

When I was done scrolling his work chat with these helpful suggestions, our unfortunate industry veteran put on a melancholy
smile and summarized the situation: “All means are fair except solving the problem.” [2](#fn2)

* * *

1. Our protagonist happens to be somewhat of an idealist, and since his condition is too acute to be treated by
   experience, he’s bound to make what pragmatists call “rookie mistakes.” But this particular story could happen to most of us. [↩︎](#fnref1)

2. [Hyrum’s law](https://www.hyrumslaw.com/) arguably diagnoses this particular problem more
   specifically from a technical point of view. However, our melancholy veteran’s phrase hints at the broader social condition from
   which the technical problem derives its significance. And by “social condition,” I mean that in Hyrum’s law, “all observable
   behaviors of your system will be depended on by somebody” is implicitly amended with “...somebody who can’t be bothered to fix
   their code, and there’s nothing you can do about it” — and it’s this quiet part which makes it into a “law.” [↩︎](#fnref2)
