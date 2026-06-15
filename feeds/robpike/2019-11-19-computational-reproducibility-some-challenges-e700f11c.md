---
title: 'Computational reproducibility: Some challenges'
url: https://commandcenter.blogspot.com/2019/11/computational-reproducibility-some.html
published: "2019-11-19T02:16:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-3423162001051780475
---

# Computational reproducibility: Some challenges

There has been recent discussion about the [reproducibility](https://www.nature.com/collections/prbfkwmwvz) of scientific results, with some unflattering conclusions. One [study](https://www.insidehighered.com/news/2018/08/30/study-raises-new-questions-about-reproducibility-research) suggests only a 62% reproducibility rate.

For some fields it's probably much worse. Any result that depends on computation is at great risk of continual change in its programming environment. A program written ten years ago has little chance of building today without change, let alone running or running correctly.

This concern is not widespread, but it is growing. One indicator is the creation of the [Ten Years Reproducibility Challenge](https://github.com/ReScience/ten-years), which asks researchers to rerun their old code—ten years or more, very old by computing standards—and see if it still works.

Can you even _find_ your old code? That's a challenge all on its own.

Any researchers out there with computational interests are encouraged to accept the challenge. The link above has details about how to proceed. The results are sure to be eye-opening. Even if you don't submit your results, the exercise will be valuable.

While one can hope the results will not be as dismaying as some would predict, few would expect a happy outcome. It's worth taking a moment to consider why computational reproducibility is such a problem. Computational methods drift through continual change in systems, languages, libraries, approaches, even deployment techniques. Some of the change is necessary, as such addressing security issues caused bad library design; and some is truly enabling, such as the shift to networked servers; but much of the change can be put down to change for its own sake, better known as "progress".

This topic was on my mind earlier this week because of the 10th anniversary of the announcement of the Go programming language as an open source project on November 10, 2009, and associated thoughts for the future. But perhaps the more important date is March 28, 2012, because that was the day that Go version 1.0 was announced.

What makes Go 1.0 so important is that it came with a [promise](https://golang.org/doc/go1compat) that users' programs would continue to compile and run without change for the indefinite future. That promise is an impenetrable bulwark against change for change's sake. Go 1.0 was far from perfect—there were many things that could have been done better, including several that we knew weren't as good as we wanted even at the time—but the promise of true stability more than compensated for any such weaknesses.

Why don't more computational projects make guarantees like this? More languages, especially? Although no Go programs would be eligible for the Ten Year Challenge, the Seven Year Challenge would have a much higher chance of success for a Go program than one in most other languages.

It's not just about promising compatibility, you have to deliver it. There have been countless times when a proposed change to Go would have been nice to accept, but would have broken existing programs, or at least had the potential to do so. The guardwall of true compatibility is constraining, but it is also enabling. It enabled the growth of the Go ecosystem, it enabled the community to prosper, it helped guarantee portability, and it reduced maintenance overhead to nearly zero for many programs.

As the effort towards [Go 2.0](https://blog.golang.org/go2-here-we-come) presses on, the compatibility promise remains. It's far too important to surrender, especially considering how far it has gotten things so far.

Semantic versioning ( [semver](https://semver.org/)) helps, but it's not enough.  It must be deployed in everything, tools included, and its compatibility properties strictly obeyed.

So here's a Ten Year Challenge of my own: Dig out some ten-year-old code, if you can find some, and try to run it today. Do whatever is necessary to make it build and run again. If it's easy, great. If it's not, reflect on what the difficulties were, and what changes caused them, and whether those changes were worthwhile. Could they have been better managed?

An even bigger challenge to system and language designers: Do a favor to your users and codify your compatibility rules. You may not be willing to be as rigid as the Go developers were, but you owe it to your community to be clear about what guarantees you do offer, and to honor them.

If you do, perhaps ten years from now we can do this exercise again with better results.
