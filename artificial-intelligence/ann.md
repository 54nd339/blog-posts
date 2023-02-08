---
title: Artificial Neural Networks - The Classical Model Zoo
description: "The neural networks that predate deep learning - the artificial neuron and the perceptron, ADALINE's delta rule, multilayer perceptrons trained by backpropagation, and the specialised architectures: radial-basis-function networks, Kohonen self-organising maps, learning vector quantisation, Hebbian learning, and Hopfield associative memory."
date: 2023-02-08
draft: false
slug: /artificial-intelligence/ann
tags:
  - Artificial Intelligence
  - Machine Learning
  - Deep Learning
---

An artificial neuron does one thing: it takes a weighted sum of its inputs and passes it through a function that decides how strongly to fire. Wire a few thousand of them together, adjust the weights from examples, and the network can approximate almost any input-output mapping. That idea is decades old, and before deep learning consolidated everything around large multilayer networks trained by backpropagation, it produced a zoo of distinct architectures — each with its own learning rule and its own job. This post is a tour of that zoo.

## From biological to artificial neurons

A biological neuron has **dendrites** that receive signals, a **soma** that integrates them, an **axon** that carries the output, and **synapses** where it meets the next neuron's dendrites. An incoming signal that pushes the soma past a threshold triggers a spike down the axon.

The artificial version keeps the structure:

- **Inputs** $x_1, \dots, x_n$ — the incoming signals.
- **Weights** $w_1, \dots, w_n$ — synapse strengths; a large weight is a strong connection.
- **Summation** — the total input, $\sum_i w_i x_i$, usually plus a **bias** $b$.
- **Activation function** $\varphi$ — maps the sum to the output, deciding if and how the neuron fires. The simplest is a threshold: output 1 if the sum exceeds $\theta$, else 0.

$$y = \varphi\!\left(\sum_{i=1}^{n} w_i x_i - \theta\right)$$

Networks of these units map input patterns to output patterns, learn from examples, generalise to unseen inputs, and degrade gracefully when part of the network is damaged or the input is noisy.

## The perceptron

The **perceptron**, from Frank Rosenblatt, is a single neuron with a step (or signum) activation producing a binary output. It learns by the **perceptron learning rule**: for a training example $(\mathbf{x}, t)$ with the perceptron's output $y$, and only when $y \ne t$,

$$w_i \leftarrow w_i + \alpha\,(t - y)\,x_i, \qquad b \leftarrow b + \alpha\,(t - y)$$

with learning rate $\alpha$. The **perceptron convergence theorem** guarantees that if the data is **linearly separable** — separable by a hyperplane — this rule finds a separating weight vector in finitely many steps. If it isn't (the XOR function is the standard counterexample), the perceptron never converges. It's the [linear classifier](/citadel/artificial-intelligence/ml) in its earliest form.

## Multilayer perceptrons

To get past linear separability you stack layers: an **input layer**, one or more **hidden layers**, and an **output layer**. The hidden layers transform the input into a representation where the problem becomes linearly separable for the output layer, so an MLP can learn non-linear boundaries and approximate non-linear functions. An architecture with $l$ inputs, hidden layers of $m_1, m_2, \dots$ neurons, and $n$ outputs is written $l\text{-}m_1\text{-}m_2\text{-}\dots\text{-}n$. Hidden and output neurons need **differentiable non-linear** activations (sigmoid, tanh) so the network can be trained by backpropagation.

## ADALINE and MADALINE

**ADALINE** (Adaptive Linear Neuron), from Widrow and Hoff around 1960, differs from the perceptron in *when* it measures error. It updates weights from the **linear** output $y_{in} = b + \sum_i w_i x_i$, *before* any thresholding, using the **delta rule** (also called Widrow-Hoff or least-mean-squares):

$$w_j \leftarrow w_j + \alpha\,(t - y_{in})\,x_j, \qquad b \leftarrow b + \alpha\,(t - y_{in})$$

This minimises the mean squared error between $y_{in}$ and the target, a smoother objective than the perceptron's discrete error. A threshold is applied to $y_{in}$ afterward for classification.

**MADALINE** ("Many ADALINEs") stacks them. The original **Rule I** fed a layer of ADALINEs into a fixed logic gate (AND, OR, or majority vote) that wasn't trained. **Rule II** made multilayer ADALINE networks trainable, by tentatively flipping weights and keeping changes that reduced the error — a trial-and-error precursor to backpropagation.

## Backpropagation

**Backpropagation** is the standard training algorithm for MLPs: supervised gradient descent that uses the chain rule to compute the error gradient for every weight. Two passes:

1. **Forward pass** — feed an input through; each neuron $j$ computes $I_j = \sum_i w_{ij} o_i$ and $o_j = f(I_j)$ with a differentiable $f$, up to the output.
2. **Backward pass** — compare the output to the target and propagate the error back. For an output neuron $k$,

$$\delta_k = (t_k - o_k)\,f'(I_k)$$

   and for a hidden neuron $j$ feeding neurons $k$ in the next layer,

