---
title: An Introduction to Deep Learning - From Linear Models to Neural Networks
description: Embark on a journey from foundational machine learning concepts and linear models like SVMs, Perceptrons, and Logistic Regression, to the basics of Neural Networks, how they learn via backpropagation and SGD, and their power as universal function approximators.
date: 2023-07-10
draft: false
slug: /pensieve/ai-ml/deep-learning
tags:
  - AI/ML
  - CS Basics
---

Hello again, AI explorers and data scientists! We previously sketched the path from foundational machine learning to the exciting realm of Deep Learning. Now, as requested, we're going to retrace those steps with a more discerning eye, delving deeper into the mathematical underpinnings of **Linear Models** and the fundamental mechanisms of **Shallow Neural Networks** – the true precursors to today's complex deep architectures.

This post will provide a more rigorous look at:
* Linear models like the Perceptron, Logistic Regression, and Support Vector Machines (from a linear classification viewpoint).
* The computational power of a shallow neural network.
* The mathematics of training networks: loss functions, the backpropagation algorithm in more detail, and stochastic gradient descent.
* The significance of neural networks as universal function approximators.

Fasten your seatbelts for a more mathematical journey!

---
## A Quick Recap: Machine Learning Fundamentals (Revisited)

Before we dive into the specifics, let's briefly reiterate that Machine Learning (ML) is a subfield of AI where systems learn from data (Experience E) to improve their performance (Measure P) on specific tasks (Task T) without being explicitly programmed for every scenario. Our focus remains largely within **Supervised Learning**, where models learn from labeled data to perform classification or regression.

---
## Starting Simple: Linear Models as a Foundation
Linear models, despite their simplicity, are foundational to understanding more complex neural networks. They assume a linear relationship between input features and the output, or a linear decision boundary in the case of classification.

### The Perceptron: An Early Neuron Model

The Perceptron, conceived by Frank Rosenblatt in the 1950s, is one of the earliest operational models of a single artificial neuron.
* **Computation**: Given an input vector $\mathbf{x} = [x_1, x_2, \dots, x_n]^T$ and a weight vector $\mathbf{w} = [w_1, w_2, \dots, w_n]^T$, the Perceptron first computes a weighted sum of its inputs, often including a bias term $b$ (which can be represented as $w_0 x_0$ where $x_0=1$). The net input $z$ is:
    $z = \mathbf{w}^T \mathbf{x} + b = \sum_{i=1}^{n} w_i x_i + b$
    This net input is then passed through a **step function** (or Heaviside step function) as its activation function, $\phi(z)$:
    $\phi(z) = \begin{cases} 1 & \text{if } z \ge \theta \\ 0 & \text{if } z < \theta \end{cases}$
    Often, the threshold $\theta$ is incorporated into the bias, so the rule becomes $\phi(z) = 1$ if $z \ge 0$ and $0$ otherwise (or $1$ and $-1$ for bipolar outputs).
* **Learning (Perceptron Learning Rule)**: The Perceptron learns by adjusting its weights and bias iteratively. For a given training example $(\mathbf{x}^{(j)}, t^{(j)})$, where $t^{(j)}$ is the target output and $y^{(j)}$ is the Perceptron's output:
    The weights are updated as: $w_i(\text{new}) = w_i(\text{old}) + \eta (t^{(j)} - y^{(j)}) x_i^{(j)}$
    The bias is updated as: $b(\text{new}) = b(\text{old}) + \eta (t^{(j)} - y^{(j)})$
    Where $\eta$ is the learning rate (a small positive constant). The updates only occur if there is a misclassification ($t^{(j)} \neq y^{(j)}$).
* **Linear Separability**: The Perceptron convergence theorem states that if the training data is linearly separable, the Perceptron learning algorithm is guaranteed to find a weight vector that correctly classifies all training examples in a finite number of steps. However, it can only solve problems where the classes are linearly separable by a hyperplane. It cannot solve non-linear problems like XOR.

### Logistic Regression: Probabilistic Linear Classification

