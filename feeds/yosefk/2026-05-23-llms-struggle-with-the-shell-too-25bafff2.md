---
title: LLMs struggle with the shell, too
url: https://yosefk.com/blog/llms-struggle-with-the-shell-too.html
published: "2026-05-23T00:00:00Z"
feed: yosefk
---

# LLMs struggle with the shell, too

You used to tell people, “why are you doing all this by hand — write a script to do it!”, and then “...I meant an actual
Python script, not a buggy grep \| sed \| crap pipeline!”

This got better since some of those too lazy to write a script (or not lazy enough to avoid the harder, buggier way?) now ask
an LLM for the script. But you know who doesn’t? LLMs!

“Agentic LLM harnesses” seem to think it’s bad to pollute the filesystem with scripts, and that they’re very good with ad-hoc
shell commands. Neither is true. I actually prefer to be able to see their scripts and run them myself (that’s how you
eventually discover that an image file diffing script the thing made says the images match when they don’t) [1](#fn1). And it turns out that LLMs are just as bad as people at
coming up with the right long-winded shell command — give them two source directories to build two program versions and debug
differences in their output, and they keep forgetting to rebuild, or they cd to the wrong place, ad infinitum — but they’ll get
this right if you ask them to do it with a script!

Except they won’t. You keep telling them to put things in scripts, you yell in all caps, you put it into their config file,
they make a “long-term memory” out of it (another file) — but they keep coming up with ad-hoc bug-ridden shell commands, and you
need to remind them again and again, _put this into a script_.

Who says you can’t emulate human intelligence with a fuckload-dimensional function? These things struggle with the shell just
the way people do!

* * *

1. There’s another “advantage” to agents using scripts — you only need to give them permission to run the
   interpreter once, instead of approving many different inline commands with features the harness thinks require approval, like cd
   or env var expansion. Of course, what this really shows is how worthless the agent “sandboxing” is, and that you “should” “just”
   run in YOLO mode inside a container or something. Why people often don’t do what they should and what “just” amounts to is a
   separate matter. [↩︎](#fnref1)
