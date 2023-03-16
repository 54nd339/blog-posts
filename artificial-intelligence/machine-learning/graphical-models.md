---
title: Probabilistic Graphical Models - Bayes Nets and MRFs
description: A joint distribution over n variables has exponentially many entries. A graphical model factors it into small local pieces using a graph of conditional independences - so you can store it, and answer "given this evidence, what is the probability of that" by passing messages along the graph.
date: 2023-03-16
draft: false
slug: /artificial-intelligence/graphical-models
tags:
  - Artificial Intelligence
  - Probabilistic Models
  - Machine Learning
---

Suppose you have 20 binary variables — symptoms, diseases, test results — and you want their full joint distribution so you can answer any query ("given a cough and a positive test, what's the probability of the flu?"). The joint table has `2²⁰ ≈ 1,000,000` entries. At 40 variables it exceeds the number of atoms you can address. You can't store it, learn it, or sum over it.

A **probabilistic graphical model** makes the joint tractable by exploiting **conditional independence**: most variables don't directly affect most others. A graph records which variables interact, and the joint factors into a product of small local functions — one per family of interacting variables. Then inference ("compute a conditional probability given evidence") becomes a matter of passing messages along the graph rather than summing over the whole table.

## Bayesian networks (directed)

A **Bayesian network** is a directed acyclic graph: a node per variable, an edge `A → B` meaning "A is a direct cause of / parent of B." Each node carries a **conditional probability table** `P(node | its parents)`. The joint factorizes as the product of those local tables:

$$P(X_1, \dots, X_n) = \prod_i P(X_i \mid \text{parents}(X_i))$$

For the classic example — `Burglary → Alarm ← Earthquake`, `Alarm → JohnCalls`, `Alarm → MaryCalls` — the joint over 5 variables needs `1 + 1 + 4 + 2 + 2 = 10` numbers instead of `2⁵ − 1 = 31`, and the saving compounds with more variables and sparse connectivity.

### d-separation

The graph tells you which independences hold, via **d-separation**. Two nodes are independent given a set of observed nodes `Z` if every path between them is "blocked" by `Z`. The three path patterns:

- **Chain** `A → C → B` — blocked if `C ∈ Z` (observing the middle cuts the influence).
- **Fork** `A ← C → B` — blocked if `C ∈ Z` (a common cause; knowing it makes the effects independent).
- **Collider** `A → C ← B` — blocked *unless* `C` **or one of its descendants** is in `Z`. This is the counter-intuitive one: `Burglary` and `Earthquake` are independent, but *given* that the `Alarm` went off, they become dependent — if you then learn it *was* a burglary, earthquake becomes less likely ("explaining away").

## Markov random fields (undirected)

When there's no natural causal direction — pixels in an image, atoms in a protein, nodes in a social graph — use an **undirected** model. Edges mean "these variables directly interact." The joint is a product of non-negative **factors** (potentials) over the graph's cliques, normalized:

$$P(X) = \frac{1}{Z} \prod_{c} \phi_c(X_c)$$

`Z` (the partition function) is the sum over all configurations that makes it a distribution — and computing `Z` is itself the hard part. The **Hammersley–Clifford theorem** says any positive distribution that respects the graph's Markov properties factorizes this way.

**Factor graphs** make the factorization explicit: a bipartite graph of variable nodes and factor nodes, which is the cleanest structure for the inference algorithms below and covers both directed and undirected models.

## Exact inference

The query is usually: given evidence `E = e`, compute `P(query | E = e)`. Naively this means summing the joint over all the non-query, non-evidence variables — exponential again.

- **Variable elimination** — sum out variables one at a time, pushing each sum as far right in the factor product as it goes, so it only touches the factors that mention that variable. The cost depends on the **treewidth** of the graph (roughly, how far it is from being a tree) — polynomial for trees and near-trees, exponential for densely connected graphs.
- **Belief propagation (sum-product)** — on a tree, each node sends a "message" to each neighbour summarizing everything on its side of the graph; after one sweep in each direction, every node knows its marginal. Exact on trees.
- **Junction tree** — for a general graph, cluster variables into a tree of cliques and run belief propagation on that. Exact, but the clique sizes (and cost) blow up with treewidth.

## Approximate inference

When exact is intractable:

- **Loopy belief propagation** — run the message-passing rules on a graph *with* cycles anyway. No guarantee of convergence or correctness, but often works well (it's essentially how LDPC error-correcting codes are decoded).
- **Variational inference** — approximate the true posterior with a simpler distribution `q` (e.g. one that factorizes — **mean field**) and optimize `q`'s parameters to minimize the KL divergence to the true posterior. Turns inference into optimization; fast, biased. The engine behind modern variational autoencoders.
- **Sampling (MCMC)** — draw samples from the posterior and estimate quantities from them. **Gibbs sampling** resamples one variable at a time from its conditional given the others (easy when those conditionals are simple, as in an MRF). Unbiased in the limit, slow to mix.

## Learning

- **Parameters, complete data** — just count: the maximum-likelihood CPT entry `P(X | parents)` is the observed frequency.
- **Parameters, latent variables** — **Expectation–Maximization**: alternate inferring the hidden variables' distribution given current parameters (E-step) and re-estimating parameters given those soft assignments (M-step). This is how you fit a Gaussian mixture or an HMM.
- **Structure** — search over possible graphs, scoring each by a penalized likelihood (BIC) or by testing conditional independences in the data. Hard; the search space is superexponential.

## Familiar special cases

Many models you already know are graphical models with a particular shape:

- **Naive Bayes** — a class node pointing to independent feature nodes. One fork, all leaves.
- **Hidden Markov Model** — a chain of hidden states, each emitting an observation. See [HMMs](/citadel/artificial-intelligence/hmm).
- **Kalman filter** — an HMM with continuous Gaussian states and linear dynamics.
- **Conditional Random Field** — an undirected model of labels conditioned on inputs; standard for sequence labelling before transformers.
- **Latent Dirichlet Allocation** — the topic model, a Bayes net over documents, topics, and words.

## The one idea to keep

A joint distribution over many variables is too big to store or sum over, but a graphical model factors it into small local pieces — one conditional probability table per node in a Bayesian network, one clique potential per edge group in a Markov random field — using the conditional independences the graph encodes. Inference then means passing messages along that graph (variable elimination, belief propagation, junction tree), exactly when the graph is tree-like and approximately (loopy BP, variational, MCMC) when it isn't. Naive Bayes, HMMs, Kalman filters, and CRFs are all this same idea with a specific graph.
