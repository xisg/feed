---
title: QUERY with curl
url: https://daniel.haxx.se/blog/2026/06/21/query-with-curl/
published: "2026-06-20T22:34:25Z"
feed: danielstenberg
guid: https://daniel.haxx.se/blog/?p=28607
---

# QUERY with curl

[RFC 10008](https://www.rfc-editor.org/info/rfc10008/) is brand new a specification detailing the new HTTP method called QUERY:

*This specification defines the QUERY method for HTTP. A QUERY requests that the request target process the enclosed content in a safe and idempotent manner and then respond with the result of that processing. This is similar to POST requests but can be automatically repeated or restarted without concern for partial state changes*

## A GET with body

For all practical purposes you can think of QUERY as a way to send a GET with a body. It looks exactly like POST, but done with another verb.

Contrary to POST, QUERY requests are *idempotent* – they can be retried or repeated when needed, for instance after a connection failure.

## curl it

You can use curl to do HTTP requests with QUERY just fine. curl offers the `--request` option (also known as -X in the short form) that you can use like this:

```
curl -d "data to send" -X QUERY https://example.com/
```

## But redirects!

There is one little caveat to remember with this curl option that changes the method. When *also* asking curl to follow any possible redirects, it is important that you use a new enough curl version because you want the [`--follow`](https://daniel.haxx.se/blog/2025/08/06/follow-redirects-but-differently/) option. **Not** the old `--location/-L` one.

Why? Because the old option changes the HTTP method on all subsequent requests independently of what the server responds, which in many cases is not what you want.

The newer `--follow` option instead acts according to what the HTTP response code suggests in should do. Stick to the same method again, or maybe switch to GET in the following request.

## Why?

Why or when would you use this? First of course you only want to use this if the server supports it, but the spec offers some reasons why this might be a good choice:

- avoid or circumvent URL size limits. Somewhere around 8000 bytes they start to no longer work reliably because servers and intermediaries set limits.
- expressing certain kinds of data in the URL is inefficient because encoding overhead
- URLs are more likely to be logged than request content
