---
title: 'Shifgrethor I: Garbage collection as a Rust library'
url: https://without.boats/blog/shifgrethor-i/
published: "2018-10-16T00:00:00Z"
feed: boats
guid: https://without.boats/blog/shifgrethor-i/
---

# Shifgrethor I: Garbage collection as a Rust library

I’m really excited to share with you an experiment that I’ve been working on for the past 5 or 6 weeks. It’s a Rust library called shifgrethor. shifgrethor implements a garbage collector in Rust with an API I believe to be properly memory safe.
I’ll be going through all of the technical details in future blog posts, so I want to kick this series off with a high level overview of the project’s purpose and design decisions.
