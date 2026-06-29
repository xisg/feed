---
title: simonw/browser-compat-db
url: https://simonwillison.net/2026/Jun/24/browser-compat-db/#atom-everything
published: "2026-06-24T23:59:03Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/24/browser-compat-db/#atom-everything
---

# simonw/browser-compat-db

**[simonw/browser-compat-db](https://github.com/simonw/browser-compat-db)**

Inspired by Mozilla's [new MDN MCP service](https://developer.mozilla.org/en-US/blog/introducing-mdn-mcp-server/) \- [source code here](https://github.com/mdn/mcp) \- I decided to try converting their comprehensive [mdn/browser-compat-data](https://github.com/mdn/browser-compat-data) repository full of browser compatibility data into a SQLite database.

This new GitHub repo includes a Claude Code for web (Opus 4.8) [generated script](https://github.com/simonw/browser-compat-db/blob/main/build_db.py) for doing that using [sqlite-utils](https://github.com/simonw/sqlite-utils).

I wanted the resulting ~66MB SQLite database to be available via the GitHub CDN with open CORS headers. GitHub releases don't have those, but any file stored in a regular GitHub repository does - so I had Codex Desktop (GPT-5.5) build [a GitHub Actions workflow](https://github.com/simonw/browser-compat-db/blob/main/.github/workflows/build-db.yml) that builds the database and then force-pushes it to a `db` "orphan" branch.

You can download the resulting database [from here](https://github.com/simonw/browser-compat-db/blob/db/browser-compat.db), and since it's hosted with open CORS headers you can also [explore it with Datasette Lite](https://lite.datasette.io/?url=https://github.com/simonw/browser-compat-db/blob/db/browser-compat.db#/browser-compat/releases_tree).

Tags: [github](https://simonwillison.net/tags/github), [mozilla](https://simonwillison.net/tags/mozilla), [projects](https://simonwillison.net/tags/projects), [github-actions](https://simonwillison.net/tags/github-actions), [datasette-lite](https://simonwillison.net/tags/datasette-lite), [ai-assisted-programming](https://simonwillison.net/tags/ai-assisted-programming), [model-context-protocol](https://simonwillison.net/tags/model-context-protocol), [mdn](https://simonwillison.net/tags/mdn)
