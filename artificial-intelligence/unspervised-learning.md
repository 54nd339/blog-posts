---
title: Unveiling Hidden Patterns - A Journey into Unsupervised Learning - Clustering and Dimensionality Reduction
description: Explore the realm of Unsupervised Learning, focusing on how Clustering algorithms like K-Means and Kernel K-Means group data, and how Dimensionality Reduction techniques like PCA and Kernel PCA simplify complexity.
date: 2023-07-02
draft: false
slug: /pensieve/ai-ml/unsupervised-learning
tags:
  - AI/ML
  - CS Basics
---

Hey data explorers and AI enthusiasts! So far, we've delved into how machines can learn when provided with labeled data (supervised learning). But what if the data comes without a guide, without explicit labels or targets? This is where **Unsupervised Learning** steps in, a fascinating branch of Machine Learning dedicated to finding hidden patterns, structures, and insights within unlabeled datasets.

Today, we're diving into two major tasks within unsupervised learning:
1.  **Clustering**: Grouping similar data points together. We'll focus on the popular K-Means algorithm and its non-linear counterpart, Kernel K-Means.
2.  **Dimensionality Reduction**: Simplifying complex datasets by reducing the number of variables while preserving important information. We'll explore Principal Component Analysis (PCA) and its non-linear extension, Kernel PCA.

Let's uncover how AI makes sense of data without explicit instructions!

---
## Unsupervised Learning: Discovering the Unknown

In **unsupervised learning**, the machine is given a set of feature vectors without any corresponding output labels. The primary goal is to unravel the underlying similarities and inherent structure within the data itself. Unlike supervised learning, there isn't a "right" answer or a predefined target to predict. Instead, unsupervised learning is more about exploration, understanding the data's intrinsic organization, and refining this understanding through various techniques.

Key tasks in unsupervised learning include:
* **Cluster Analysis**: Grouping similar data instances together.
* **Association Rule Mining**: Discovering interesting relationships or associations among variables in large datasets (e.g., "customers who buy X also tend to buy Y").
* **Dimensionality Reduction**: Reducing the number of features while retaining essential information.

Today, we'll focus on clustering and dimensionality reduction.

---
## Clustering: Finding Natural Groupings in Data
**Cluster analysis** is the task of grouping a set of objects (represented by data points or feature vectors) in such a way that objects in the same group (called a **cluster**) are more similar to each other than to those in other clusters.

**Why Cluster?**
* **Data Exploration**: To understand the natural structure and groupings within a dataset. Breaking data into smaller, more manageable clusters can reveal patterns that are otherwise obscured.
* **Outlier Detection**: To identify instances that don't fit neatly into any cluster, potentially indicating anomalies or errors in the data.
* **Applications**: Customer segmentation for targeted marketing, organizing text documents by topic, image compression by grouping similar colors, and identifying patterns in biological data.

**Defining Similarity and Desired Cluster Properties**:
* The core of clustering lies in defining "similarity" or "dissimilarity" between data points, often using **distance metrics** (like Euclidean, Manhattan, etc., which we discussed in the context of K-NN).
* Good data partitions usually exhibit:
    * **Homogeneity within clusters**: Data points within a cluster should be highly similar to each other. This is often measured by **compactness** (e.g., low variance within a cluster).
    * **Heterogeneity between clusters**: Data points in different clusters should be as dissimilar as possible. This is often measured by **separation** (e.g., large Euclidean distance between cluster centroids).

### K-Means Clustering: Simplicity and Power

The **K-Means algorithm** is one of the most popular and widely used partitional clustering algorithms. It aims to partition $N$ data points into $K$ predefined, non-overlapping clusters, where each data point belongs to the cluster with the nearest mean (cluster centroid).

**How K-Means Works (The Algorithm)**:
1.  **Initialization**: Randomly select $K$ data points from the dataset to serve as the initial cluster **centroids** (means).
2.  **Assignment Step**: Assign each data point in the dataset to the cluster whose centroid is the closest (most similar), typically based on Euclidean distance.
3.  **Update Step**: Recalculate the centroid (mean) for each of the $K$ clusters based on all the data points currently assigned to that cluster. These new means become the new centroids for their respective clusters.
4.  **Iteration**: Repeat the Assignment Step (Step 2) and the Update Step (Step 3) until a convergence criterion is met. This usually happens when the cluster assignments no longer change, or the centroids stabilize, or a maximum number of iterations is reached.

