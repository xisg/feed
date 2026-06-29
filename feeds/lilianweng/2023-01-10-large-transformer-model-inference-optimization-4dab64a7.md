---
title: Large Transformer Model Inference Optimization
url: https://lilianweng.github.io/posts/2023-01-10-inference-optimization/
published: "2023-01-10T17:00:00Z"
feed: lilianweng
guid: https://lilianweng.github.io/posts/2023-01-10-inference-optimization/
---

# Large Transformer Model Inference Optimization

\[Updated on 2023-01-24: add a small section on [Distillation](#distillation).\]

Large transformer models are mainstream nowadays, creating SoTA results for a variety of tasks. They are powerful but very expensive to train and use. The extremely high inference cost, in both time and memory, is a big bottleneck for adopting a powerful transformer for solving real-world tasks at scale.

**Why is it hard to run inference for large transformer models?** Besides the increasing size of SoTA models, there are two main factors contributing to the inference challenge ( [Pope et al. 2022](https://arxiv.org/abs/2211.05102)):
