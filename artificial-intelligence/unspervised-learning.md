---
title: Unsupervised Learning - Clustering and Principal Component Analysis
description: Finding structure in unlabelled data - what clustering asks for, the k-means algorithm and the sum-of-squared-error it minimises, kernel k-means for non-spherical clusters, and principal component analysis step by step, with its kernel extension for non-linear structure.
date: 2023-03-14
draft: false
slug: /artificial-intelligence/unspervised-learning
tags:
  - Artificial Intelligence
  - Machine Learning
  - Probabilistic Models
---

[Supervised learning](/citadel/artificial-intelligence/ml) comes with an answer key: every training input has a correct output. **Unsupervised learning** has none. It's handed a pile of feature vectors and asked to find the structure in them — which points group together, and what lower-dimensional shape the data really has. This post covers the two staple tasks: clustering, and dimensionality reduction by principal component analysis.

## The unsupervised setting

The algorithm gets feature vectors with no labels or targets. The goal isn't prediction — it's discovering the data's intrinsic organisation. Three tasks:

- **Cluster analysis** — group similar instances.
- **Association rule mining** — find relationships among variables ("buyers of X also buy Y").
- **Dimensionality reduction** — cut the number of features while keeping the important information.

## Clustering

**Clustering** partitions objects so that those in the same group are more similar to each other than to those in other groups. A good partition has:

- **Homogeneity within clusters** — points in a cluster are alike; measured by **compactness** (low within-cluster variance).
- **Separation between clusters** — different clusters are far apart; measured by distance between centroids.

Similarity is a [distance metric](/citadel/artificial-intelligence/ml) — Euclidean, Manhattan, and so on. Clustering is used for data exploration, outlier detection (points that fit no cluster), customer segmentation, document organisation, and image colour quantisation.

## K-means

**K-means** partitions $N$ points into $K$ non-overlapping clusters, each point belonging to the cluster with the nearest centroid.

1. **Initialise** — pick $K$ points as initial centroids.
2. **Assign** — put each point in the cluster of its nearest centroid (usually by Euclidean distance).
3. **Update** — recompute each centroid as the mean of its assigned points.
4. **Repeat** steps 2–3 until assignments stop changing (or a max iteration count).

It implicitly minimises the **sum of squared error** — the total within-cluster variance:

$$E = \sum_{k=1}^{K} \sum_{\mathbf{x}^{(i)} \in C_k} \lVert \mathbf{x}^{(i)} - \boldsymbol{\mu}_k \rVert^2$$

with $\boldsymbol{\mu}_k$ the centroid of cluster $k$. Practical points:

- **Choosing $K$** — it must be set in advance. The elbow method (plot SSE against $K$) or silhouette analysis help.
- **Initialisation sensitivity** — the result depends on the starting centroids; run several times with different random starts and keep the lowest-SSE result.
- **Cluster shape** — k-means finds roughly spherical, convex clusters and struggles with elongated or nested shapes.
- **Outliers** drag centroids around.
- **Scale** — features must be normalised, since the objective is a Euclidean distance.

## Kernel k-means

Standard k-means fails when clusters aren't linearly separable — two concentric rings, for instance. **Kernel k-means** applies the [kernel trick](/citadel/artificial-intelligence/ml): implicitly map points into a higher-dimensional feature space where the clusters *do* separate, via a kernel $k(\mathbf{x}_i, \mathbf{x}_j) = \phi(\mathbf{x}_i) \cdot \phi(\mathbf{x}_j)$, and run the assignment and update steps there. Every distance and centroid computation is rewritten in terms of the kernel, so $\phi$ is never formed explicitly. With an RBF or polynomial kernel, kernel k-means recovers clusters of complex shape.

## Principal component analysis

**PCA** is an unsupervised linear transformation that finds the orthogonal directions — **principal components** — capturing the most variance in the data, and projects onto the top few.

Given a dataset $\mathbf{X}$ of $N$ samples and $d$ features:

1. **Standardise** — rescale each feature to zero mean and unit variance, giving $\mathbf{X}_{std}$. PCA is scale-sensitive, so this matters.
2. **Covariance matrix** — $\Sigma = \frac{1}{N - 1}\mathbf{X}_{std}^{\mathsf T}\mathbf{X}_{std}$, a $d \times d$ symmetric matrix of feature variances and covariances.
3. **Eigendecomposition** — solve $\Sigma\mathbf{v} = \lambda\mathbf{v}$. The **eigenvectors** are the principal directions; the **eigenvalues** are the variance captured along each. (See [linear algebra](/citadel/maths/linear-algebra).)
4. **Sort** the eigenvectors by descending eigenvalue: $\lambda_1 \ge \lambda_2 \ge \dots \ge \lambda_d$. The first is PC1, capturing the most variance.
5. **Select** the top $k$ eigenvectors. Pick $k$ to retain a target fraction of variance — $\frac{\sum_{i=1}^{k}\lambda_i}{\sum_{j=1}^{d}\lambda_j}$, say 95% — or by the elbow of a scree plot.
6. **Projection matrix** — stack the $k$ chosen eigenvectors as columns of a $d \times k$ matrix $\mathbf{W}$.
7. **Transform** — $\mathbf{Z} = \mathbf{X}_{std}\,\mathbf{W}$, the data in $k$ dimensions.

PCA removes correlation (the components are orthogonal), reduces noise (low-variance directions are dropped), and helps with overfitting. The costs: the components are linear combinations of the original features, so they're less interpretable; some information is always lost; and it can't capture non-linear structure.

## Kernel PCA

**Kernel PCA** extends PCA to non-linear structure using the same trick as kernel k-means. Map the data into a high-dimensional feature space with a non-linear kernel, then do ordinary PCA there — but entirely through the **kernel matrix** $K_{ij} = k(\mathbf{x}_i, \mathbf{x}_j)$, never forming $\phi(\mathbf{x})$. An eigenvalue problem on the centred kernel matrix yields components that are non-linear in the original space, so kernel PCA can unfold curved manifolds that linear PCA flattens. Other non-linear dimensionality-reduction methods — manifold learning, autoencoders — are in [dimensionality reduction](/citadel/artificial-intelligence/dimesionality-reduction).

## The one idea to keep

Unsupervised learning trades the answer key for an objective you define. K-means makes "good clustering" concrete as minimum within-cluster squared error and descends it by alternating assignment and update; PCA makes "important information" concrete as variance and keeps the directions with the most of it. Both go non-linear the same way — swap the dot product for a kernel — without changing the underlying algorithm.
