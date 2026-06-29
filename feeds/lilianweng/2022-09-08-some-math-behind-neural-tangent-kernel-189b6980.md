---
title: Some Math behind Neural Tangent Kernel
url: https://lilianweng.github.io/posts/2022-09-08-ntk/
published: "2022-09-08T17:00:00Z"
feed: lilianweng
guid: https://lilianweng.github.io/posts/2022-09-08-ntk/
---

# Some Math behind Neural Tangent Kernel

Neural networks are [well known](https://lilianweng.github.io/posts/2019-03-14-overfit/) to be over-parameterized and can often easily fit data with near-zero training loss with decent generalization performance on test dataset. Although all these parameters are initialized at random, the optimization process can consistently lead to similarly good outcomes. And this is true even when the number of model parameters exceeds the number of training data points.

**Neural tangent kernel (NTK)** ( [Jacot et al. 2018](https://arxiv.org/abs/1806.07572)) is a kernel to explain the evolution of neural networks during training via gradient descent. It leads to great insights into why neural networks with enough width can consistently converge to a global minimum when trained to minimize an empirical loss. In the post, we will do a deep dive into the motivation and definition of NTK, as well as the proof of a deterministic convergence at different initializations of neural networks with infinite width by characterizing NTK in such a setting.
