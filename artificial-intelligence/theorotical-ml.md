---
title: Foundations of Machine Learning - Theoretical Limits and Bayesian Reasoning
description: A journey into the theoretical underpinnings of Machine Learning, exploring models of learnability like PAC learning and Boosting, and diving into the probabilistic world of Bayesian Learning, including Maximum Likelihood Estimates and Bayesian Belief Networks.
date: 2023-07-03
draft: false
slug: /pensieve/ai-ml/theorotical-ml
tags:
  - AI/ML
  - CS Basics
---

Hey AI enthusiasts and lifelong learners! We've explored various machine learning algorithms that empower computers to learn from data. But have you ever wondered about the fundamental limits of learning? What makes a problem learnable? How can we be confident in what our models have learned? And how can we incorporate prior knowledge and manage uncertainty in a principled way?

Today, we're venturing into two profound areas:
1.  **Theoretical Machine Learning**: We'll touch upon concepts that help us understand the *possibility* and *efficiency* of learning, such as the PAC model and Boosting.
2.  **Bayesian Learning**: We'll explore a probabilistic approach to learning that allows us to update our beliefs as we see more data, focusing on Maximum Likelihood Estimates and an introduction to Bayesian Belief Networks.

Let's explore the science behind making machines learn!

---
## Theoretical Machine Learning: Understanding Learnability
While practical algorithms are exciting, theoretical machine learning provides crucial insights into what machines can learn, how much data they need, and how efficiently they can learn it. It offers a formal way to analyze and compare learning algorithms.

### Identification in the Limit (Gold's Model)

One of the earliest formal models of learnability, particularly influential in computational linguistics and the study of language acquisition, is **identification in the limit**, proposed by E. Mark Gold in 1967.
* **Concept**: Imagine a learner being presented with an infinite sequence of examples related to a target concept (e.g., sentences from a language). After each example, the learner proposes a hypothesis (e.g., a grammar for the language). The learner is said to identify the concept "in the limit" if, after some finite number of examples, its hypothesis becomes correct and remains unchanged for all subsequent examples.
* **Relevance**: This model helps us understand which classes of concepts are learnable given an infinite stream of data and highlights the types of information (e.g., positive examples only, or both positive and negative examples) required for learning.

