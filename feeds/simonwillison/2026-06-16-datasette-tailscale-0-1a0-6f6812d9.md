---
title: datasette-tailscale 0.1a0
url: https://simonwillison.net/2026/Jun/16/datasette-tailscale/#atom-everything
published: "2026-06-16T16:18:20Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/16/datasette-tailscale/#atom-everything
---

# datasette-tailscale 0.1a0

**Release:** [datasette-tailscale 0.1a0](https://github.com/datasette/datasette-tailscale/releases/tag/0.1a0)

A very experimental alpha plugin which lets you do this:

```
datasette tailscale mydata.db \
  --ts-authkey tskey-auth-xxxx --ts-hostname datasette-preview

```

This starts a localhost Datasette server with a [Tailscale](https://tailscale.com/) sidecar that connects it to your Tailnet, such that `http://datasette-preview/` serves Datasette.

It's using the Python bindings for the experimental [tailscale-rs](https://github.com/tailscale/tailscale-rs) library. I [filed an issue](https://github.com/tailscale/tailscale-rs/issues/243) asking if there's a cleaner way of setting up the proxy mechanism.

Tags: [datasette](https://simonwillison.net/tags/datasette), [tailscale](https://simonwillison.net/tags/tailscale)
