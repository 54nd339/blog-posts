---
title: Evaluating and Scaling Machine Learning - Metrics, Ensembles, and Distribution
description: What comes after training a model - the metrics for classification and regression, cross-validation and the bias-variance trade-off, ensemble methods (bagging, random forests, boosting), modelling sequential data, representation learning, and scaling with online and distributed training.
date: 2023-03-16
draft: false
slug: /artificial-intelligence/advanced-ml
tags:
  - Artificial Intelligence
  - Machine Learning
  - Distributed Systems
---

Fitting a model is the easy part. The harder questions come next: is it actually any good on data it hasn't seen, is there a better one, can several mediocre models be combined into a strong one, and how do you train when the data won't fit on one machine? This post works through evaluation, model selection, ensembles, sequence models, and the two ways to scale training.

## Evaluating a model

The only thing that matters is **generalisation** — performance on new data from the same distribution. A model that scores perfectly on its training set and poorly on fresh data is worthless.

**For classification**, start with the **confusion matrix**: counts of true positives (TP), true negatives (TN), false positives (FP), false negatives (FN). From it:

- **Accuracy** — $\frac{TP + TN}{TP + TN + FP + FN}$. Misleading on imbalanced classes.
- **Precision** — $\frac{TP}{TP + FP}$: of everything flagged positive, how much really was. High precision means few false alarms.
- **Recall** (sensitivity) — $\frac{TP}{TP + FN}$: of the real positives, how many were caught. High recall means few misses.
- **F1** — the harmonic mean $\frac{2 \cdot P \cdot R}{P + R}$, balancing the two.
- **Micro vs macro averaging** — for multi-class, macro computes the metric per class then averages (all classes equal weight); micro pools all classes' counts first (common classes dominate).
- **Mean reciprocal rank** — for ranked outputs, the average of $1/\text{rank}$ of the first correct answer.

**For regression**: **MSE** (mean squared error), **RMSE** (its square root, back in the target's units), **MAE** (mean absolute error, less outlier-sensitive), and **$R^2$** — the fraction of the target's variance the model explains.

## Model selection

To estimate generalisation you must test on data the model didn't train on:

- **Hold-out** — split into a training set and a test set. Simple, but a single split is noisy with limited data.
- **k-fold cross-validation** — split into $k$ folds; train on $k-1$, test on the held-out one, rotate so each fold is the test set once; average the $k$ scores.
- **Validation (development) set** — a third split used *during* development to tune hyperparameters (the $k$ in k-NN, a tree's depth). Tuning on the test set contaminates it; the test set is used once, at the end.

Underlying all of this is the **bias-variance trade-off**: a model too simple has high **bias** and underfits; one too complex has high **variance**, fitting noise, and overfits. Selection is the search for the balance.

## Ensembles

Combining models often beats any single one.

**Bagging** (bootstrap aggregating) reduces **variance**. Draw several bootstrap samples — same-size samples with replacement — from the training set, train a copy of the model independently on each, and combine by majority vote (classification) or averaging (regression). It helps most for high-variance, low-bias models like fully grown [decision trees](/citadel/data-structures/trees).

**Random forests** are bagging with an extra twist: at each split, only a random subset of features is considered. This de-correlates the trees — without it, a few dominant features would make every tree look alike — and averaging de-correlated trees cuts variance further. Random forests resist overfitting and tolerate missing data.

**Boosting** reduces **bias** by building models sequentially, each correcting its predecessors. **AdaBoost** raises the weight of misclassified examples each round so later learners focus on them; the final prediction is a weighted vote, better learners weighted more. The learnability theory behind it — weak learners provably combine into a strong one — is in [learning theory](/citadel/artificial-intelligence/theorotical-ml).

## Sequential data

When order carries meaning — text, speech, sensor streams, financial series — the model has to capture dependencies across time:

- **N-gram models** predict the next item from the previous $n - 1$. Covered in [natural language processing](/citadel/artificial-intelligence/nlp).
- **Hidden Markov models** describe a sequence of hidden states emitting observations. Covered in [HMMs and tagging](/citadel/artificial-intelligence/hmm).
- **Recurrent neural networks** carry a hidden state across time steps. Covered in [optimisation and generalisation](/citadel/artificial-intelligence/generalisation).
- **Maximum-entropy Markov models** and **conditional random fields** are discriminative sequence labellers — they model the label sequence given the whole observation sequence, allowing rich overlapping features. Also in [HMMs and tagging](/citadel/artificial-intelligence/hmm).

## Representation learning

Classical ML depends on **feature engineering** — a domain expert hand-crafts the input features. Deep networks do **representation learning** instead: their layers transform raw input into progressively more useful features, learned directly from data. Early layers catch low-level structure (edges in an image), later layers combine them into abstractions (object parts, objects). See [an introduction to deep learning](/citadel/artificial-intelligence/dl).

## Scaling training

**Online (incremental) learning** updates the model as each example or mini-batch arrives, rather than training on the whole dataset at once. It never needs the full data in memory, suits streaming data, and adapts as the data distribution drifts. Stochastic gradient descent is the canonical online algorithm.

**Distributed learning** spreads training across machines:

- **Data parallelism** — split the data; each machine trains a copy of the model on its shard; gradients are aggregated.
- **Model parallelism** — split the model itself across devices, for models too big for one.

Frameworks like [Apache Spark](/citadel/tech/apache) and MapReduce-style systems provide the plumbing for large-scale distributed training.

## The one idea to keep

Everything here serves generalisation. Cross-validation *estimates* it, the bias-variance trade-off *explains* when it fails, ensembles *improve* it by averaging away variance or chipping away bias, and online and distributed training let you keep improving it as data grows past one machine. A model you can't evaluate honestly isn't a model, just a fit.
