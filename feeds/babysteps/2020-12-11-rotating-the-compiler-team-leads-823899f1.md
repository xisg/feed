---
title: Rotating the compiler team leads
url: https://smallcultfollowing.com/babysteps/blog/2020/12/11/rotating-the-compiler-team-leads/?utm_source=atom_feed
published: "2020-12-11T00:00:00Z"
feed: babysteps
guid: https://smallcultfollowing.com/babysteps/blog/2020/12/11/rotating-the-compiler-team-leads/
---

# Rotating the compiler team leads

Since we created the Rust teams, I have been serving as lead of two teams: the [compiler team](https://www.rust-lang.org/governance/teams/compiler) and the [language design team](https://www.rust-lang.org/governance/teams/lang) (I’ve also been a member of the [core team](https://www.rust-lang.org/governance/teams/core), which has no lead). For those less familiar with Rust’s governance, the compiler team is focused on the maintenance and implementation of the compiler itself (and, more recently, the standard library). The language design team is focused on the design aspects. Over that time, all the Rust teams have grown and evolved, with the compiler team in particular being home to a number of really strong members.

Last October, [I announced that pnkfelix was joining me as compiler team co-lead](https://blog.rust-lang.org/inside-rust/2019/10/24/pnkfelix-compiler-team-co-lead.html). Today, I am stepping back from my role as compiler team co-lead altogether. After taking nominations from the compiler team, pnkfelix and I are proud to announce that **[wesleywiser](https://github.com/wesleywiser) will replace me as compiler team co-lead**. If you don’t know Wesley, there’ll be an announcement on [Inside Rust](https://blog.rust-lang.org/inside-rust/) where you can learn a bit more about what he has done, but let me just say I am pleased as punch that he agreed to serve as co-lead. He’s going to do a great job.

### You’re not getting rid of me this easily

Stepping back as compiler team co-lead does not mean I plan to step away from the compiler. In fact, quite the opposite. I’m still quite enthusiastic about pushing forward on ongoing implementaton efforts like the work to implement [RFC 2229](https://github.com/rust-lang/rust/issues/53488), or the development on [chalk](https://github.com/rust-lang/chalk) and [polonius](https://github.com/rust-lang/polonius). In fact, I am hopeful that stepping back as co-lead will create more time for these efforts, as well as time to focus on leadership of the language design team.

### Rotation is key

I see these changes to compiler team co-leads as fitting into a larger trend, one that I believe is going to be increasingly important in Rust: **rotation of leadership**. To me, the “corest of the core” value of the Rust project is the importance of [“learning from others”](https://github.com/rust-lang/foundation-faq-2020/blob/main/FAQ.md#q-sharing-experience) – or as I put it in [my rust-latam talk from 2019](https://nikomatsakis.github.io/rust-latam-2019/#94) [1](#fn:1), “a commitment to a CoC and a culture that emphasizes curiosity and deep research”. **Part of learning from others has to be actively seeking out fresh leadership and promoting them into positions of authority.**

### But rotation has a cost too

Another core value of Rust is [recognizing the inevitability of tradeoffs](https://smallcultfollowing.com/babysteps/
/blog/2019/04/19/aic-adventures-in-consensus/) [2](#fn:2). Rotating leadership is no exception: there is a lot of value in having the same people lead for a long time, as they accumulate all kinds of context and skills. But it also means that you are missing out on the fresh energy and ideas that other people can bring to the problem. I feel confident that Felix and Wesley will help to shape the compiler team in ways that I never would’ve thought to do.

### Rotation with intention

The tradeoff between experience and enthusiasm makes it all the more important, in my opinion, to rotate leadership intentionally. I am reminded of [Emily Dunham’s classic post on leaving a team](http://edunham.net/2018/05/15/team.html) [3](#fn:3), and how it was aimed at normalizing the idea of “retirement” from a team as something you could actively choose to do, rather than just waiting until you are too burned out to continue.

Wesley, Felix, and I have discussed the idea of “staggered terms” as co-leads. The idea is that you serve as co-lead for two years, but we select one new co-lead per year, with the oldest co-lead stepping back. This way, at every point you have a mix of a new co-lead and someone who has already done it for one year and has some experience.

### Lang and compiler need separate leadership

Beyond rotation, another reason I would like to step back from being co-lead of the compiler team is that I don’t really think it makes sense to have one person lead two teams. It’s too much work to do both jobs well, for one thing, but I also think it works to the detriment of the teams. I think the compiler and lang team will work better if they each have their own, separate “advocates”.

I’m actually very curious to work with pnkfelix and Wesley to talk about how the teams ought to coordinate, since I’ve always felt we could do a better job. I would like us to be actively coordinating how we are going to manage the implementation work at the same time as we do the design, to help avoid [unbounded queues](https://smallcultfollowing.com/babysteps/
/blog/2019/07/10/aic-unbounded-queues-and-lang-design/). I would also like us to be doing a better job getting feedback from the implementation and experimentation stage into the lang team.

You might think having me be the lead of both teams would enable coordination, but I think it can have the opposite effect. Having separate leads for compiler and lang means that those leads must actively communicate and avoids the problem of one person just holding things in their head without realizing other people don’t share that context.

### Idea: Deliberate team structures that enable rotation

In terms of the compiler team structure, I think there is room for us to introduce “rotation” as a concept in other ways as well. Recently, I’ve been [kicking around an idea for “compiler team officers”](https://zulip-archive.rust-lang.org/185694tcompilerwgmeta/79956compilerteamofficers.html) [4](#fn:4), which would introduce a number of defined roles, each of which is setup in with staggered terms to allow for structured handoff. I don’t think the current proposal is quite right, but I think it’s going in an intriguing direction.

This proposal is trying to address the fact that a successful open source organization needs [more than coders](https://smallcultfollowing.com/babysteps/
/blog/2019/04/15/more-than-coders/), but all too often we fail to recognize and honor that work. Having fixed terms is important because when someone _is_ willing to do that work, they can easily wind up getting stuck being the only one doing it, and they do that until they burn out. The proposal also aims to enable more “part-time” leadership within the compiler team, by making “finer grained” duties that don’t require as much time to complete.

* * *

1. Oh-so-subtle plug: I really quite liked that talk. [↩︎](#fnref:1)

2. Though not always the tradeoffs you expect. [Read the post.](https://smallcultfollowing.com/babysteps/
/blog/2019/04/19/aic-adventures-in-consensus/) [↩︎](#fnref:2)

3. If you haven’t read it, stop reading now and [go do so](http://edunham.net/2018/05/15/team.html). Then come back. Or don’t. [Just read it already.](http://edunham.net/2018/05/15/team.html) [↩︎](#fnref:3)

4. I am not sure that ‘officer’ is the right word here, but I’m not sure what the best replacement is. I want something that conveys respect and responsibility. [↩︎](#fnref:4)
