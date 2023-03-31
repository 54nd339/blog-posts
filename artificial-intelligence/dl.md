---
title: Deep Learning Foundations - Shallow Networks and How They Learn
description: The step from linear models to neural networks - what a single hidden layer with a non-linear activation computes, the loss functions for regression and classification, the backpropagation algorithm derived layer by layer, stochastic gradient descent and its batch sizes, and the universal approximation theorem.
date: 2023-03-31
draft: false
slug: /artificial-intelligence/dl
tags:
  - Artificial Intelligence
  - Deep Learning
  - Machine Learning
---

A [linear model](/citadel/artificial-intelligence/ml) — a perceptron, logistic regression, a linear SVM — draws a straight boundary through feature space. That's a hard ceiling: no linear model can separate XOR, or any dataset where the classes interleave. A neural network with a single hidden layer and a non-linear activation has no such ceiling; given enough hidden units it can approximate any continuous function.

This post is the machinery that gets you from one to the other: what a hidden layer actually computes, how the network learns its weights, and the theorem that says it can learn anything.

## Where linear models stop

The [perceptron](/citadel/artificial-intelligence/ann), logistic regression, and the linear SVM all compute a weighted sum $\mathbf{w}^{\mathsf T}\mathbf{x} + b$ and threshold or squash it. The decision boundary is the hyperplane $\mathbf{w}^{\mathsf T}\mathbf{x} + b = 0$ — flat, by construction. When the data isn't linearly separable, no amount of training fixes it; the model family is wrong. You need a boundary that can curve.

## What a shallow network computes

A shallow network — a multilayer perceptron with one hidden layer — is a two-stage computation.

**Hidden layer.** Each hidden neuron $j$ takes the input $\mathbf{x}$, computes a weighted sum, and applies a *non-linear* activation $\phi_h$ (sigmoid, tanh, ReLU):

$$h_j = \phi_h\!\left(\mathbf{w}_j^{(h)\mathsf T}\mathbf{x} + b_j^{(h)}\right)$$

The layer of these activations, $\mathbf{h} = [h_1, \dots, h_K]^{\mathsf T}$, is a new representation of the input — the hidden layer *learns a feature space*.

**Output layer.** The output neuron takes $\mathbf{h}$ and applies an output activation $\phi_o$ suited to the task:

$$y_{out} = \phi_o\!\left(\mathbf{w}^{(o)\mathsf T}\mathbf{h} + b^{(o)}\right)$$

The non-linearity in the hidden layer is what makes this work. Without it, the composition of two linear maps is just another linear map — the network would be no more powerful than a single perceptron. With it, the hidden layer can bend the input space until the output layer's linear decision becomes adequate.

## Loss functions

Training minimises a **loss** measuring the gap between predictions $\hat{y}$ and targets $y$. The choice matches the task:

- **Mean squared error**, for regression:

$$L = \frac{1}{N}\sum_{j=1}^{N}\left(y^{(j)} - \hat{y}^{(j)}\right)^2$$

- **Cross-entropy (log loss)**, for classification. Binary, with predicted probability $\hat{y}$:

$$L = -\frac{1}{N}\sum_{j=1}^{N}\left[y^{(j)}\log \hat{y}^{(j)} + (1 - y^{(j)})\log(1 - \hat{y}^{(j)})\right]$$

  Multi-class, with one-hot targets and a softmax output over $C$ classes:

$$L = -\frac{1}{N}\sum_{j=1}^{N}\sum_{k=1}^{C} y_k^{(j)}\log \hat{y}_k^{(j)}$$

## Backpropagation

**Backpropagation** computes the gradient of the loss with respect to every weight and bias, efficiently, by applying the chain rule layer by layer from the output backward.

**Forward pass.** For neuron $j$ in layer $l$:

$$z_j^{(l)} = \sum_i w_{ji}^{(l)}\,a_i^{(l-1)} + b_j^{(l)}, \qquad a_j^{(l)} = \phi\!\left(z_j^{(l)}\right)$$

running up to the output $\hat{\mathbf{y}} = \mathbf{a}^{(L)}$, then compute the loss.

**Backward pass.** Define an error term $\delta_j^{(l)} = \partial L / \partial z_j^{(l)}$ for each neuron. At the output layer $L$, for the common case of sigmoid or softmax output with cross-entropy loss, it simplifies to

$$\delta_j^{(L)} = \hat{y}_j - y_j$$

For a hidden layer $l$, the error is the next layer's error passed back through the weights, scaled by the local activation slope:

$$\delta_j^{(l)} = \left(\sum_k w_{kj}^{(l+1)}\,\delta_k^{(l+1)}\right)\phi'\!\left(z_j^{(l)}\right)$$

The gradients then fall out:

$$\frac{\partial L}{\partial w_{ji}^{(l)}} = \delta_j^{(l)}\,a_i^{(l-1)}, \qquad \frac{\partial L}{\partial b_j^{(l)}} = \delta_j^{(l)}$$

Every activation function in the chain must be differentiable — this is why the step function of the original perceptron gave way to sigmoid, tanh, and ReLU.

## Gradient descent and its batch sizes

With gradients in hand, an optimiser updates the parameters:

$$w \leftarrow w - \eta\,\frac{\partial L}{\partial w}$$

with **learning rate** $\eta$. The variants differ in how much data goes into each gradient estimate:

- **Batch gradient descent** — the gradient is averaged over the *entire* training set before one update. Stable, slow, memory-heavy.
- **Stochastic gradient descent (SGD)** — one update per training example. The gradient is noisy, which is a feature: the noise helps the optimiser jump out of shallow local minima.
- **Mini-batch SGD** — one update per small batch (32, 64, 256). The default in deep learning — it balances the stability of batch GD against the speed and noise of pure SGD.

Modern training rarely uses plain SGD; optimisers like **Adam**, **RMSprop**, and **AdaGrad** add momentum and per-parameter adaptive learning rates. Their update equations are in [optimisation and generalisation](/citadel/artificial-intelligence/generalisation).

## Universal approximation

The theoretical justification for all of this: the **universal approximation theorem**. A feedforward network with a *single* hidden layer of finitely many neurons and a non-linear activation can approximate any continuous function on a compact subset of $\mathbb{R}^n$ to any desired accuracy, given enough hidden units.

Two caveats. It's an existence result — it says the weights exist, not how to find them (that's training's job) or how many neurons you need. And "single hidden layer is sufficient" doesn't mean "efficient": for functions with compositional structure — images, speech, language — a *deep* network approximates them with far fewer parameters than a wide shallow one, because it can reuse features across layers. That efficiency is the whole case for [deep networks](/citadel/artificial-intelligence/deep-networks).

## The one idea to keep

A hidden layer plus a non-linearity turns a fixed linear model into a learned feature extractor, and backpropagation is just the chain rule organised so that one backward sweep gives you every gradient. The universal approximation theorem says one hidden layer is enough in principle; depth is what makes it practical.
