---
title: Optimisation and Generalisation - Landscapes, Optimisers, and Sequence Models
description: "What stands between a deep network and its job - the non-convex loss landscape and the optimisers built for it (momentum, AdaGrad, RMSprop, Adam), the training-versus-test gap, and the advanced architectures: spatial transformers, recurrent networks and LSTMs, RNN language models, and deep reinforcement learning."
date: 2023-04-06
draft: false
slug: /artificial-intelligence/generalisation
tags:
  - Artificial Intelligence
  - Deep Learning
  - Machine Learning
---

Two problems sit between a deep network and useful behaviour. Its loss landscape is not a smooth bowl — it's full of saddle points and flat regions that stall gradient descent. And low error on the training data is not the goal; low error on data it has never seen is. This post covers the optimisers built for that landscape, the training-versus-test gap, and the architectures that extend deep learning to sequences and to learning by acting.

## The non-convex landscape

[Linear and logistic regression](/citadel/artificial-intelligence/ml) have convex losses: one global minimum, and gradient descent walks straight to it. Deep networks don't. Their loss surfaces are highly **non-convex**, with:

- **Local minima** — lower than their surroundings but not the global best.
- **Saddle points** — gradient zero, but a minimum along some directions and a maximum along others. In high dimensions these, not local minima, are what slow training down.
- **Plateaus** — large flat regions where the gradient is tiny and progress crawls.

There's no way to guarantee convergence to the global minimum. In practice, good-enough minima are common and reachable — the engineering is in the optimiser.

## Stochastic optimisation

[SGD](/citadel/artificial-intelligence/dl) estimates the gradient from a mini-batch instead of the whole dataset:

$$\mathbf{w}_{t+1} = \mathbf{w}_t - \eta\,\frac{1}{m}\sum_{j=1}^{m}\nabla L_j(\mathbf{w}_t)$$

The estimate is noisy, and the noise is useful — it kicks the optimiser off saddle points and out of shallow minima that batch gradient descent would sit in. Four refinements are standard:

- **Momentum** — accumulate a velocity and step with it:

$$\mathbf{v}_{t+1} = \mu\,\mathbf{v}_t - \eta\,\nabla L(\mathbf{w}_t), \qquad \mathbf{w}_{t+1} = \mathbf{w}_t + \mathbf{v}_{t+1}$$

  This accelerates along directions where the gradient is consistent and damps oscillation across narrow valleys.

- **AdaGrad** — scale each parameter's learning rate by the inverse square root of its accumulated squared gradients, so rarely updated parameters take larger steps. Downside: the accumulator only grows, so the effective rate decays toward zero.

- **RMSprop** — AdaGrad with a *moving average* of squared gradients instead of a sum, so the learning rate stops decaying to nothing.

- **Adam** — combines momentum with RMSprop's per-parameter adaptive rate, tracking exponentially decaying averages of both the gradient and its square. The default optimiser for most deep learning.

## Generalisation

The target is low **test** error — performance on unseen data from the same distribution — not low training error. Deep networks have enormous capacity and can memorise the training set, noise and all, then fail on new data.

In [bias-variance](/citadel/artificial-intelligence/advanced-ml) terms, deep networks achieve very low bias because they're so expressive; the fight is controlling **variance**. The tools — L1/L2 penalties, dropout, early stopping, data augmentation, batch normalisation — are covered with [deep networks](/citadel/artificial-intelligence/deep-networks). Capacity measures like VC dimension are hard to compute for deep nets, so generalisation is mostly managed empirically, by watching a validation set.

## Spatial transformer networks

A **spatial transformer network** (STN) is a learnable module — usually dropped into a CNN — that applies a spatial transformation to its input feature map, giving the network geometric invariance to translation, scale, rotation, and warping. Three parts:

1. **Localisation network** — a small net that regresses the parameters $\theta$ of a transformation (e.g. an affine matrix) from the input feature map.
2. **Grid generator** — builds a sampling grid from $\theta$: for each output pixel, which input coordinates to sample.
3. **Sampler** — warps the input using the grid, via differentiable bilinear interpolation so gradients flow back and the localisation network trains end-to-end.

