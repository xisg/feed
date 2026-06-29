---
title: Temporary Cloudflare Accounts for AI agents
url: https://simonwillison.net/2026/Jun/21/temporary-cloudflare-accounts/#atom-everything
published: "2026-06-21T22:01:04Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/21/temporary-cloudflare-accounts/#atom-everything
---

# Temporary Cloudflare Accounts for AI agents

**[Temporary Cloudflare Accounts for AI agents](https://blog.cloudflare.com/temporary-accounts/)**

The announcement says this is "for AI agents" but (as is pretty common these days) the AI hook isn't really necessary, this is an interesting feature for everyone else as well.

Short version: you can now create a Cloudflare Workers project and run this, without even creating a Cloudflare account:

```
npx wrangler deploy --temporary

```

Cloudflare will deploy the application to a new, ephemeral project which will stay live for 60 minutes.

I [had GPT-5.5 xhigh](https://gist.github.com/simonw/264bd6b8a39fc34c91c9c867454c64b9) in Codex Desktop [build this test application](https://github.com/simonw/cloudflare-redirect-resolver) providing a tool for following HTTP redirects and returning the final destination. The temporary deployment worked as advertised.

Running the deployment spits out the URL to a page for claiming the new project, for if you want it to last for more than 60 minutes. Here's what that claim screen looks like:

![Screenshot of a Cloudflare account claim page. A red banner at top reads "This claim link expires in 49:26". Below, a card titled "Educated Celery" with the text "Claim this account to take ownership of cloudflare-redirect-resolver and all its resources." and a blue "Claim Account" button. A worker entry shows "cloudflare-redirect-resolver" with the URL "cloudflare-redirect-resolver.educated-celery.workers.dev".](https://static.simonwillison.net/static/2026/cloudflare-claim.jpg)

Via [Hacker News](https://news.ycombinator.com/item?id=48608394)

Tags: [cloudflare](https://simonwillison.net/tags/cloudflare)
