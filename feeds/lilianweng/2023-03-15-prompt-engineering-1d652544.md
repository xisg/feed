---
title: Prompt Engineering
url: https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/
published: "2023-03-15T00:00:00Z"
feed: lilianweng
guid: https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/
---

# Prompt Engineering

**Prompt Engineering**, also known as **In-Context Prompting**, refers to methods for how to communicate with LLM to steer its behavior for desired outcomes *without* updating the model weights. It is an empirical science and the effect of prompt engineering methods can vary a lot among models, thus requiring heavy experimentation and heuristics.

This post only focuses on prompt engineering for autoregressive language models, so nothing with Cloze tests, image generation or multimodality models. At its core, the goal of prompt engineering is about alignment and model steerability. Check my [previous post](https://lilianweng.github.io/posts/2021-01-02-controllable-text-generation/) on controllable text generation.
