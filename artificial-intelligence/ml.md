---
title: Machine Learning - Supervised Learning and Its Core Algorithms
description: Learning rules from examples instead of writing them - the supervised setting, distance-based methods and k-NN, decision trees and their splitting criteria, Naive Bayes, linear and logistic regression, generalised linear models, and the max-margin idea behind support vector machines and kernels.
date: 2023-03-12
draft: false
slug: /artificial-intelligence/ml
tags:
  - Artificial Intelligence
  - Machine Learning
  - Probabilistic Models
---

In ordinary programming you write the rules and the computer applies them: `program + data → output`. Machine learning flips the middle: you supply example inputs *and* their correct outputs, and the computer produces the rule — `data + outputs → model` — which then predicts on inputs it hasn't seen.

Tom Mitchell's definition pins it down: a program learns from **experience E** at some **task T**, measured by **performance P**, if its performance at T (as measured by P) improves with E. A spam filter's task is classifying mail; its experience is your "spam"/"not spam" marks; its performance is filtering accuracy — and it gets better as you mark more.

This post covers the **supervised** setting — learning from labelled examples — and the algorithms that show up first in every course.

## Where machine learning sits

ML is a subset of [AI](/citadel/artificial-intelligence/ai): one way to build the learning element of an agent. It leans heavily on statistics — regression, variance, probability. **Deep learning** is the sub-branch that uses [neural networks with many layers](/citadel/artificial-intelligence/dl). **Data mining** overlaps but aims at discovering unknown patterns in data, where ML aims at improving performance on a defined task.

## Supervised learning

The algorithm gets a dataset of **examples**, each an input (a feature vector) paired with a **correct label** or target. It learns the mapping from features to target. Two task types:

- **Regression** — the target is a continuous number. Predict a house price from size and location; forecast next month's sales; estimate tomorrow's temperature.
- **Classification** — the target is one of a fixed set of classes. Spam or not; fraudulent or legitimate; tumour benign or malignant; article about politics, sports, or business.

## Distance-based methods

Many algorithms rest on a notion of **similarity** between feature vectors, made concrete by a distance metric:

- **Euclidean** — straight-line distance, $\sqrt{\sum_i (a_i - b_i)^2}$. Sensitive to feature scale and outliers; usually needs normalisation.
- **Manhattan** — $\sum_i |a_i - b_i|$. More robust to outliers.
- **Minkowski** — $\left(\sum_i |a_i - b_i|^p\right)^{1/p}$, generalising both ($p = 2$ Euclidean, $p = 1$ Manhattan).
- **Mahalanobis** — accounts for correlations between features via the covariance matrix.
- **Hamming** — number of differing positions between two equal-length strings or bit vectors.

## k-nearest neighbours

**k-NN** is the direct use of distance. It's a **lazy learner** — no model is built at training time; it just stores the data. To predict for a new point: compute its distance to every training point, take the $k$ nearest, and for **classification** assign the majority class among them, for **regression** average their targets.

The knobs: **$k$** trades off noise sensitivity (small $k$) against blurred boundaries (large $k$), and is usually picked by cross-validation. Features **must be scaled**, since the metric is scale-sensitive. Prediction is $O(n)$ per query — it touches every training point — and accuracy degrades in high dimensions (the curse of dimensionality) and with irrelevant features.

## Decision trees

A **decision tree** learns a hierarchy of `if-then-else` rules. The **root** holds all the data; each **internal node** tests one feature; each **branch** is an outcome of that test; each **leaf** is a predicted class or value.

It's built **top-down and greedily**: at each node, pick the feature that best splits the data into pure subsets, and recurse until a stopping rule fires (node is pure, no features left, or a depth limit). "Best" is measured by one of:

- **Information gain** (ID3) — reduction in entropy after the split.
- **Gain ratio** (C4.5) — information gain penalised for features with many values.
- **Gini index** (CART) — impurity of the resulting partition.

