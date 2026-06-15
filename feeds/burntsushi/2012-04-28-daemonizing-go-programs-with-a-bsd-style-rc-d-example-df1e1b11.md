---
title: Daemonizing Go Programs (with a BSD-style rc.d example)
url: https://burntsushi.net/golang-daemonize-bsd/
published: "2012-04-28T02:12:00Z"
feed: burntsushi
guid: https://burntsushi.net/golang-daemonize-bsd/
---

# Daemonizing Go Programs (with a BSD-style rc.d example)

Go, by its very nature, is multithreaded. This makes a traditional approach of
daemonizing Go programs by forking a bit difficult.

To get around this, you could try something as simple as backgrounding your Go
program and instructing it to [ignore the HUP\
signal](http://en.wikipedia.org/wiki/Nohup):
