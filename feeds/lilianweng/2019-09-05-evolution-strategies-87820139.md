---
title: Evolution Strategies
url: https://lilianweng.github.io/posts/2019-09-05-evolution-strategies/
published: "2019-09-05T00:00:00Z"
feed: lilianweng
guid: https://lilianweng.github.io/posts/2019-09-05-evolution-strategies/
---

# Evolution Strategies

Stochastic gradient descent is a universal choice for optimizing deep learning models. However, it is not the only option. With black-box optimization algorithms, you can evaluate a target function $f(x): \\mathbb{R}^n \\to \\mathbb{R}$, even when you don’t know the precise analytic form of $f(x)$ and thus cannot compute gradients or the Hessian matrix. Examples of black-box optimization methods include [Simulated Annealing](https://en.wikipedia.org/wiki/Simulated_annealing), [Hill Climbing](https://en.wikipedia.org/wiki/Hill_climbing) and [Nelder-Mead method](https://en.wikipedia.org/wiki/Nelder%E2%80%93Mead_method).
