---
title: Deep Networks - Why Depth Works and the Architectures Built On It
description: What changed to make deep networks trainable and what depth buys - the vanishing-gradient wall and the breakthroughs past it, regularisation and batch normalisation, model capacity and VC dimension, convolutional networks and the landmark CNN architectures, weight initialisation, GANs, and semi-supervised learning.
date: 2023-04-02
draft: false
slug: /artificial-intelligence/deep-networks
tags:
  - Artificial Intelligence
  - Deep Learning
  - Machine Learning
---

The [universal approximation theorem](/citadel/artificial-intelligence/dl) says a single hidden layer is enough to approximate any continuous function. Yet the field spent two decades unable to train networks more than a few layers deep — and then, in the mid-2000s, suddenly could. This post is about what changed, why *depth* is worth the trouble, and the architectures and training tricks that came out of it.

## From winters to breakthrough

The groundwork — McCulloch and Pitts' neuron, Hebbian learning, [Rosenblatt's perceptron](/citadel/artificial-intelligence/ann) — was laid in the 1940s–60s. The first **AI winter** followed in the 1970s, after Minsky and Papert showed a single-layer perceptron can't compute XOR and training deeper ones seemed hopeless. **Backpropagation**, popularised in the 1980s, revived multilayer networks and drove a wave of pattern-recognition successes.

But *deep* networks stayed stuck through the 1990s and early 2000s:

- **Vanishing gradients** — with sigmoid/tanh activations, the error signal shrinks by a factor at every layer it's propagated back through, so early layers learn glacially or not at all.
- **Exploding gradients** — the opposite failure: gradients blow up, and training diverges.
- **Compute cost** — training large networks was prohibitively slow.
- **Overfitting** — millions of parameters, not enough labelled data.

Support vector machines, needing less tuning and less data, outperformed neural nets for most of this period. The breakthrough came from a stack of changes landing together: **ReLU activations and better weight initialisation** (which keep gradients alive), better optimisers, **dropout** for regularisation, **ImageNet**-scale labelled datasets, **GPUs** for cheap parallel matrix math, and specialised architectures — CNNs for vision, [LSTMs](/citadel/artificial-intelligence/generalisation) for sequences.

## Why depth

Universal approximation is an *existence* claim, not an efficiency one. A deep network's advantage is that it learns a **feature hierarchy**. In image recognition:

- Layer 1: edges, corners.
- Layer 2: textures, simple shapes.
- Layer 3: object parts — an eye, a wheel.
- Higher layers: whole objects.

For functions with this kind of compositional structure — and real-world data mostly has it — a deep network needs *exponentially fewer* parameters than a wide shallow one to reach the same accuracy. The learned intermediate features are also more robust and often transferable to related tasks.

## Regularisation

With millions of parameters, a deep network overfits readily. The countermeasures:

- **L2 (weight decay)** — add $\lambda \sum_i w_i^2$ to the loss, pushing weights small for a simpler model.
- **L1** — add $\lambda \sum_i |w_i|$, which drives many weights to exactly zero (a form of feature selection).
- **Dropout** — during training, randomly zero a fraction of neurons per mini-batch. Neurons can't rely on specific partners, so co-adaptation breaks down; it's roughly like training and averaging many thinned networks.
- **Early stopping** — watch validation error; stop when it starts rising while training error is still falling.
- **Data augmentation** — expand the training set with transformed copies (rotations, flips, crops, colour shifts), teaching invariance to those transformations.

## Batch normalisation

As early-layer parameters change during training, the distribution of inputs to *later* layers keeps shifting — **internal covariate shift** — which slows training and makes it sensitive to the learning rate and initialisation.

**Batch normalisation** normalises each layer's activations per mini-batch:

$$\hat{x}_i = \frac{x_i - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}}, \qquad y_i = \gamma\,\hat{x}_i + \beta$$

subtracting the batch mean, dividing by the batch standard deviation, then applying a learnable scale $\gamma$ and shift $\beta$ so the network can still choose the distribution it wants. It makes training faster and more stable, allows higher learning rates, has a mild regularising effect, and reduces initialisation sensitivity.

## Model capacity: VC dimension

The **Vapnik-Chervonenkis dimension** measures a hypothesis class's capacity: the largest set of points it can **shatter** — classify correctly under *every* possible labelling. A single perceptron in $d$ dimensions has VC dimension $d + 1$. Higher VC dimension means more expressive power but more data needed to generalise without overfitting. For neural nets the VC dimension grows with the parameter count and depth, and the [PAC framework](/citadel/artificial-intelligence/theorotical-ml) uses it to bound how many examples good generalisation requires.