**Objective Function (Criterion Function)**:
K-Means implicitly tries to minimize the **sum-of-squared-error (SSE)** within clusters, also known as intra-cluster variance. The SSE is defined as:
$E = \sum_{k=1}^{K} \sum_{\mathbf{x}^{(i)} \in \text{cluster}_k} \|\mathbf{x}^{(i)} - \mathbf{\mu}_k\|^2$
where $\mathbf{\mu}_k$ is the centroid of cluster $k$.

**Important Considerations for K-Means**:
* **Choosing K**: The number of clusters, $K$, must be specified beforehand. This can be challenging if the optimal number of clusters is unknown. Heuristic methods (like the elbow method or silhouette analysis) or trying different values of K and evaluating the results are common approaches.
* **Initialization Sensitivity**: The final clustering result can depend on the initial random selection of centroids. It's common practice to run the algorithm multiple times with different random initializations and choose the clustering with the lowest SSE.
* **Cluster Shape**: K-Means tends to find spherical or convex-shaped clusters and may struggle with clusters of arbitrary shapes or varying densities.
* **Outliers**: K-Means can be sensitive to outliers, as they can significantly affect the position of centroids.
* **Data Scaling**: Since K-Means uses distance measures (like Euclidean), it's crucial to standardize or normalize features to the same scale if they have different units or ranges.

**Performance Evaluation and Stopping Criteria**:
* **Stopping Criteria**: The algorithm typically stops when cluster assignments no longer change between iterations, centroids stabilize, the SSE falls below a threshold, or a maximum number of iterations is reached.
* **Cluster Interpretability**: After clustering, it's important to understand the characteristics of each cluster by examining summary statistics or visualizing the data points within each cluster to assign meaningful labels if possible.

### Kernel K-Means: Clustering in Higher Dimensions

Standard K-Means works well when clusters are roughly spherical and linearly separable. However, for more complex data structures (e.g., concentric circles, non-convex shapes), K-Means may fail to identify meaningful clusters.

**Kernel K-Means** extends the K-Means algorithm to handle such non-linearly separable data. It leverages the **kernel trick** (popularized by Support Vector Machines):
1.  **Implicit Mapping**: Data points are implicitly mapped from the original input space to a higher-dimensional feature space where, ideally, the clusters become linearly separable (or at least more amenable to K-Means). This mapping is done via a **kernel function** $k(\mathbf{x}_i, \mathbf{x}_j) = \phi(\mathbf{x}_i) \cdot \phi(\mathbf{x}_j)$, which computes the dot product in the high-dimensional space without explicitly forming the coordinates $\phi(\mathbf{x})$.
2.  **Clustering in Feature Space**: The K-Means algorithm (assignment and update steps) is then performed in this high-dimensional feature space. Distances and centroid calculations are all done using the kernel function.
    * The assignment of a point $\mathbf{x}$ to a cluster $C_k$ with centroid $\mathbf{m}_k$ (in feature space) depends on minimizing $\|\phi(\mathbf{x}) - \mathbf{m}_k\|^2$. This can be rewritten in terms of kernel functions.
    * Centroids are also effectively recomputed in the feature space.

By using appropriate kernel functions (e.g., Radial Basis Function (RBF) kernel, polynomial kernel), Kernel K-Means can identify clusters with complex, non-spherical shapes that standard K-Means would miss.

---
## Dimensionality Reduction: Simplifying Complexity
Many real-world datasets are high-dimensional, meaning they have a large number of features or variables. While rich in information, high dimensionality can lead to problems known as the "curse of dimensionality," increased computational cost, and difficulties in visualization and model interpretability. **Dimensionality reduction** techniques aim to reduce the number of features while retaining as much of the original data's important information (variance or structure) as possible.

### Principal Component Analysis (PCA): Finding the Most Important Directions

**Principal Component Analysis (PCA)** is a widely used unsupervised linear transformation technique for dimensionality reduction. It identifies the directions (principal components) in the data that capture the maximum variance.

**The Goal of PCA**:
* To transform the data from its original $n$-dimensional space to a new $k$-dimensional space (where $k < n$) with minimal loss of information (variance).
* To remove redundancy (correlation) among features. The transformed features (principal components) are uncorrelated.
* To reduce noise by focusing on components with higher signal-to-noise ratio (higher variance).

**How PCA Works (Key Steps)**:

1.  **Standardize the Data**: Transform the data so that each feature has zero mean and unit variance. This is crucial because PCA is sensitive to the scales of the original variables.
2.  **Compute the Covariance Matrix**: Calculate the covariance matrix $\Sigma_{\tilde{X}}$ of the standardized data $\tilde{X}$. The covariance matrix describes the relationships and variances between pairs of features.
3.  **Calculate Eigenvectors and Eigenvalues**: Compute the eigenvectors and eigenvalues of the covariance matrix.
    * Eigenvectors represent the principal directions of variance in the data.
    * Eigenvalues represent the magnitude of variance along those principal directions.
4.  **Sort Eigenvectors**: Sort the eigenvectors in descending order based on their corresponding eigenvalues. The eigenvector with the largest eigenvalue is the first principal component, capturing the most variance, and so on.
5.  **Select Principal Components**: Choose the top $k$ eigenvectors (those corresponding to the $k$ largest eigenvalues) to form the new feature subspace. The value of $k$ can be chosen based on the desired amount of variance to retain (e.g., 90-99% of the total variance).
    The proportion of variance explained by the top $k$ principal components is $\frac{\sum_{i=1}^k \lambda_i}{\sum_{j=1}^n \lambda_j}$.
6.  **Construct the Projection Matrix (W)**: Form a matrix $W$ whose columns are the selected $k$ eigenvectors.
7.  **Transform the Data**: Project the original standardized data $X_{std}$ onto the new $k$-dimensional subspace to obtain the new feature set $Z$: $Z = X_{std} W$.

**Advantages of PCA**:
* Reduces dimensionality, making data easier to visualize and process.
* Removes correlations between features (principal components are orthogonal).
* Can improve the performance of subsequent machine learning algorithms by reducing noise and redundancy.
* Can help in overcoming overfitting.

**Disadvantages of PCA**:
* The principal components are linear combinations of the original features, making them less interpretable.
* Information loss is inevitable, though minimized by selecting components with the highest variance. The amount of loss depends on the chosen $k$.
* Requires features to be scaled before applying PCA due to its variance-maximizing nature.
* PCA is a linear method and may not capture non-linear relationships in the data.

### Kernel PCA (KPCA): Non-Linear Dimensionality Reduction

When data has complex non-linear structures, linear PCA might not be effective in finding a meaningful low-dimensional representation. **Kernel Principal Component Analysis (Kernel PCA or KPCA)** extends PCA to handle non-linearity by using the kernel trick.

**The Core Idea**:
1.  **Implicit High-Dimensional Mapping**: Similar to Kernel K-Means and SVMs, Kernel PCA implicitly maps the original data into a very high-dimensional (possibly infinite-dimensional) feature space using a non-linear kernel function $\phi(\mathbf{x})$. In this space, complex non-linear relationships in the original data might become linear.
2.  **Linear PCA in Feature Space**: Standard PCA is then performed in this high-dimensional feature space. The kernel trick allows us to do this without explicitly computing the coordinates $\phi(\mathbf{x})$. Instead, all operations are performed using the **kernel matrix** $K$, where $K_{ij} = k(\mathbf{x}_i, \mathbf{x}_j) = \phi(\mathbf{x}_i) \cdot \phi(\mathbf{x}_j)$.
3.  **Finding Non-Linear Components**: Solving an eigenvalue problem on the (centered) kernel matrix yields the principal components in the feature space. These correspond to non-linear principal components in the original input space.

Kernel PCA can uncover non-linear manifolds and structures in the data that linear PCA would miss, providing a more powerful tool for dimensionality reduction when non-linearity is present. Common kernels include the RBF kernel, polynomial kernel, and sigmoid kernel.

---
## Conclusion: Discovering Structure and Simplicity in Data

Unsupervised learning is a powerful paradigm for exploring and understanding data when explicit labels are absent. Clustering techniques like K-Means (and its non-linear extension Kernel K-Means) help us discover natural groupings and segments within our data, revealing inherent similarities. Dimensionality reduction techniques like PCA (and its non-linear variant Kernel PCA) allow us to simplify complex, high-dimensional datasets by identifying the most important underlying features or directions of variance.

These methods are not just theoretical tools; they are essential for practical data analysis, feature engineering, and building more efficient and insightful machine learning models. By uncovering the hidden landscapes within our data, unsupervised learning helps us make better sense of the world around us.