Logistic Regression is a linear model adapted for **binary classification** tasks, where the goal is to predict a probability that an instance belongs to a particular class.
* **Core Idea**: It models the logarithm of the odds (log-odds or logit) of the positive class as a linear function of the input features:
    $\ln\left(\frac{P(Y=1|\mathbf{x})}{1-P(Y=1|\mathbf{x})}\right) = \mathbf{w}^T \mathbf{x} + b$
    Solving for $P(Y=1|\mathbf{x})$, we get the **logistic (or sigmoid) function**:
    $P(Y=1|\mathbf{x}) = h_{\mathbf{w}}(\mathbf{x}) = \frac{1}{1 + e^{-(\mathbf{w}^T \mathbf{x} + b)}}$.
    This function maps the linear output $z = \mathbf{w}^T \mathbf{x} + b$ to a probability between 0 and 1.
* **Learning**: The parameters ($\mathbf{w}$, $b$) are typically estimated using **Maximum Likelihood Estimation (MLE)**. The likelihood function for $N$ independent training examples $(\mathbf{x}^{(j)}, y^{(j)})$ is:
    $L(\mathbf{w}, b) = \prod_{j=1}^{N} [h_{\mathbf{w}}(\mathbf{x}^{(j)})]^{y^{(j)}} [1 - h_{\mathbf{w}}(\mathbf{x}^{(j)})]^{1-y^{(j)}}$
    It's often easier to maximize the log-likelihood:
    $\ell(\mathbf{w}, b) = \sum_{j=1}^{N} [y^{(j)} \log h_{\mathbf{w}}(\mathbf{x}^{(j)}) + (1-y^{(j)}) \log(1 - h_{\mathbf{w}}(\mathbf{x}^{(j)}))]$
    This function is convex, and its negative (the cross-entropy loss or log loss) can be minimized using gradient descent or other optimization algorithms. The partial derivative of the negative log-likelihood with respect to a weight $w_i$ is:
    $\frac{\partial J}{\partial w_i} = \sum_{j=1}^{N} (h_{\mathbf{w}}(\mathbf{x}^{(j)}) - y^{(j)}) x_i^{(j)}$ (where $J = -\ell(\mathbf{w},b)$).
* **Decision Boundary**: Although the output is a probability transformed by a non-linear sigmoid function, the decision boundary (where $P(Y=1|\mathbf{x}) = 0.5$, i.e., $\mathbf{w}^T \mathbf{x} + b = 0$) is linear in the feature space $\mathbf{x}$.

### Support Vector Machines (SVMs): Maximizing the Margin

Support Vector Machines (SVMs) are primarily known as powerful classifiers that can also perform linear classification.
* **Linear SVMs**: For a linearly separable dataset, an SVM finds the **optimal separating hyperplane** which is defined as the hyperplane that maximizes the **margin** between the two classes. The margin is the distance from the hyperplane to the closest data points from either class. These closest data points are called **support vectors**.
* **Mathematical Formulation (Simplified for separable case)**:
    The hyperplane is defined as $\mathbf{w} \cdot \mathbf{x} - b = 0$. We want to find $\mathbf{w}$ and $b$ such that for all training data $(\mathbf{x}_i, y_i)$ where $y_i \in \{-1, 1\}$:
    $y_i(\mathbf{w} \cdot \mathbf{x}_i - b) \ge 1$.
    The margin is $\frac{2}{\|\mathbf{w}\|}$. Maximizing the margin is equivalent to minimizing $\frac{1}{2}\|\mathbf{w}\|^2$.
    So, the optimization problem is:
    Minimize $\frac{1}{2}\|\mathbf{w}\|^2$ subject to $y_i(\mathbf{w} \cdot \mathbf{x}_i - b) \ge 1$ for all $i$.
    This is a convex quadratic programming problem.
* **Non-linear SVMs**: For data that is not linearly separable, SVMs can use the **kernel trick** to map data to a higher-dimensional space where a separating hyperplane might be found. This allows SVMs to learn non-linear decision boundaries.

---
## Introduction to Neural Networks: Beyond Linearity with Shallow Networks

Linear models, while foundational, have limitations in capturing complex, non-linear relationships in data. Neural networks, even simple **shallow networks** (typically one hidden layer), offer a way to overcome these limitations.

### What a Shallow Network Computes: Learning Non-Linear Mappings