## Convolutional networks

For grid-structured data — images above all — **convolutional neural networks** (CNNs) replace the fully connected layer with three ideas: local receptive fields, weight sharing, and pooling.

**Convolutional layers.** A layer holds a set of small learnable **filters**. Each filter slides across the input, computing a dot product with the local patch it covers, producing one value in an output **feature map**:

$$O_{i,j} = \sum_m \sum_n I_{i+m,\,j+n}\,K_{m,n} + b$$

Two properties matter. **Local receptive field** — each output depends on only a small input region. **Parameter sharing** — the same filter is used at every position, so the layer has few parameters and detects a feature wherever it appears (translation equivariance). A ReLU is applied after the convolution.

**Pooling layers** downsample feature maps — max pooling takes the largest value in each patch, average pooling the mean. This shrinks the spatial dimensions, cuts computation, and adds tolerance to small shifts.

**Fully connected layers** at the end take the flattened high-level features and produce the final classification (softmax) or regression.

## Landmark CNN architectures

Each of these won or reshaped the ImageNet challenge (ILSVRC):

- **AlexNet (2012)** — Krizhevsky et al. won ILSVRC 2012 by a wide margin, reigniting the field. Trained on GPUs, used ReLU (trains faster than sigmoid/tanh), dropout, and data augmentation. Five convolutional layers, three fully connected.
- **VGGNet (2014)** — Oxford's Visual Geometry Group. Used only stacked $3 \times 3$ filters to build very deep, uniform networks (VGG-16, VGG-19). Showed depth alone, with a simple design, drives accuracy.
- **GoogLeNet / Inception (2014)** — the **Inception module** runs $1 \times 1$, $3 \times 3$, $5 \times 5$ convolutions and pooling in parallel within one layer and concatenates the results, capturing features at multiple scales while keeping the parameter count low.
- **ResNet (2015)** — Microsoft Research. **Residual (skip) connections** let a block learn a residual $F(x)$ and output $H(x) = F(x) + x$, so an unhelpful block can approximate the identity. This tamed the vanishing-gradient problem enough to train networks of 152 layers and beyond.

## Training a deep network in practice

**Weight initialisation** matters because bad initial weights cause vanishing or exploding activations before learning even starts. **Xavier/Glorot** initialisation suits sigmoid/tanh layers; **He** initialisation suits ReLU layers. Both scale the initial weight variance to keep the variance of activations and gradients roughly constant across layers.

**Hyperparameter optimisation** — learning rate and its schedule, optimiser choice, batch size, number and width of layers, filter sizes, pooling strategy, dropout rate, L2 strength — is usually done by grid search, random search, or Bayesian optimisation, evaluated on a validation set.

## Generative adversarial networks

A **GAN** trains two networks against each other:

- **Generator** $G$ — maps random noise to a fake sample (an image, say) meant to look real.
- **Discriminator** $D$ — a classifier that sees real and fake samples and outputs the probability that each is real.

$G$ is trained to fool $D$; $D$ is trained to catch $G$. It's a minimax game, and as it proceeds $G$ produces increasingly realistic samples while $D$ gets sharper. GANs generate strikingly realistic images but are notoriously unstable to train.

## Semi-supervised learning

Deep networks are **data-hungry**, and labels are the expensive part. Semi-supervised learning uses a lot of unlabelled data alongside a little labelled data:

- **Pre-train representations** on the unlabelled data — with an [autoencoder](/citadel/artificial-intelligence/dimesionality-reduction), or a **self-supervised** pretext task (predict a masked image patch, predict the next word) — then **fine-tune** on the small labelled set.
- **Pseudo-labelling** — label the unlabelled data with a model trained on the labelled data, then retrain on both.
- **Consistency regularisation** — require the model's prediction to stay stable when the input is perturbed.

## The one idea to keep

Depth was never blocked by theory — it was blocked by gradients that died before they reached the early layers, and by not enough data or compute to fit the parameters. ReLU, good initialisation, batch normalisation, and residual connections fixed the gradient flow; ImageNet and GPUs supplied the rest. What depth buys, once you can train it, is a feature hierarchy that represents compositional data with far fewer parameters than any shallow network could.
