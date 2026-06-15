---
title: 'Go: Ten years and climbing'
url: https://commandcenter.blogspot.com/2017/09/go-ten-years-and-climbing.html
published: "2017-09-21T22:02:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-1012283476006527133
---

# Go: Ten years and climbing

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh9Yx-ATKr9RgkN-X38eDYh5MsBdTU329zEZNHonUYPkI1MvN3bLDGm2ytpwqQl9EmXmkx-iefEit96R0uubFyFwS7lE8i85r7rMBh_Oufox4KlWL7eI3K0yEcfLQsfio-WfgmmhA/s320/gophers10th.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh9Yx-ATKr9RgkN-X38eDYh5MsBdTU329zEZNHonUYPkI1MvN3bLDGm2ytpwqQl9EmXmkx-iefEit96R0uubFyFwS7lE8i85r7rMBh_Oufox4KlWL7eI3K0yEcfLQsfio-WfgmmhA/s1600/gophers10th.jpg)

Drawing Copyright ©2017 [Renee French](http://reneefrench.io/)

This week marks the 10th anniversary of the creation of Go.

The initial discussion was on the afternoon of Thursday, the 20th of September, 2007. That led to an organized meeting between Robert Griesemer, Rob Pike, and Ken Thompson at 2PM the next day in the conference room called Yaounde in Building 43 on Google's Mountain View campus. The name for the language arose on the 25th, several messages into the first mail thread about the design:

Subject: Re: prog lang discussion
From: Rob 'Commander' Pike
Date: Tue, Sep 25, 2007 at 3:12 PM
To: Robert Griesemer, Ken Thompson

i had a couple of thoughts on the drive home.

1\. name

'go'. you can invent reasons for this name but it has nice properties.
it's short, easy to type. tools: goc, gol, goa. if there's an interactive
debugger/interpreter it could just be called 'go'. the suffix is .go
...

(It's worth stating that the language is called Go; "golang" comes from the web site address (go.com was already a Disney web site) but is not the proper name of the language.)

The Go project counts its birthday as November 10, 2009, the day it launched as open source, originally on code.google.com before migrating to GitHub a few years later. But for now let's date the language from its conception, two years earlier, which allows us to reach further back, take a longer view, and witness some of the earlier events in its history.

The first big surprise in Go's development was the receipt of this mail message:

Subject: A gcc frontend for Go

From: Ian Lance Taylor
Date: Sat, Jun 7, 2008 at 7:06 PM
To: Robert Griesemer, Rob Pike, Ken Thompson

One of my office-mates pointed me at http://.../go\_lang.html . It
seems like an interesting language, and I threw together a gcc
frontend for it. It's missing a lot of features, of course, but it
does compile the prime sieve code on the web page.

The shocking yet delightful arrival of an ally (Ian) and a second compiler (gccgo) was not only encouraging, it was enabling. Having a second implementation of the language was vital to the process of locking down the specification and libraries, helping guarantee the high portability that is part of Go's [promise](https://golang.org/doc/go1compat).

Even though his office was not far away, none of us had even met Ian before that mail, but he has been a central player in the design and implementation of the language and its tools ever since.

Russ Cox joined the nascent Go team in 2008 as well, bringing his own bag of tricks. Russ discovered—that's the right word—that the generality of Go's methods meant that a function could have methods, leading to the [http.HandlerFunc](https://golang.org/pkg/net/http/#HandlerFunc) idea, which was an unexpected result for all of us. Russ promoted more general ideas too, like the the [io.Reader](https://golang.org/pkg/io/#Reader) and [io.Writer](https://golang.org/pkg/io/#Writer) interfaces, which informed the structure of all the I/O libraries.

Jini Kim, who was our product manager for the launch, recruited the security expert Adam Langley to help us get Go out the door. Adam did a lot of things for us that are not widely known, including creating the original [golang.org](https://golang.org/) web page and the [build dashboard](https://build.golang.org/), but of course his biggest contribution was in the cryptographic libraries. At first, they seemed disproportionate in both size and complexity, at least to some of us, but they enabled so much important networking and security software later that they become a crucial part of the Go story. Network infrastructure companies like [Cloudflare](https://www.cloudflare.com/) lean heavily on Adam's work in Go, and the internet is better for it. So is Go, and we thank him.

In fact a number of companies started to play with Go early on, particularly startups. Some of those became powerhouses of cloud computing. One such startup, now called [Docker](https://www.docker.com/), used Go and catalyzed the container industry for computing, which then led to other efforts such as [Kubernetes](https://kubernetes.io/). Today it's fair to say that Go is the language of containers, another completely unexpected result.

Go's role in cloud computing is even bigger, though. In March of 2014 Donnie Berkholz, writing for [RedMonk](https://redmonk.com/), [claimed](http://redmonk.com/dberkholz/2014/03/18/go-the-emerging-language-of-cloud-infrastructure/) that Go was "the emerging language of cloud infrastructure". Around the same time, Derek Collison of [Apcera](https://www.apcera.com/) stated that Go was already the language of the cloud. That might not have been quite true then, but as the word "emerging" used by Berkholz implied, it was becoming true.

Today, Go _is_ the language of the cloud, and to think that a language only ten years old has come to dominate such a large and growing industry is the kind of success one can only dream of. And if you think "dominate" is too strong a word, take a look at the internet inside China. For a while, the huge usage of Go in China signaled to us by the [Google trends graph](https://trends.google.com/trends/explore?q=golang) seemed some sort of mistake, but as anyone who has been to the Go conferences in China can attest, the measurements are real. Go is huge in China.

In short, ten years of travel with the language have brought us past many milestones. The most astonishing is at our current position: a [conservative estimate](https://research.swtch.com/gophercount) suggests there are at least half a million Go programmers. When the mail message naming Go was sent, the idea of there being half a million gophers would have sounded preposterous. Yet here we are, and the number continues to grow.

Speaking of gophers, it's been fun to watch how [Renee French](http://reneefrench.io/)'s idea for a mascot, the Go gopher, became not only a much loved creation but also a symbol for Go programmers everywhere. Many of the biggest Go conferences are called GopherCons as they gather together gophers from all over the world.

Gopher conferences are taking off. The [first one](https://www.youtube.com/playlist?list=PLE7tQUdRKcyb-k4TMNm2K59-sVlUJumw7) was only three years ago, yet today there are many, all around the world, plus countless smaller local " [meetups](https://www.meetup.com/topics/golang/)". On any given day, there is more likely than not a group of gophers meeting somewhere in the world to share ideas.

Looking back over ten years of Go design and development, it is astounding to reflect on the growth of the Go community. The number of conferences and meetups, the long and ever-increasing list of contributors to the Go project, the profusion of open source repositories hosting Go code, the number of companies using Go, some exclusively: these are all astonishing to contemplate.

For the three of us, Robert, Rob, and Ken, who just wanted  to make our programming lives easier, it's incredibly gratifying to witness what our work has started.

What will the next ten years bring?

_\- Rob Pike, with Robert Griesemer and Ken Thompson_