A shallow neural network, or a Multilayer Perceptron (MLP) with a single hidden layer, can be seen as a two-stage computation:
1.  **Hidden Layer Transformation**: Each neuron in the hidden layer receives the input vector $\mathbf{x}$. It computes a weighted sum of its inputs and applies a **non-linear activation function** $\phi_h$ (e.g., sigmoid, tanh, ReLU).
    For hidden neuron $j$, its activation $h_j$ is:
    $h_j = \phi_h(\mathbf{w}_{j}^{(h)T} \mathbf{x} + b_j^{(h)})$
    where $\mathbf{w}_{j}^{(h)}$ and $b_j^{(h)}$ are the weights and bias for hidden neuron $j$.
    This layer transforms the original input space $\mathbf{x}$ into a new feature space represented by the activations of the hidden neurons $\mathbf{h} = [h_1, h_2, \dots, h_K]^T$ (where K is the number of hidden neurons). Each hidden neuron can learn to respond to different patterns or features in the input data.
2.  **Output Layer Computation**: The output layer neuron(s) take the activations from the hidden layer $\mathbf{h}$ as input. They compute a weighted sum of these hidden activations and apply an output activation function $\phi_o$ suitable for the task.
    For a single output neuron (e.g., for regression or binary classification):
    $y_{out} = \phi_o(\mathbf{w}^{(o)T} \mathbf{h} + b^{(o)})$
    where $\mathbf{w}^{(o)}$ and $b^{(o)}$ are the weights and bias for the output neuron.

The crucial element here is the non-linear activation function in the hidden layer. Without it, a multi-layer network would simply compute a linear function of the input, no more powerful than a single-layer perceptron. With non-linear activations, the hidden layer learns to transform the input data into a representation where the problem might become linearly separable (or easier to model linearly) for the output layer. This allows even a shallow MLP to learn complex, non-linear decision boundaries or approximate non-linear functions.

---
## Training a Neural Network: The Learning Process ️

Training a neural network means finding the optimal values for its weights and biases such that it performs a desired task accurately. This is achieved by minimizing a **loss function**.

### Loss Functions: Quantifying Error

A **loss function** (or cost function) measures the discrepancy between the network's predictions ($\hat{y}$) and the true target values ($y$) from the training data. The choice of loss function depends on the task:
* **Mean Squared Error (MSE)**: Commonly used for regression tasks. For $N$ training examples:
    $L = \frac{1}{N} \sum_{j=1}^{N} (y^{(j)} - \hat{y}^{(j)})^2$.
* **Cross-Entropy Loss (or Log Loss)**: Commonly used for classification tasks, especially when the network outputs probabilities. For binary classification with true label $y \in \{0,1\}$ and predicted probability $\hat{y} = P(Y=1|\mathbf{x})$:
    $L = - \frac{1}{N} \sum_{j=1}^{N} [y^{(j)} \log(\hat{y}^{(j)}) + (1-y^{(j)}) \log(1-\hat{y}^{(j)})]$.
    For multi-class classification with one-hot encoded true labels $\mathbf{y}$ and predicted probabilities $\hat{\mathbf{y}}$ (from a softmax output layer):
    $L = - \frac{1}{N} \sum_{j=1}^{N} \sum_{k=1}^{C} y_k^{(j)} \log(\hat{y}_k^{(j)})$, where $C$ is the number of classes.

The goal of training is to find the network parameters (weights and biases) that minimize this loss function.

### Backpropagation Algorithm: The Engine of Learning

The **Backpropagation algorithm** is the cornerstone for training multi-layered neural networks by efficiently computing the gradients of the loss function with respect to all network parameters. It relies on the chain rule of calculus.

1.  **Forward Pass**:
    * An input pattern $\mathbf{x}$ is fed into the network.
    * Activations are propagated forward through each layer:
        * For a neuron $j$ in layer $l$, its net input is $z_j^{(l)} = \sum_i w_{ji}^{(l)} a_i^{(l-1)} + b_j^{(l)}$.
        * Its activation is $a_j^{(l)} = \phi(z_j^{(l)})$, where $\phi$ is the activation function.
    * This continues until the final output $\hat{\mathbf{y}} = \mathbf{a}^{(L)}$ (where $L$ is the output layer) is computed.
    * The loss $L( \mathbf{y}, \hat{\mathbf{y}})$ is calculated.

