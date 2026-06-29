---
title: Are Deep Neural Networks Dramatically Overfitted?
url: https://lilianweng.github.io/posts/2019-03-14-overfit/
published: "2019-03-14T00:00:00Z"
feed: lilianweng
guid: https://lilianweng.github.io/posts/2019-03-14-overfit/
---

# Are Deep Neural Networks Dramatically Overfitted?

\[Updated on 2019-05-27: add the [section](#the-lottery-ticket-hypothesis) on Lottery Ticket Hypothesis.\]

If you are like me, entering into the field of deep learning with experience in traditional machine learning, you may often ponder over this question: Since a typical deep neural network has so many parameters and training error can easily be perfect, it should surely suffer from substantial overfitting. How could it be ever generalized to out-of-sample data points?
