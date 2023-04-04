---
title: Dimensionality Reduction - PCA, LDA, and Non-Linear Methods
description: "Taming high-dimensional data - the curse of dimensionality, principal component analysis for unsupervised variance-preserving projection, linear discriminant analysis for supervised class separation, and the non-linear family: manifold learning, metric learning, and autoencoders."
date: 2023-04-04
draft: false
slug: /artificial-intelligence/dimesionality-reduction
tags:
  - Artificial Intelligence
  - Machine Learning
  - Deep Learning
---

More features are not more information. Past a point, adding dimensions makes a dataset *harder* to learn from — the data spreads thin, models overfit, and nothing can be visualised. **Dimensionality reduction** transforms data into fewer dimensions while keeping what matters. This post covers the two linear methods, PCA and LDA, and the non-linear family that handles curved structure.

## The curse of dimensionality

As the number of features $d$ grows, the volume of the feature space grows exponentially, so a fixed number of data points becomes increasingly sparse — statistically significant patterns get harder to find. On top of that:

- **Computational cost** — more features means more parameters, longer training, more memory.
- **Overfitting** — with many features relative to examples, a model fits noise.
- **Redundancy and noise** — irrelevant or highly correlated features drag down performance.
- **Visualisation** — humans can't see beyond three dimensions.

Reducing dimensions trades some information for simpler models, better generalisation, and often interpretability.

## Principal component analysis

**PCA** is the standard *unsupervised* linear method. It finds the orthogonal directions of maximum variance in the data — the **principal components** — and projects onto the top $k$ of them. Concretely: standardise the features, form the covariance matrix, take its eigenvectors sorted by eigenvalue, and keep enough to retain a target fraction of the total variance. The full step-by-step derivation, with the covariance formula and the choice of $k$, is in [unsupervised learning](/citadel/artificial-intelligence/unspervised-learning); [kernel PCA](/citadel/artificial-intelligence/unspervised-learning) is its non-linear extension.

PCA ignores class labels — it maximises variance, which isn't the same as maximising class separation. When you have labels and want a projection that keeps the classes apart, use LDA instead.

## Linear discriminant analysis

**LDA** is a *supervised* dimensionality reduction (and classification) method. It seeks the projection that maximises the ratio of **between-class** variance to **within-class** variance — spreading the class means apart while keeping each class tight.

Two scatter matrices capture this. Between-class:

$$S_B = \sum_{c=1}^{C} N_c\,(\mathbf{m}_c - \mathbf{m})(\mathbf{m}_c - \mathbf{m})^{\mathsf T}$$

with $N_c$ the size of class $c$, $\mathbf{m}_c$ its mean, $\mathbf{m}$ the overall mean, $C$ the number of classes. Within-class:

$$S_W = \sum_{c=1}^{C} \sum_{\mathbf{x}_i \in c} (\mathbf{x}_i - \mathbf{m}_c)(\mathbf{x}_i - \mathbf{m}_c)^{\mathsf T}$$

LDA finds the projection $\mathbf{W}$ maximising $J(\mathbf{W}) = \dfrac{|\mathbf{W}^{\mathsf T} S_B \mathbf{W}|}{|\mathbf{W}^{\mathsf T} S_W \mathbf{W}|}$, which reduces to the generalised eigenvalue problem $S_B \mathbf{w} = \lambda S_W \mathbf{w}$ — the columns of the optimal $\mathbf{W}$ are the top eigenvectors of $S_W^{-1} S_B$. LDA can produce at most $C - 1$ dimensions, since $S_B$ has rank $C - 1$.

## Non-linear methods

Linear methods flatten curved structure. When the data lies on a curved surface — a 2D sheet rolled up in 3D — you need something that can unroll it.

**Manifold learning** assumes the data lies on or near a low-dimensional non-linear **manifold** inside the high-dimensional space, and tries to recover it:

- **Isomap** — preserves *geodesic* distances (shortest paths along the manifold) rather than straight-line distances.
- **Locally linear embedding (LLE)** — assumes each point and its neighbours sit on a locally flat patch, and preserves those local linear relationships.
- **t-SNE** — optimised for *visualising* high-dimensional data in 2D or 3D by preserving local neighbourhoods, which makes cluster structure pop out.

**Metric learning** learns the *distance function* itself from data, rather than assuming Euclidean distance. The goal is a metric under which semantically similar points (same class) are close and dissimilar ones far, which then improves any distance-based method — [k-NN](/citadel/artificial-intelligence/ml), clustering, similarity search.

**Autoencoders** are neural networks for unsupervised representation learning. An **encoder** $f$ maps the input $\mathbf{x}$ to a lower-dimensional code $\mathbf{z} = f(\mathbf{x})$; a **decoder** $g$ reconstructs $\hat{\mathbf{x}} = g(\mathbf{z})$. Training minimises the **reconstruction error** $\lVert \mathbf{x} - \hat{\mathbf{x}} \rVert^2$. With non-linear activations, the encoder performs non-linear dimensionality reduction; the code $\mathbf{z}$ is the reduced representation. Sparse, denoising, and variational autoencoders add further structure. Autoencoders are trained by [backpropagation](/citadel/artificial-intelligence/dl) like any other network.

## Where CNNs fit

[Convolutional networks](/citadel/artificial-intelligence/deep-networks) also shrink dimensionality as they go — pooling layers halve spatial resolution repeatedly — while building up a feature hierarchy. But a CNN is a full supervised architecture, not a dimensionality-reduction technique; its layers, its landmark designs (AlexNet, VGG, Inception, ResNet), and its training considerations are covered with the rest of deep learning.

## The one idea to keep

Every method here answers "which $k$ directions matter?" with a different criterion. PCA says: the ones with the most variance. LDA says: the ones that separate the classes. Manifold learning says: the ones that preserve local neighbourhood structure on a curved surface. Autoencoders say: whatever a bottleneck can reconstruct the input from. Pick the criterion that matches what you'll do with the reduced data.