2.  **Backward Pass (Error Propagation)**:
    * **Output Layer Error ($\delta^{(L)}$)**: The error term for each neuron $j$ in the output layer $L$ is calculated. If using MSE loss and the output activation is linear, $\delta_j^{(L)} = (\hat{y}_j - y_j) \phi'(z_j^{(L)})$. If using cross-entropy loss with sigmoid/softmax, the expression often simplifies. For example, with sigmoid and cross-entropy, $\delta_j^{(L)} = (\hat{y}_j - y_j)$.
    * **Propagate Error to Hidden Layers**: The error term for each neuron $j$ in a hidden layer $l$ is calculated based on the error terms of the neurons in the next layer ($l+1$) it connects to:
        $\delta_j^{(l)} = \left( \sum_k w_{kj}^{(l+1)} \delta_k^{(l+1)} \right) \phi'(z_j^{(l)})$
        where $w_{kj}^{(l+1)}$ are weights from neuron $j$ in layer $l$ to neuron $k$ in layer $l+1$.
    * **Compute Gradients**: The gradients of the loss function with respect to each weight $w_{ji}^{(l)}$ and bias $b_j^{(l)}$ are:
        $\frac{\partial L}{\partial w_{ji}^{(l)}} = \delta_j^{(l)} a_i^{(l-1)}$
        $\frac{\partial L}{\partial b_j^{(l)}} = \delta_j^{(l)}$

Activation functions must be differentiable for backpropagation (e.g., sigmoid, tanh, ReLU ).

### Stochastic Gradient Descent (SGD) and its Variants: Finding the Minimum

Once gradients are computed, an optimization algorithm updates the parameters to minimize the loss. **Gradient Descent** is the most common.
* **Parameter Update Rule**:
    $w_{ji}^{(l)}(\text{new}) = w_{ji}^{(l)}(\text{old}) - \eta \frac{\partial L}{\partial w_{ji}^{(l)}}$
    $b_j^{(l)}(\text{new}) = b_j^{(l)}(\text{old}) - \eta \frac{\partial L}{\partial b_j^{(l)}}$
    where $\eta$ is the **learning rate**.
* **Types of Gradient Descent**:
    * **Batch Gradient Descent**: Gradients are computed by summing/averaging over the *entire* training dataset before one parameter update.
    * **Stochastic Gradient Descent (SGD)**: Parameters are updated after processing *each individual training example*. The "stochastic" nature (noisy gradients) can help escape shallow local minima.
    * **Mini-Batch SGD**: A compromise, updating parameters after processing a small "mini-batch" of training examples. This is the most common approach in deep learning as it balances computational efficiency with stable convergence.
* Modern deep learning often uses more advanced optimizers like Adam, RMSprop, Adagrad, which adapt the learning rate for each parameter or use momentum to accelerate convergence.

---
## Neural Networks as Universal Function Approximators
A remarkable theoretical result underpinning the power of neural networks is the **Universal Approximation Theorem**. It states that:
* A feedforward neural network with a **single hidden layer** containing a finite number of neurons, using **non-linear activation functions** (e.g., sigmoid functions, though other types like ReLU also work under certain conditions), can approximate any continuous function on compact subsets of $\mathbb{R}^n$ to any desired degree of accuracy, provided there are enough hidden neurons.

**Implications**:
* This means that even a relatively simple (shallow) neural network is theoretically capable of learning an incredibly wide range of complex input-output mappings.
* It doesn't tell us *how* to find the weights (that's the job of training algorithms like backpropagation) or *how many* neurons are needed for a specific problem, but it provides a strong theoretical justification for the expressive power of neural networks.
* While a single hidden layer is *sufficient*, deep networks (with multiple hidden layers) are often found to be more efficient in practice. They can learn hierarchical representations of features, often requiring fewer parameters overall to approximate complex functions compared to very wide shallow networks. This efficiency and ability to learn feature hierarchies are key motivations for **Deep Learning**.

---
## Conclusion: Laying the Groundwork for Deep Exploration

From the foundational principles of linear models like Perceptrons, Logistic Regression, and SVMs, we've transitioned to the more powerful and flexible paradigm of neural networks. Understanding how even a shallow network with a single hidden layer can compute non-linear functions, and the core training mechanisms of loss functions, backpropagation, and stochastic gradient descent, is essential. The Universal Approximation Theorem assures us of their vast potential.

These concepts are not just stepping stones; they are the active components and theoretical underpinnings upon which the more complex and often revolutionary architectures of Deep Learning are built. Our next journey will take us deeper into these multi-layered marvels!