Trees **overfit** readily, growing complex branches that fit noise. Two remedies: **pre-pruning** (stop growing early on a threshold) and **post-pruning** (grow the full tree, then cut back branches that don't help on a validation set).

## Naive Bayes

A probabilistic classifier built on **Bayes' theorem** (see [probability and statistics](/citadel/maths/probablity-statistics)):

$$P(C \mid X) = \frac{P(X \mid C)\,P(C)}{P(X)}$$

with $P(C \mid X)$ the posterior, $P(X \mid C)$ the likelihood, $P(C)$ the prior, $P(X)$ the evidence.

The **naive** assumption is that features are conditionally independent given the class, so the likelihood factorises: $P(X \mid C) = \prod_i P(x_i \mid C)$. Classification then picks the class with the largest posterior, dropping the constant $P(X)$:

$$y_{NB} = \arg\max_{C_k}\; P(C_k) \prod_{i=1}^{n} P(x_i \mid C_k)$$

Priors and likelihoods are estimated by counting in the training data. Because an unseen feature-class pair would give a zero and wipe out the product, **Laplace (add-one) smoothing** is essential. It's crude but works remarkably well for text classification; the fuller Bayesian view is in [theoretical machine learning](/citadel/artificial-intelligence/theorotical-ml).

## Linear regression

Fit a line (or hyperplane) to continuous data:

$$\hat{y} = w_0 + w_1 x_1 + \dots + w_n x_n$$

The weights minimise the **residual sum of squares**. The closed-form **least-squares** solution is $\mathbf{w}^* = (X^{\mathsf T} X)^{-1} X^{\mathsf T} \mathbf{y}$; **gradient descent** is the iterative alternative, preferred when $X^{\mathsf T} X$ is large or ill-conditioned.

## Logistic regression

For binary classification, model the probability of the positive class by squashing the linear output through the **sigmoid**:

$$P(Y = 1 \mid \mathbf{x}) = \frac{1}{1 + e^{-(\mathbf{w}^{\mathsf T}\mathbf{x} + b)}}$$

which stays in $(0, 1)$. Parameters are fit by **maximum likelihood**, equivalently by minimising **log loss** (cross-entropy), via gradient descent — least squares doesn't suit the S-curve. Despite the non-linear sigmoid, the **decision boundary** (where $P = 0.5$, i.e. $\mathbf{w}^{\mathsf T}\mathbf{x} + b = 0$) is linear in the features.

## Generalised linear models

**GLMs** extend linear regression to targets that aren't normally distributed, via three parts:

1. **Random component** — the target's distribution (Normal, Binomial, Poisson, Gamma), from the exponential family.
2. **Systematic component** — a linear predictor $\eta = \mathbf{X}\boldsymbol{\beta}$.
3. **Link function** $g$ — relates the mean $\mu = E(Y)$ to the predictor: $g(\mu) = \eta$.

Linear regression is the **identity link** with a Normal distribution; logistic regression is the **logit link** $g(\mu) = \ln\frac{\mu}{1-\mu}$ with a Binomial; Poisson regression for counts uses the **log link**.

## Support vector machines

An SVM classifier finds the separating **hyperplane** with the largest **margin** — the widest gap to the nearest points of either class. Those nearest points are the **support vectors**; they alone determine the hyperplane. The margin is $\frac{2}{\lVert \mathbf{w} \rVert}$, so maximising it means minimising $\frac{1}{2}\lVert \mathbf{w} \rVert^2$ subject to $y_i(\mathbf{w} \cdot \mathbf{x}_i - b) \ge 1$ for every training point — a convex quadratic program. A wide margin tends to generalise better.

## Kernel methods

Real data is often not linearly separable. The fix: map the features into a higher-dimensional space $\phi(\mathbf{x})$ where a separating hyperplane exists. The **kernel trick** avoids ever computing $\phi(\mathbf{x})$ — for any algorithm that only uses dot products (SVMs among them), replace the dot product with a **kernel function**

$$K(\mathbf{x}_i, \mathbf{x}_j) = \phi(\mathbf{x}_i) \cdot \phi(\mathbf{x}_j)$$

computed directly from the original inputs. Common kernels: **polynomial** $(\mathbf{x}_i \cdot \mathbf{x}_j + c)^d$, **RBF** $\exp(-\gamma \lVert \mathbf{x}_i - \mathbf{x}_j \rVert^2)$, and sigmoid. This buys non-linear decision boundaries at the cost of a linear model.

## The one idea to keep

Supervised learning is curve-fitting with a purpose: find a function from features to target that does well on data you haven't seen. The algorithms differ in what family of functions they search — piecewise-constant regions (k-NN, trees), a probabilistic generative story (Naive Bayes), a linear boundary (logistic regression, linear SVM), or a linear boundary in a lifted space (kernels). Choosing well means matching that family to the shape of your data, and the next post — [evaluation and ensembles](/citadel/artificial-intelligence/advanced-ml) — is about how to tell whether you did.
