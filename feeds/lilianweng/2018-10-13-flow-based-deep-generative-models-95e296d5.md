---
title: Flow-based Deep Generative Models
url: https://lilianweng.github.io/posts/2018-10-13-flow-models/
published: "2018-10-13T00:00:00Z"
feed: lilianweng
guid: https://lilianweng.github.io/posts/2018-10-13-flow-models/
---

# Flow-based Deep Generative Models

So far, I’ve written about two types of generative models, [GAN](https://lilianweng.github.io/posts/2017-08-20-gan/) and [VAE](https://lilianweng.github.io/posts/2018-08-12-vae/). Neither of them explicitly learns the probability density function of real data, $p(\\mathbf{x})$ (where $\\mathbf{x} \\in \\mathcal{D}$) — because it is really hard! Taking the generative model with latent variables as an example, $p(\\mathbf{x}) = \\int p(\\mathbf{x}\\vert\\mathbf{z})p(\\mathbf{z})d\\mathbf{z}$ can hardly be calculated as it is intractable to go through all possible values of the latent code $\\mathbf{z}$.
