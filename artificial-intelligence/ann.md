---
title: Artificial Neural Networks - Unveiling the Digital Brain
description: A comprehensive exploration of Artificial Neural Networks (ANNs), from their biological inspiration and foundational concepts like Perceptrons and MLPs, to advanced architectures and learning paradigms like Backpropagation, RBFs, SOMs, and Hebbian Learning.
date: 2023-06-27
draft: false
slug: /pensieve/ai-ml/artificial-neural-networks
tags:
  - AI/ML
  - CS Basics
---

Welcome back, AI explorers! In our previous discussion on Computational Intelligence, we touched upon Artificial Neural Networks (ANNs) as one of its key pillars. Today, we're zooming in on these fascinating systems that are designed to mimic the learning capabilities of the human brain. From the simplest Perceptron to complex architectures trained by Backpropagation, and even networks that organize themselves, ANNs are at the forefront of many AI breakthroughs. Let's unravel the world of ANNs!

---
## Introduction to Artificial Neural Networks (ANNs): The Biological Spark

Artificial Neural Networks (ANNs), or simply Neural Networks (NNs), are computational models inspired by the structure and function of biological nervous systems. The human brain, a highly complex, interconnected network of processing units called **neurons**, serves as the primary inspiration. Each biological neuron is approximately $10\mu m$ long and operates in parallel with billions of others. These neurons communicate via electrical impulses.

A biological neuron typically has:
* **Dendrites**: Bushy, thin fibers that receive input signals from other neurons.
* **Soma (Cell Body)**: Processes these signals, much like a cell's nucleus.
* **Axon**: A long cylindrical fiber that transmits the neuron's output signal.
* **Synapse**: The junction where an axon makes contact with the dendrites of neighboring neurons, transmitting signals via chemicals called neurotransmitters.

An incoming signal might generate an electrical impulse in the axon if it crosses a certain threshold value. This behavior of biological neural networks can be captured by simplified artificial models.

An ANN, therefore, is a simplified imitation of this biological system, designed to perform tasks like computation, signal transportation, information storage, perception, and learning through automatic training. Every component in an artificial neuron model bears a direct analogy to its biological counterpart.

The basic structure of an artificial neuron (or node) involves:
* **Inputs ($x_1, x_2, \dots, x_n$)**: These are the signals received by the neuron, analogous to signals received by dendrites.
* **Weights ($w_1, w_2, \dots, w_n$)**: Each input is associated with a weight, which models the strength of a synapse. An effective synapse transmitting a stronger signal will have a larger weight.
* **Summation Function**: This function calculates the total input to the neuron by summing the weighted inputs, often including a **bias** term ($b_j$). The total input $I$ (or net input $n$) to a neuron $j$ can be expressed as:
    $I = \sum_{i=1}^{n} w_i x_i$  or more generally, $x_j = \sum_{k=1}^{m} x_k w_{k,j} + b_j$.
* **Activation Function (or Transfer Function, $\phi()$ or $f()$)**: The summed input is passed through an activation function to produce the neuron's final output. This function determines if and how the neuron "fires". A common type is the **thresholding function**, where if the sum $I$ is greater than a threshold $\theta$, the output is 1, otherwise 0.
    $y = \phi(\sum_{i=1}^{n} w_i x_i - \theta)$.
* **Output ($y$ or $a$)**: The final value produced by the neuron after applying the activation function.

ANNs exhibit mapping capabilities (input patterns to output patterns), learn by example, and possess the ability to generalize to new, untrained data. They are also robust, fault-tolerant, and can process information in parallel.

---
## Perceptrons and Multilayer Perceptrons (MLPs): From Simple Classifiers to Complex Problem Solvers

### The Perceptron: A Building Block

The **Perceptron**, developed by Frank Rosenblatt, is one of the earliest and simplest types of ANNs, often consisting of a single neuron. It typically uses a **step function** or **hard-limit transfer function** as its activation function to produce a binary output.
* **Step Function (Hard Limit)**:
    $f(z) = \begin{cases} 1 & \text{if } z \ge \theta \\ 0 & \text{if } z < \theta \end{cases}$
    Or, commonly, if the net input $n \ge 0$, output is 1, else 0.
* **Signum (Sign) Function (Symmetrical Hard Limit)**:
    $f(z) = \begin{cases} 1 & \text{if } z > 0 \\ -1 & \text{if } z < 0 \end{cases}$ (often with a case for $z=0$).

Perceptrons can learn to classify linearly separable patterns. The **Perceptron learning rule** involves adjusting the weights ($w_i$) and bias ($b$) based on the error between the actual output and the target output. If the output is incorrect for a given input pattern:
* If output is 1 instead of 0 (for binary 0/1 output): $w_i(new) = w_i(old) - \alpha x_i$; $b(new) = b(old) - \alpha$.
* If output is 0 instead of 1: $w_i(new) = w_i(old) + \alpha x_i$; $b(new) = b(old) + \alpha$.
Where $\alpha$ is the learning rate.

### Multilayer Perceptrons (MLPs): Tackling Complexity

A single perceptron can only solve linearly separable problems. For more complex, non-linearly separable problems (like the XOR problem), we need **Multilayer Perceptrons (MLPs)**. MLPs consist of:
* An **input layer** that receives the input signals.
* One or more **hidden layers** of neurons. These layers enable the network to learn complex features and non-linear mappings from the input data. The computations in hidden layers are intermediate steps before the output is produced.
* An **output layer** that produces the final result of the network.

The architecture of an MLP with $l$ input neurons, $p$ hidden layers with $m_1, m_2, \dots, m_p$ neurons respectively, and $n$ output neurons is often denoted as $l-m_1-m_2-\dots-m_p-n$. The connections between layers have associated weights, and neurons in hidden and output layers have activation functions, which are typically non-linear and differentiable (like sigmoid or tanh) to allow for training with algorithms like backpropagation.

---
## Adaline and Madaline: Early Adaptive Neurons

### ADALINE (Adaptive Linear Neuron)

Developed by Bernard Widrow and Tedd Hoff around 1960, **ADALINE** is an early single-layer neural network.
* It uses a **linear activation function** during the weight adjustment process, meaning its output (before thresholding for classification) is a linear combination of its inputs: $y_{in} = b + \sum x_i w_i$.
* The learning rule used is the **Widrow-Hoff learning rule**, also known as the **Delta Rule** or the **Least Mean Squares (LMS)** algorithm. This rule adjusts the weights and bias to minimize the mean squared error between the linear output ($y_{in}$) and the target value ($t$).
* For a single output neuron, the weight update for weight $w_j$ is given by:
    $w_j(new) = w_j(old) + \alpha (t - y_{in}) x_j$
    And the bias update is:
    $b(new) = b(old) + \alpha (t - y_{in})$
    Where $\alpha$ is the learning rate.
* Although the weights are updated based on the linear output, a threshold function (like a step function) can be applied to $y_{in}$ to produce a final binary output for classification tasks after training.

### MADALINE (Multiple Adaptive Linear Neurons)

**MADALINE** stands for Multiple ADALINE. It's an architecture that uses multiple Adaline units.
* The earliest form, **Madaline Rule I (MRI)**, consisted of a single layer of Adaline units whose outputs were then fed into a fixed logic device (like an AND, OR, or MAJORITY voter) to produce the final output. This output layer was not adaptive.
* Later versions like Madaline Rule II (MRII) introduced methods for adapting weights in multilayer networks of Adalines, making them more powerful. The training for MRII was more complex, involving a trial-and-error process where weights were tentatively flipped and changes kept if they reduced the error.

---
## Backpropagation Multilayer Perceptrons (BPMLP): The Workhorse of ANNs

**Backpropagation** is the most widely used algorithm for training Multilayer Perceptrons. It's a supervised learning algorithm that uses **gradient descent** to minimize the error between the network's output and the target values.

The process involves two main passes:
1.  **Forward Pass**: An input pattern is presented to the network. The activations flow forward from the input layer, through the hidden layer(s), to the output layer, producing an output for each neuron. For a neuron $j$ in a layer, its input $I_j$ is $\sum_i w_{ij} o_i$ (where $o_i$ is output from neuron $i$ in the previous layer), and its output $o_j$ is $f(I_j)$, where $f$ is a differentiable activation function like sigmoid or tanh.
2.  **Backward Pass (Error Propagation)**:
    * The network's output is compared to the target output, and an error is calculated (commonly Mean Squared Error).
    * This error is then propagated backward through the network, from the output layer to the input layer.
    * The algorithm computes the gradient of the error with respect to each weight in the network using the chain rule of differentiation.
        For a weight $w_{jk}$ connecting neuron $j$ (in a hidden layer) to neuron $k$ (in the output layer), the error term for neuron $k$ ($\delta_k$) is typically $(t_k - o_k) f'(I_k)$, and for neuron $j$, $\delta_j = (\sum_k \delta_k w_{jk}) f'(I_j)$.
    * **Weight Update**: Each weight $w_{ij}$ is updated to reduce the error, typically using the delta rule:
        $\Delta w_{ij} = \eta \cdot \delta_j \cdot o_i$
        where $\eta$ is the learning rate, $\delta_j$ is the error term of neuron $j$, and $o_i$ is the output of neuron $i$ from the previous layer.
        The updated weight is $w_{ij}(new) = w_{ij}(old) + \Delta w_{ij}$.

