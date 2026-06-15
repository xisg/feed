---
title: 'Shifgrethor III: Rooting'
url: https://without.boats/blog/shifgrethor-iii/
published: "2018-10-24T00:00:00Z"
feed: boats
guid: https://without.boats/blog/shifgrethor-iii/
---

# Shifgrethor III: Rooting

After the digression in the previous post, it’s time to get back to what I promised in the first post: a look at how shifgrethor handles rooting. Shifgrethor’s solution is somewhat novel and takes advantage of some of Rust’s specific features, so I want to start by looking briefly at some of the other options.
How to root a GC’d object There are two broad categories of rooting strategies that are common among precise, tracing garbage collectors:
