---
title: 'Announcing a new project: configure'
url: https://without.boats/blog/configure/
published: "2018-01-18T00:00:00Z"
feed: boats
guid: https://without.boats/blog/configure/
---

# Announcing a new project: configure

Hi :) I’ve been working on a project called configure, which is intended to create a uniform way to load configuration variables from the environment of the program. Specifically, the goal is to create something that libraries can rely on to allow applications to delegate decisions about how configuration is loaded to applications, without those applications having to write a lot of bespoke configuration management glue.
Storing configuration in the environment “The 12 Factor App” has this very good advice about managing configuration:
