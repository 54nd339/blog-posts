---
title: Paths of Fixed Length - Matrix Exponentiation on the Adjacency Matrix
description: Counting walks of exactly k edges as the k-th power of the adjacency matrix, and getting shortest walks of exactly k edges by running the same exponentiation over the (min, +) semiring.
date: 2024-09-19
draft: false
slug: /cp/fixed-length-paths
tags:
  - Competitive Programming
  - Graphs
  - Shortest Paths
---

The number of walks of exactly $k$ edges from $u$ to $v$ is the $(u, v)$ entry of $A^k$, where $A$ is the adjacency matrix. [Binary exponentiation](/citadel/cp/binary-exponentiation) computes $A^k$ in $O(n^3 \log k)$ — so "how many length-$10^{18}$ walks" is tractable. Swap ordinary $(+, \times)$ for $(\min, +)$ and the same power gives the **shortest** walk of exactly $k$ edges.

## The problem

Given a graph on $n$ vertices and an integer $k$ (possibly up to $10^{18}$):

- **Count** walks of exactly $k$ edges between every pair (or one pair).
- **Shortest** walk using exactly $k$ edges.
- **Shortest** walk using **at most** $k$ edges.
- Same, modulo a prime for the counts.

A *walk* may repeat vertices and edges; a *path* may not — counting simple paths of length $k$ is #P-hard, this is walks.

## Counting: A^k over (+, ×)

$A_{ij} = $ number of edges $i \to j$ (0 or 1 for a simple graph, or a multiplicity). Then

$$(A^k)_{ij} = \sum_{\text{sequences } i = w_0, w_1, \dots, w_k = j} \prod_{t} A_{w_{t-1} w_t} = \#\{\text{walks } i \to j \text{ of length } k\}.$$

Because matrix multiplication is associative, $A^k$ is [binary exponentiation](/citadel/cp/binary-exponentiation) with matrices, $O(n^3 \log k)$.

```python
MOD = 10 ** 9 + 7

def mat_mul(X, Y):
    n, m, p = len(X), len(Y), len(Y[0])
    Z = [[0] * p for _ in range(n)]
    for i in range(n):
        Xi = X[i]
        for t in range(m):
            if Xi[t]:
                a = Xi[t]
                Yt = Y[t]
                Zi = Z[i]
                for j in range(p):
                    Zi[j] = (Zi[j] + a * Yt[j]) % MOD
    return Z

def mat_pow(A, k):
    n = len(A)
    R = [[int(i == j) for j in range(n)] for i in range(n)]   # identity
    while k:
        if k & 1:
            R = mat_mul(R, A)
        A = mat_mul(A, A)
        k >>= 1
    return R

def count_walks(adj_matrix, k):
    return mat_pow([row[:] for row in adj_matrix], k)
```

## Shortest walk of exactly k edges: the (min, +) semiring

Replace: $+ \to \min$, $\times \to +$, the additive identity $0 \to \infty$, the multiplicative identity $1 \to 0$. Then "matrix multiply" becomes

$$(X \otimes Y)_{ij} = \min_t \big(X_{it} + Y_{tj}\big),$$

and $A^{\otimes k}_{ij}$ is the minimum total weight of a walk from $i$ to $j$ with **exactly** $k$ edges ($A_{ij}$ = edge weight, or $\infty$ if no edge). This is **min-plus** (tropical) matrix exponentiation — same $O(n^3 \log k)$.

```python
INF = float("inf")

def minplus_mul(X, Y):
    n, m, p = len(X), len(Y), len(Y[0])
    Z = [[INF] * p for _ in range(n)]
    for i in range(n):
        for t in range(m):
            if X[i][t] == INF:
                continue
            xit = X[i][t]
            for j in range(p):
                if Y[t][j] != INF and xit + Y[t][j] < Z[i][j]:
                    Z[i][j] = xit + Y[t][j]
    return Z

def shortest_exactly_k(weight, k):
    n = len(weight)
    # identity for min-plus: 0 on the diagonal, INF elsewhere
    R = [[0 if i == j else INF for j in range(n)] for i in range(n)]
    A = [row[:] for row in weight]
    while k:
        if k & 1:
            R = minplus_mul(R, A)
        A = minplus_mul(A, A)
        k >>= 1
    return R
```

**At most $k$ edges:** add a self-loop of weight $0$ at every vertex (so a shorter walk can "wait"), then run `shortest_exactly_k` for $k$. Or augment the matrix with an extra absorbing state.

## Complexity

- **Time:** $O(n^3 \log k)$ — $\log k$ matrix multiplies of $O(n^3)$ each. Feasible for $n$ up to $\sim 100$–$150$ and any $k$.
- **Space:** $O(n^2)$.
- Bitset tricks push the mod-2 (parity of walk count) or boolean-reachability version to $O(n^3 / 64 \cdot \log k)$.

## Common pitfalls

- **"Walk" vs "path".** These count/optimise walks (repeats allowed). Length-$k$ *simple paths* are a different, hard problem.
- **Semiring identity.** For $(\min, +)$ the identity matrix has $0$ on the diagonal and $\infty$ off it — not the $0/1$ identity. Getting this wrong makes $A^{\otimes 0}$ wrong and the whole power off.
- **"Exactly $k$" vs "at most $k$".** They differ; the self-loop trick converts one to the other.
- **Overflow / modulus.** Walk counts explode; take everything mod the required prime *inside* `mat_mul`.
- **Directed vs undirected.** For undirected, put edges in both directions of $A$.

## The keystone

Walks of exactly $k$ edges are counted by $A^k$ (ordinary matrix product) and minimised in weight by $A^{\otimes k}$ over the $(\min, +)$ semiring — both computed by binary exponentiation in $O(n^3 \log k)$. This is the [Fibonacci matrix trick](/citadel/cp/fibonacci-numbers) generalised from a $2 \times 2$ recurrence to an $n \times n$ graph.