$$\delta_j = \left(\sum_k \delta_k\,w_{jk}\right) f'(I_j)$$

   Each weight then updates by $\Delta w_{ij} = \eta\,\delta_j\,o_i$.

The requirement that $f$ be differentiable is why sigmoid and tanh replaced the step function. The full layer-by-layer derivation, the loss functions, the stochastic and mini-batch variants, and the universal approximation theorem are in [an introduction to deep learning](/citadel/artificial-intelligence/dl).

## Activation functions

| Function | Formula | Range |
| --- | --- | --- |
| Sigmoid | $f(z) = \dfrac{1}{1 + e^{-z}}$ | $(0, 1)$ |
| Tanh | $f(z) = \dfrac{e^z - e^{-z}}{e^z + e^{-z}}$ | $(-1, 1)$ |
| ReLU | $f(z) = \max(0, z)$ | $[0, \infty)$ |
| Leaky ReLU | $f(z) = z$ if $z \ge 0$, else $a z$ (small $a > 0$) | $(-\infty, \infty)$ |

Sigmoid and tanh are the classic MLP choices; ReLU and its variants became standard for deep networks because they don't saturate for positive inputs, which keeps gradients alive through many layers.

## Radial-basis-function networks

An **RBF network** is a three-layer feedforward net with a different kind of hidden unit. Each hidden neuron $j$ has a **centre** $\mathbf{c}_j$ and a **width** $\sigma_j$, and responds to how close the input is to its centre — typically a Gaussian:

$$\phi_j(\mathbf{x}) = \exp\!\left(-\frac{\lVert \mathbf{x} - \mathbf{c}_j \rVert^2}{2\sigma_j^2}\right)$$

The response is **local**: strong near the centre, near zero far away. The output layer is linear, $y_k = \sum_j w_{kj}\,\phi_j(\mathbf{x}) + b_k$.

Training is two-stage: first fix the hidden parameters — pick centres by sampling the data or by **k-means clustering**, set widths heuristically (say, the average nearest-neighbour distance) — then, with the hidden layer frozen, solve the linear output weights in one step by least squares (the pseudo-inverse). RBF networks are strong at function approximation and classification where the input-output relationship is localised.

## Self-organising maps

A **Kohonen self-organising map** (SOM) is an *unsupervised* network for dimensionality reduction, clustering, and visualisation. The output neurons sit on a grid (usually 2D), each carrying a weight vector the size of the input. Training is **competitive**:

1. **Competition** — for an input, find the neuron whose weight vector is closest: the **best matching unit** (BMU).
2. **Cooperation** — the BMU and its grid neighbours within a radius are selected; the radius shrinks over time.
3. **Adaptation** — move the BMU and its neighbours' weights toward the input, more for the BMU, less for distant neighbours, with a learning rate that also decays.

Over many inputs the grid arranges itself so that similar inputs activate nearby neurons — a topology-preserving map of the data. It sits alongside k-means in [unsupervised learning](/citadel/artificial-intelligence/unspervised-learning).

## Learning vector quantisation

**LVQ** is SOM's supervised sibling, for classification. It keeps a set of **codebook vectors**, each labelled with a class. For a labelled training input, find the nearest codebook vector; if its class *matches* the input's, move it toward the input; if it *doesn't*, move it away. This pushes the codebook vectors into positions that carve out clean class regions.

## Hebbian learning

The oldest rule, from Donald Hebb: **neurons that fire together, wire together**. It's unsupervised — strengthen a connection when both its neurons are active:

$$\Delta w_{ij} = \eta\,x_i\,y_j$$

For a set of input-output pattern pairs $(X_k, Y_k)$ the whole weight matrix can be assembled at once as a correlation sum, $W = \sum_k X_k Y_k^{\mathsf T}$.

## Hopfield networks

A **Hopfield network** is recurrent: every neuron connects to every other (never itself), the weights are symmetric ($w_{ij} = w_{ji}$), and states are binary or bipolar ($\pm 1$).

Its use is **associative memory**. Store a set of patterns by setting the weights with a Hebbian rule; then present a noisy or partial pattern and let the neurons update repeatedly until the state stops changing — it converges to the stored pattern closest to the input. Hopfield defined an **energy function** for the network whose value never increases under the update dynamics, so the network always settles into a local energy minimum, and the stored patterns are engineered to be those minima. The same mechanism solves certain optimisation problems: encode the problem so its good solutions are energy minima, and let the network relax.

## The one idea to keep

Every model here is the same neuron — weighted sum, activation — arranged and trained differently. The perceptron and ADALINE differ only in where they measure error; backpropagation is the delta rule pushed through hidden layers by the chain rule; RBF networks swap a distance-to-centre response for the weighted sum; SOMs and Hopfield nets drop supervision entirely and let competition or energy minimisation do the organising. Deep learning is what happened when the MLP-plus-backprop branch got much bigger and much better at staying trainable.
