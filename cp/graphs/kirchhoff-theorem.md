---
title: Kirchhoff's Theorem - Counting Spanning Trees by Determinant
description: The number of spanning trees of a graph equals any cofactor of its Laplacian matrix, so one determinant counts them — with the weighted and directed (arborescence) generalisations.
date: 2024-09-20
draft: false
slug: /cp/kirchhoff-theorem
tags:
  - Competitive Programming
  - Graphs
  - Minimum Spanning Tree
---

The matrix-tree theorem (Kirchhoff, 1847) says the number of spanning trees of a graph is **any cofactor of its Laplacian matrix** — delete one row and the matching column, take the [determinant](/citadel/cp/matrix-determinant), done. A counting problem that sounds like it needs exponential enumeration collapses to one $O(n^3)$ determinant.

## The problem

Count the spanning trees of a connected undirected graph on $n$ vertices. Generalisations: **weighted** count (sum over spanning trees of the product of edge weights), and **directed** count (number of spanning arborescences rooted at a fixed vertex).

Example: $K_4$ (complete graph on 4 vertices) has $4^{4-2} = 16$ spanning trees ([Cayley's formula](/citadel/cp/prufer-code)); Kirchhoff on its Laplacian gives $16$.

## The idea

The **Laplacian** $L = D - A$, where $D$ is the diagonal degree matrix and $A$ the adjacency matrix. So $L_{ii} = \deg(i)$ and $L_{ij} = -(\text{number of edges between } i \text{ and } j)$ for $i \ne j$.

$L$ is singular (every row sums to $0$), but **every cofactor is equal**, and that common value is the spanning-tree count:

$$\#\text{spanning trees} = \det\big(L \text{ with row } r \text{ and column } r \text{ deleted}\big)$$

for any choice of $r$.

Why, intuitively: expand the determinant via the Cauchy-Binet formula on $L = M M^\top$ where $M$ is the (reduced) incidence matrix; each nonzero term corresponds to a set of $n - 1$ edges forming a spanning tree, contributing $\pm 1$, and the signs work out to $+1$ each.

## Algorithm

```python
def count_spanning_trees(n, edges, mod=None):
    L = [[0] * n for _ in range(n)]
    for u, v in edges:
        L[u][u] += 1
        L[v][v] += 1
        L[u][v] -= 1
        L[v][u] -= 1
    # delete last row and column
    M = [row[:n - 1] for row in L[:n - 1]]
    return determinant(M, mod)                 # see the matrix-determinant post

def determinant(a, mod=None):
    a = [row[:] for row in a]
    n = len(a)
    if n == 0:
        return 1
    det = 1
    for c in range(n):
        piv = next((r for r in range(c, n) if a[r][c] % (mod or 10**18) != 0), -1) if mod \
              else max(range(c, n), key=lambda r: abs(a[r][c]))
        if (mod and piv == -1) or (not mod and a[piv][c] == 0):
            return 0
        if piv != c:
            a[c], a[piv] = a[piv], a[c]
            det = -det
        if mod:
            inv = pow(a[c][c] % mod, mod - 2, mod)
            det = det * a[c][c] % mod
            for r in range(c + 1, n):
                f = a[r][c] * inv % mod
                a[r] = [(a[r][k] - f * a[c][k]) % mod for k in range(n)]
        else:
            det *= a[c][c]
            for r in range(c + 1, n):
                f = a[r][c] / a[c][c]
                a[r] = [a[r][k] - f * a[c][k] for k in range(n)]
    return round(det) if not mod else det % mod
```

For an exact large integer answer with no modulus, use the [Bareiss](/citadel/cp/matrix-determinant) fraction-free determinant instead of the floating-point one.

## Generalisations

- **Weighted spanning trees:** put edge weight $w$ (not $1$) into the Laplacian: $L_{ii} \mathrel{+}= w$, $L_{ij} \mathrel{-}= w$. The cofactor is $\sum_{\text{spanning trees } T} \prod_{e \in T} w_e$. Set $w_e$ = a formal variable to get generating functions; set $w_e = 1$ to just count.
- **Multigraphs:** parallel edges add their contributions; the formula already handles it.
- **Directed graphs — spanning arborescences (matrix-tree for digraphs):** use $L_{ii} = \text{in-degree}(i)$ (or out-degree, depending on root convention), $L_{ij} = -(\text{arcs } j \to i)$. Delete the row/column of the **root**; the cofactor counts arborescences oriented toward (or away from) that root.
- **BEST theorem:** the number of **Eulerian circuits** of a connected digraph = $\text{ec}(G) \cdot \prod_v (\deg^+(v) - 1)!$, where $\text{ec}(G)$ is any cofactor of the Laplacian — Kirchhoff feeding [Eulerian path](/citadel/cp/eulerian-path) counting.

## Complexity

- **Time:** $O(n^3)$ for the determinant.
- **Space:** $O(n^2)$.
- Modulo a prime: exact. Without a modulus: use exact-integer Bareiss ($O(n^3)$ integer ops on numbers up to $\sim n$ digits) — floating point is fine only for small $n$.

## Common pitfalls

- **Which row/column to delete.** *Any* single index works and all give the same value — for the undirected case. For the directed (arborescence) case you **must** delete the root's row and column.
- **Determinant precision.** For $n \gtrsim 15$ the floating-point determinant of an integer matrix is unreliable; use modular or Bareiss.
- **Disconnected graph.** The cofactor is $0$ — as it should be, there are no spanning trees.
- **Self-loops.** They do not affect spanning trees; drop them before building $L$ (a self-loop would wrongly inflate the degree).
- **Sign in the digraph Laplacian.** In-degree vs out-degree on the diagonal decides "arborescence toward the root" vs "away from the root". Match it to the problem.

## The keystone

The Laplacian $L = D - A$ is singular, but every $(n-1)\times(n-1)$ cofactor equals the number of spanning trees — so one $O(n^3)$ determinant counts them. Weighting the Laplacian entries counts weighted trees; deleting the root's row/column and using degrees-in counts directed spanning arborescences.
