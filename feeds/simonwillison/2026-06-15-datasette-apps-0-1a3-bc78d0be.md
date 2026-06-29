---
title: datasette-apps 0.1a3
url: https://simonwillison.net/2026/Jun/15/datasette-apps-2/#atom-everything
published: "2026-06-15T20:25:07Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/15/datasette-apps-2/#atom-everything
---

# datasette-apps 0.1a3

**Release:** [datasette-apps 0.1a3](https://github.com/datasette/datasette-apps/releases/tag/0.1a3)

> - Fixed a bug where users without the `create-app` permission could still create apps. [#27](https://github.com/datasette/datasette-apps/issues/27)
> - Fixed a bug where it was impossible to grant permission to edit an app to users who were not the app's owner. The rules for edit/delete are now the same as view: if the app is private only the owner can modify it, otherwise permission is controlled by Datasette's regular permission system. [#29](https://github.com/datasette/datasette-apps/issues/29)

Tags: [datasette](https://simonwillison.net/tags/datasette)
