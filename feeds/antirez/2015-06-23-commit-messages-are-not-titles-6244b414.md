---
title: Commit messages are not titles
url: http://antirez.com/news/90
published: "2015-06-23T08:55:22Z"
feed: antirez
guid: http://antirez.com/news/90
---

# Commit messages are not titles

Nor subjects, for what matters. Everybody will tell you to don't add a dot at the end of the first line of a commit message. I followed the advice for some time, but I'll stop today, because I don't believe commit messages are titles or subjects. They are synopsis of the meaning of the change operated by the commit, so they are small sentences. The sentence can be later augmented with more details in the next lines of the commit message, however many times there is \*no\* body, there is just the first line. How many emails or articles you see with just the subject or the title? Very little, I guess. So for me it is like:

This is a smart synopsis, as information dense as possible.

And when needed, this is the long version since:

1\. I did this.

2\. And resulted into this.

3\. And you could reproduce this way.

So every time I'll be told again to don't put a dot at the end, I'll link to this article.

But no, it's not just a matter of a dot. If the first line of a commit message is a title, it changes \*the way\* you write it. It becomes just some text to introduce some more text, without any stress on the information density. Coders gotta code, so if something can be told in a very short way in one line, do it, and reserve the other additional informations for the next line, without sacrificing the first line since "It's a title".

Moreover, programming is the art of writing synopsis, otherwise you end with programs much more complex they should be. So perhaps it's also a good exercise for us.
[Comments](http://antirez.com/news/90)