*(While this specific model is not detailed in the provided general AI/ML documents, it's a foundational concept in learning theory.)*

### Oracle Based Learning (e.g., Query Learning)

Another theoretical approach considers a learner that can actively interact with its environment by querying an **oracle** (an expert or a teacher).
* **Concept**: Instead of passively receiving examples, the learner can ask specific questions to gain information.
* **Types of Queries**:
    * **Membership Queries**: "Is this specific instance an example of the concept?"
    * **Equivalence Queries**: "Is my current hypothesis equivalent to the true concept?" (If not, the oracle might provide a counterexample).
* **Relevance**: Oracle-based learning models explore how active interaction and targeted questioning can make learning more efficient or even possible for concepts that are hard to learn from random examples alone.

*(Like identification in the limit, this is a fundamental theoretical model. The provided documents don't explicitly detail it, but it's key to understanding different learning paradigms.)*

### Probably Approximately Correct (PAC) Model (Valiant's Model)

The **Probably Approximately Correct (PAC)** learning model, introduced by Leslie Valiant in 1984, is a cornerstone of computational learning theory. It provides a framework for analyzing the complexity of learning algorithms in terms of the number of examples needed and the computational resources required. PAC- probably approximate correct learning in its historical overview of ML.

* **Concept**: A concept class is PAC-learnable if there exists an algorithm that, with high probability (the "Probably" part, at least $1-\delta$), outputs a hypothesis $h$ that has a low error (the "Approximately Correct" part, error at most $\epsilon$) on unseen data, provided it's given a "reasonable" number of training examples.
* **Key Parameters**:
    * $\epsilon$ (Error parameter): The maximum allowed error rate of the learned hypothesis.
    * $\delta$ (Confidence parameter): The minimum probability that the learned hypothesis will have an error rate less than $\epsilon$.
* **Focus**: The PAC model is concerned with the **efficiency** of learning, specifically:
    * **Sample Complexity**: How many training examples are needed to achieve the desired accuracy and confidence?
    * **Computational Complexity**: How much computational effort is required to learn the hypothesis?
    A concept class is efficiently PAC-learnable if both complexities are polynomial in terms of $1/\epsilon$, $1/\delta$, the size of the concept, and the dimensionality of the data.

### Boosting: Turning Weaknesses into Strength

**Boosting** is a powerful ensemble learning technique that aims to convert a collection of "weak learners" (classifiers that perform only slightly better than random guessing) into a single "strong learner" with high accuracy. The `ML.pdf` pg 3768 describes boosting as a method where "each new model is impacted by the performance of those built earlier".

* **How it Generally Works (e.g., AdaBoost - Adaptive Boosting)**:
    1.  Boosting algorithms train weak learners sequentially.
    2.  In each iteration, more weight is given to training examples that were misclassified by the preceding weak learners. This forces subsequent learners to focus on the "harder" examples.
    3.  The final strong learner is a weighted combination (e.g., a weighted vote) of all the trained weak learners, where learners that performed better on the training data are given higher weights in the final decision.
* **Goal**: Boosting algorithms are designed to reduce both **bias** (by making the model more complex through the combination of learners) and **variance** (by averaging over multiple learners).
* It's a widely used and effective technique in practice for many classification and regression tasks.

---
## Bayesian Learning: A Probabilistic Perspective
Bayesian learning provides a probabilistic approach to inference and learning. It's based on **Bayes' Theorem**, which describes how to update the probability of a hypothesis based on new evidence.

$P(H|D) = \frac{P(D|H)P(H)}{P(D)}$
Where:
* $P(H|D)$ is the **posterior probability** of hypothesis $H$ given data $D$.
* $P(D|H)$ is the **likelihood** of observing data $D$ if hypothesis $H$ is true.
* $P(H)$ is the **prior probability** of hypothesis $H$ (our belief in $H$ before seeing data).
* $P(D)$ is the **evidence** (probability of observing data $D$).

The core idea is to start with prior beliefs about hypotheses and update these beliefs as more data is observed.

### Maximum Likelihood Estimates (MLE)

A common task in machine learning is to find the parameters of a model that best fit the training data. **Maximum Likelihood Estimation (MLE)** is a method for estimating the parameters of a statistical model.
* **Goal**: To find the parameter values ($\theta$) that maximize the **likelihood function** $L(\theta|D) = P(D|\theta)$. This function represents the probability of observing the given training data $D$ if the parameters were $\theta$.
* **Process**: We search for the parameter values $\hat{\theta}_{MLE}$ that make the observed data "most probable" or "most likely."
* Often, it's analytically easier to maximize the **log-likelihood** $\ln(P(D|\theta))$, because the logarithm is a monotonically increasing function, the parameters that maximize the log-likelihood also maximize the likelihood itself.
* MLE is used, for example, in estimating the parameters of a logistic regression model.

### Parameter Estimation: Beyond MLE (Bayesian Approach)

While MLE finds parameters that maximize the data likelihood, a fully Bayesian approach to parameter estimation also incorporates **prior beliefs** about what the parameters are likely to be.

* **Maximum A Posteriori (MAP) Estimation**: This method seeks to find the parameter values $\hat{\theta}_{MAP}$ that maximize the **posterior probability** of the parameters given the data:
    $\hat{\theta}_{MAP} = \arg\max_{\theta} P(\theta|D) = \arg\max_{\theta} \frac{P(D|\theta)P(\theta)}{P(D)} = \arg\max_{\theta} P(D|\theta)P(\theta)$.
    Here, $P(\theta)$ is the prior probability distribution over the parameters.
* **Connection to MLE**: If the prior distribution $P(\theta)$ is uniform (i.e., all parameter values are considered equally likely beforehand), then MAP estimation reduces to MLE.
* **Regularization**: In many ML contexts, incorporating a prior in MAP estimation is equivalent to adding a regularization term to the objective function (as seen in Ridge or Lasso regression). This helps prevent overfitting by penalizing overly complex models or extreme parameter values.

### Bayesian Belief Networks (BBNs)

**Bayesian Belief Networks (BBNs)**, also known as Bayesian Networks or Probabilistic Directed Acyclic Graphical Models, are powerful tools for representing and reasoning about uncertainty.
* **Definition**: A BBN is a **directed acyclic graph (DAG)** where:
    * **Nodes** represent random variables (which can be discrete or continuous).
    * **Directed Edges** (arrows) represent probabilistic dependencies between variables. An arrow from node X to node Y means X has a direct influence on Y.
    * Each node is associated with a **Conditional Probability Table (CPT)** (or a conditional probability distribution for continuous variables). The CPT for a node specifies the probability distribution of that variable given the values of its parent nodes in the graph. Nodes with no parents have CPTs that represent their prior probabilities.
* **Structure**: The graph structure encodes conditional independence relationships. A variable is conditionally independent of its non-descendants, given its parents.
* **Purpose**:
    * **Knowledge Representation**: Compactly represent the joint probability distribution over a set of variables.
    * **Reasoning under Uncertainty**: Perform probabilistic inference, such as:
        * Predicting the probability of an outcome given some evidence.
        * Diagnosing causes given observed effects.
        * Explaining away evidence (e.g., if one cause explains an effect, other potential causes become less likely).
    * **Learning**: Both the structure of the BBN and the parameters of the CPTs can be learned from data.

---
## Conclusion: Understanding the How and Why of Learning

Theoretical machine learning provides essential frameworks like PAC learning and Boosting that help us understand the capabilities and limitations of learning algorithms, guiding the design of more effective and efficient systems. Complementing this, Bayesian learning offers a robust, probabilistic approach to dealing with uncertainty, updating beliefs in the face of new evidence, and modeling complex dependencies through structures like Bayesian Belief Networks.

Together, these foundational concepts empower us to not only build intelligent systems but also to understand the principles that govern their ability to learn and reason about the world.
