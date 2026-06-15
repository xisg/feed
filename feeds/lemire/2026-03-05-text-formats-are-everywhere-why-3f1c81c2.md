---
title: Text formats are everywhere. Why?
url: https://lemire.me/blog/2026/03/05/text-formats-are-everywhere-why/
published: "2026-03-05T14:40:58Z"
feed: lemire
guid: https://lemire.me/blog/?p=22526
---

# Text formats are everywhere. Why?

![](https://lemire.me/blog/wp-content/uploads/2026/03/HCp4uWQX0AAhxh0-150x150.jpeg)

The Internet relies on text formats. Thus, we spend a lot of time producing and consuming data encoded in text.

Your web pages are HTML. The code running in them is JavaScript, sent as text (JavaScript source), not as already-parsed code. Your emails, including their attachments, are sent as text (your binary files are sent as text).

It does not stop there. The Python code that runs your server is stored as text. It queries data by sending text queries. It often gets back the answer as text that must then be decoded.

JSON is the universal data interchange format online today. We share maps as JSON (GeoJSON).

Not everything is text, of course. There is no common video or image format that is shared as text. Transmissions over the Internet are routinely compressed to binary formats. There are popular binary formats that compete with JSON.

But why is text dominant?

It is not because, back in the 1970s, programmers did not know about binary formats.

In fact, we did not start with text formats. Initially, we worked with raw binary data. Those of us old enough will remember programming in assembly using raw byte values.

Why text won?

1.Text is efficient.

In the XML era, when everything had to be XML, there were countless proposals for binary formats. People were sometimes surprised to find that the binary approach was not much faster in practice. Remember that many text formats date back to an era when computers were much slower. Had text been a performance bottleneck, it would not have spread. Of course, there are cases where text makes things slower. You then have a choice: optimize your code further or transition to another format. Often, both are viable.

It is easy to make wrong assumptions about binary formats, such as that you can consume them without any parsing or validation. If you pick up data from the Internet, you must assume that it could have been sent by an adversary or someone who does not follow your conventions.

2.Text is easy to work with.

If you receive text from a remote source, you can often transform it, index it, search it, quote it, version it… with little effort and without in-depth knowledge of the format. Text is often self-documenting.

In an open world, when you will never speak with the person producing the data, text often makes everything easier and smoother.

If there is an issue to report and the data is in text, you can usually copy-paste the relevant section into a message. Things are much harder with a binary format.