An STN lets a network learn to "normalise" the pose of objects before classifying them.

## Recurrent networks

**Recurrent neural networks** (RNNs) process sequences by carrying a hidden state across time steps. At step $t$:

$$h_t = \phi_h(W_{hh}\,h_{t-1} + W_{xh}\,x_t + b_h), \qquad y_t = \phi_y(W_{hy}\,h_t + b_y)$$

The same weights are used at every step. Training is **backpropagation through time** — unroll the recurrence over the sequence and apply backpropagation to the unrolled network.

The weakness: over many time steps the repeated multiplication by $W_{hh}$ makes gradients **vanish** or **explode**, so a plain RNN can't learn dependencies between elements far apart in a sequence.

## LSTMs

**Long short-term memory** networks fix this with a cell that has an explicit **cell state** $C_t$ — a long-term memory — regulated by three gates:

$$f_t = \sigma(W_f[h_{t-1}, x_t] + b_f) \qquad \text{(forget: what to drop from } C_{t-1})$$
$$i_t = \sigma(W_i[h_{t-1}, x_t] + b_i), \quad \tilde{C}_t = \tanh(W_C[h_{t-1}, x_t] + b_C) \qquad \text{(input: what to add)}$$
$$C_t = f_t \odot C_{t-1} + i_t \odot \tilde{C}_t \qquad \text{(update the cell state)}$$
$$o_t = \sigma(W_o[h_{t-1}, x_t] + b_o), \quad h_t = o_t \odot \tanh(C_t) \qquad \text{(output: what to expose)}$$

with $\odot$ elementwise multiplication and $\sigma$ the sigmoid. The gates let information pass along the cell state nearly unchanged over long spans, or be deliberately forgotten — which is what lets LSTMs capture long-range context. **Gated recurrent units** (GRUs) are a simpler two-gate variant.

## RNN language models

An RNN language model processes text word by word (each word a learned embedding). At each step, from the current word and the hidden state, it predicts a probability distribution over the vocabulary for the next word:

$$P(w_{t+1} \mid w_1, \dots, w_t) \approx P(w_{t+1} \mid h_t)$$

Because the hidden state summarises *all* preceding words, not a fixed window, an RNN language model can in principle use arbitrarily long context — which gives lower [perplexity](/citadel/artificial-intelligence/nlp) and more coherent generation than n-gram models. The same word-level RNNs underpin sentiment analysis, sequence-to-sequence translation, and question answering.

## Deep reinforcement learning

**Reinforcement learning** is learning to act: an agent interacts with an environment, receives rewards and penalties, and learns a **policy** (a map from states to actions) that maximises cumulative future reward.

**Deep RL** uses a neural network as the function approximator when the state or action space is too large for a lookup table:

- **Value approximation** — a **Q-network** (as in DQN) takes a state, e.g. raw game-screen pixels, and outputs the expected future reward of each action.
- **Policy approximation** — a policy-gradient network takes a state and outputs a distribution over actions directly.

Deep RL mastered Atari games from pixels (DQN) and reached superhuman play in Go and chess (AlphaGo, AlphaZero). Its open problems are sample efficiency (it needs a lot of environment interaction), the exploration-versus-exploitation balance, and training stability.

## Brains and AI

The exchange runs both ways. Neuroscience has fed AI: the visual cortex's hierarchical processing shaped CNNs; Hebbian plasticity ("fire together, wire together") is a learning-rule idea; attention in human cognition inspired attention mechanisms in models. And AI feeds neuroscience: trained networks serve as testable hypotheses about brain function, and deep-learning tools analyse large-scale neural recordings and simulate circuits.

## The one idea to keep

The optimisers — momentum, RMSprop, Adam — exist because a deep network's loss surface is not a bowl; they add memory of past gradients so the descent doesn't stall on saddle points and plateaus. The architectures — STNs, LSTMs, Q-networks — are each a way of building an inductive bias into the network: spatial invariance, a gated long-term memory, a value function over states. Both are about making a very general function approximator behave well on a specific kind of problem.