This iterative process of forward and backward passes continues over many training examples (epochs) until the network's error reaches an acceptable level or a stopping criterion is met.

**Key Requirements for Backpropagation**:
* The activation functions used by the neurons must be **differentiable**. Common choices include:
    * **Sigmoid (Logistic) Function**: $f(z) = \frac{1}{1+e^{-z}}$ or $f(z) = \frac{e^z}{1+e^z}$. Range: (0, 1).
    * **Hyperbolic Tangent (tanh)**: $f(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}$. Range: (-1, 1).
    * **Rectified Linear Unit (ReLU)**: $f(z) = \max(0, z)$. Range: $[0, \infty)$. Used commonly in deep learning.
    * **Leaky ReLU**: $f(z) = \begin{cases} z & \text{if } z \ge 0 \\ az & \text{if } z < 0 \end{cases}$ (where $a$ is a small positive constant). Range: $(-\infty, \infty)$.

---
## Radial Basis Function Networks (RBFN): Localized Responses

**Radial Basis Function Networks (RBFNs or RBF networks)** are another type of feedforward neural network known for their simple structure and fast training.
* **Architecture**: Typically three layers:
    1.  **Input Layer**: Source nodes that connect to the hidden layer.
    2.  **Hidden Layer**: Composed of RBF neurons. Each RBF neuron has a "center" (a prototype vector) and a "width" (or spread). The activation of an RBF neuron is determined by the distance between the input vector and its center, typically using a Gaussian function or similar radial basis function. The neuron's response is localized, meaning it's strongest when the input is close to its center.
        A common Gaussian RBF: $\phi_j(\mathbf{x}) = \exp\left(-\frac{\|\mathbf{x} - \mathbf{c}_j\|^2}{2\sigma_j^2}\right)$, where $\mathbf{c}_j$ is the center and $\sigma_j$ is the width of the $j$-th RBF unit.
    3.  **Output Layer**: This layer is typically linear. It computes a weighted sum of the outputs from the RBF hidden neurons to produce the final network output. $y_k = \sum_j w_{kj} \phi_j(\mathbf{x}) + b_k$.
* **Training Algorithm for RBFNs**: Often a two-stage hybrid learning process:
    1.  **Determine Hidden Layer Parameters (Centers and Widths)**: This stage can be unsupervised.
        * **Centers ($\mathbf{c}_j$)**: Can be selected randomly from the training data, or determined using clustering algorithms like K-Means.
        * **Widths ($\sigma_j$)**: Can be set heuristically (e.g., based on the average distance to nearest neighbors) or using more sophisticated methods.
    2.  **Determine Output Layer Weights ($w_{kj}$)**: Once the RBF parameters are fixed, this stage is supervised. Since the output layer is linear, the weights can be found efficiently using methods like the pseudo-inverse (least squares) method.
RBFNs are particularly good at function approximation and pattern classification tasks where the input-output relationship is localized.

---
## Kohonen Self-Organizing Networks (SOMs): Mapping High-Dimensional Data

**Kohonen Self-Organizing Networks**, also known as **Self-Organizing Maps (SOMs)**, are a type of unsupervised neural network used for dimensionality reduction, clustering, and visualization of high-dimensional data. They create a low-dimensional (typically 2D) discretized representation (a "map") of the input space, preserving the topological relationships of the input data.

