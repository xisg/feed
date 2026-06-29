---
title: Cloudflare CAPTCHA on at least one ampersand
url: https://simonwillison.net/2026/Jun/16/captcha-on-at-least-one-ampersand/#atom-everything
published: "2026-06-16T00:21:36Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/16/captcha-on-at-least-one-ampersand/#atom-everything
---

# Cloudflare CAPTCHA on at least one ampersand

**TIL:** [Cloudflare CAPTCHA on at least one ampersand](https://til.simonwillison.net/cloudflare/captcha-on-at-least-one-ampersand)

I'm using Cloudflare's CAPTCHA (they call it a "Web Application Firewall > Custom rules > Managed Challenge" these days) to prevent crawlers from aggresively spidering my [faceted search engine](https://simonwillison.net/2017/Oct/5/django-postgresql-faceted-search/) on this site, but I got fed up of even simple `?q=term` searches triggering the challenge.

After some mucking around with Claude Code it turns out you can register the following rule instead, so the CAPTCHA only kicks in for search URLs containing at least one ampersand:

`(http.request.uri.path wildcard r"/search/*" and http.request.uri.query contains "&")`

And now [/search/?q=lemur](https://simonwillison.net/search/?q=lemur) works without triggering a CAPTCHA!

Also included: notes on [trying out the Cloudflare MCP with Claude Code](https://til.simonwillison.net/cloudflare/captcha-on-at-least-one-ampersand#trying-the-cloudflare-mcp), though it turned out not to be able to edit the rules in question so I had Claude Code [switch to the Cloudflare API](https://til.simonwillison.net/cloudflare/captcha-on-at-least-one-ampersand#using-the-api-instead) instead.

Tags: [captchas](https://simonwillison.net/tags/captchas), [cloudflare](https://simonwillison.net/tags/cloudflare), [model-context-protocol](https://simonwillison.net/tags/model-context-protocol), [claude-code](https://simonwillison.net/tags/claude-code)
