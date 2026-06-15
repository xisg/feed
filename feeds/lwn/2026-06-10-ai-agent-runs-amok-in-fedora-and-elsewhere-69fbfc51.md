---
title: '[$] AI agent runs amok in Fedora and elsewhere'
url: https://lwn.net/Articles/1077035/
published: "2026-06-10T14:35:25Z"
feed: lwn
guid: https://lwn.net/Articles/1077035/
---

# [$] AI agent runs amok in Fedora and elsewhere

Agentic AI systems can be used to do a variety of things
autonomously on behalf of a human user: open or manage bugs, generate
code, submit pull-requests, and (apparently) even [complain about\
rejection](https://lwn.net/Articles/1058643/). In May, a Fedora developer discovered that an allegedly
rogue agent had been pestering the project in a number of ways:
reassigning bugs, fabricating unhelpful replies to bugs, and even
persuading maintainers to merge questionable code into the [Anaconda\
installer](https://github.com/rhinstaller/anaconda#anaconda). It also submitted a number of pull requests (PRs),
some accepted, to several upstream projects. The Fedora account
associated with the agent has had its group privileges revoked and the
messes have been mopped up, but the motive behind the agent's actions is still
a mystery.