* **Architecture**: Consists of an input layer and an output layer of neurons arranged in a grid (e.g., 2D). Each neuron in the output map is associated with a weight vector of the same dimension as the input data.
* **Learning Process (Competitive Learning)**:
    1.  **Initialization**: Weights are initialized (e.g., randomly or to small values).
    2.  **Competition**: For each input pattern, the distance between the input vector and all neuron weight vectors is calculated. The neuron whose weight vector is closest to the input vector is declared the **winner** or **Best Matching Unit (BMU)**.
    3.  **Cooperation**: The BMU and its neighboring neurons on the map (within a certain radius) are activated. The neighborhood size often decreases over time.
    4.  **Adaptation (Weight Update)**: The weight vectors of the BMU and its neighbors are adjusted to move them closer to the input vector. The amount of adjustment is typically larger for the BMU and smaller for more distant neighbors, and it decreases over time (learning rate).

Through this iterative process, the SOM learns the structure of the input data, with similar input patterns activating neurons that are close to each other on the map.

---
## Learning Vector Quantization (LVQ): Supervised Competitive Learning

**Learning Vector Quantization (LVQ)** is a supervised learning algorithm that uses a competitive learning approach similar to SOMs but is used for classification tasks.
* It starts with a set of "codebook vectors" (prototypes), each assigned to a class.
* When a training input pattern (with a known class label) is presented:
    1.  The closest codebook vector (the winner) is found using a distance metric.
    2.  If the winner's class label **matches** the training input's class label, the winner's weight vector is moved **closer** to the input vector.
    3.  If the winner's class label **does not match**, the winner's weight vector is moved **away** from the input vector.
* LVQ effectively learns decision boundaries in the feature space based on these prototypes.

---
## Hebbian Learning: "Fire Together, Wire Together"
Hebbian learning is one of the oldest and simplest learning rules for ANNs, based on Donald Hebb's theory of synaptic plasticity in the brain. The core idea is often summarized as: **"Neurons that fire together, wire together."**
* It's typically an unsupervised learning rule.
* The weight of a connection between two neurons is strengthened if both neurons are simultaneously active.
* **Basic Hebb Rule**: The change in weight $\Delta w_{ij}$ for the connection from neuron $i$ to neuron $j$ is proportional to the product of their activations $x_i$ (input/presynaptic) and $y_j$ (output/postsynaptic):
    $\Delta w_{ij} = \eta \cdot x_i \cdot y_j$
    where $\eta$ is the learning rate.
* The weight update rule is often given as: $w_{ij}(new) = w_{ij}(old) + x_i y_j$ (assuming $\eta=1$) or more generally incorporating the learning rate.
* For a set of input-output pattern pairs $(X_k, Y_k)$, the weight matrix $W$ can be computed using a correlation matrix approach: $W = \sum_{k=1}^{p} X_k Y_k^T$.
* The Hebbian learning algorithm involves initializing weights (often to zero), setting activations for input units (often identity function), setting activations for output units, and then performing weight and bias adjustments.

---
## Hopfield Networks: Associative Memory and Optimization
**Hopfield networks** are a type of recurrent neural network with binary or bipolar ({-1, 1}) neuron states, where every neuron is connected to every other neuron (but not to itself), and the connections are symmetric ($w_{ij} = w_{ji}$).
* **Associative Memory**: They are well-known for their ability to function as content-addressable (associative) memories. A set of patterns can be "stored" in the network by setting its weights appropriately (often using a Hebbian-like rule). When presented with a noisy or incomplete version of a stored pattern, the network can iteratively update its neuron states until it converges to the closest stored pattern.
* **Energy Function**: Hopfield defined an "energy function" for the network. The network dynamics (how neuron states update) are designed such that the energy always decreases or stays the same with each update. Thus, the network always converges to a stable state, which is a local minimum of this energy function. These stable states correspond to the stored patterns.
* **Applications**: Besides associative memory, they have been used for solving certain optimization problems where the problem can be mapped onto the network's energy function.

---
## Conclusion: The Diverse World of ANNs

Artificial Neural Networks represent a vast and powerful family of models within Computational Intelligence. From the simple classification capabilities of Perceptrons to the complex function approximation of MLPs trained with Backpropagation, the localized expertise of RBFs, the unsupervised mapping of SOMs, and the associative recall of Hopfield networks, ANNs offer a rich toolkit for tackling diverse AI problems. Their ability to learn from data and adapt makes them indispensable in modern artificial intelligence.

Our journey into ANNs has just begun! There's much more to explore, including different activation functions in detail, specific training optimizations, and how these networks are applied in cutting-edge AI applications.
