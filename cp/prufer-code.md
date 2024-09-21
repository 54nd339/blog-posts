---
title: Prufer Code - Encoding Labeled Trees as Sequences
description: The bijection between labeled trees on n vertices and sequences of n-2 labels, giving Cayley's formula for free and O(n) conversion in both directions using a leaf-priority sweep.
date: 2024-09-21
draft: false
slug: /cp/prufer-code
tags:
  - Competitive Programming
  - Graphs
  - Trees
---

A Prüfer code turns a labeled tree on $n$ vertices into a sequence of $n - 2$ vertex labels — and back. Because the map is a **bijection** with $\{1, \dots, n\}^{n-2}$, it instantly proves Cayley's formula ($n^{n-2}$ labeled trees) and lets you sample, count, or rank labeled trees.

## The problem

- **Encode:** given a labeled tree (vertices $0 \dots n-1$), produce its Prüfer sequence of length $n - 2$.
- **Decode:** given a sequence in $\{0, \dots, n-1\}^{n-2}$, reconstruct the unique tree.
- **Consequences:** count labeled trees ($n^{n-2}$), labeled forests with $k$ trees on specified roots ($k\, n^{n-k-1}$), trees with prescribed degrees ($\binom{n-2}{d_1 - 1,\ \dots,\ d_n - 1}$ — a multinomial, since vertex $v$ appears $\deg(v) - 1$ times in the code).

Example: the path $0 - 1 - 2 - 3$ has Prüfer code $[1, 2]$.

## The idea

**Encoding** — repeat $n - 2$ times: find the **leaf with the smallest label**, append its unique neighbour to the code, delete the leaf. The last two vertices are left implicit.

**Decoding** — the number of times a label appears in the code is its degree minus $1$; so a vertex *not* in the remaining code is currently a leaf. Repeat: take the smallest-labelled vertex whose "remaining degree" is $1$ (or $0$), connect it to the next code entry, decrement that entry's remaining count. At the end, join the two vertices still at degree $1$.

A min-heap (or a pointer that only moves forward, exploiting a monotonicity) makes both directions $O(n \log n)$, or $O(n)$ with the pointer trick.

## Algorithm

```python
import heapq

def tree_to_prufer(n, adj):
    degree = [len(adj[v]) for v in range(n)]
    alive = [True] * n
    leaves = [v for v in range(n) if degree[v] == 1]
    heapq.heapify(leaves)
    code = []
    for _ in range(n - 2):
        leaf = heapq.heappop(leaves)
        alive[leaf] = False
        nb = next(x for x in adj[leaf] if alive[x])      # the one remaining neighbour
        code.append(nb)
        degree[nb] -= 1
        if degree[nb] == 1:
            heapq.heappush(leaves, nb)
    return code

def prufer_to_tree(n, code):
    degree = [1] * n
    for x in code:
        degree[x] += 1
    leaves = [v for v in range(n) if degree[v] == 1]
    heapq.heapify(leaves)
    edges = []
    for x in code:
        leaf = heapq.heappop(leaves)
        edges.append((leaf, x))
        degree[leaf] -= 1
        degree[x] -= 1
        if degree[x] == 1:
            heapq.heappush(leaves, x)
    u = v = -1
    for w in range(n):
        if degree[w] == 1:
            if u == -1:
                u = w
            else:
                v = w
    edges.append((u, v))
    return edges
```

The neighbour-lookup in `tree_to_prufer` is written awkwardly above; the clean version tracks, per still-alive vertex, the sum of its neighbours' labels or uses an adjacency set and removes the leaf. A common $O(n)$ implementation keeps `parent` pointers from a root and a pointer `ptr` scanning for the next leaf.

## An O(n) encoding

Root the tree at vertex $n - 1$, compute `parent[v]` for all $v$. Keep a pointer `ptr` at the smallest-index leaf. Repeat $n - 2$ times: let `leaf` be the current smallest unprocessed leaf; append `parent[leaf]`; decrement `parent[leaf]`'s child count; if it becomes a leaf **and** its index is `< ptr`, process it immediately (it is the new smallest); otherwise advance `ptr` to the next leaf. Amortised $O(n)$.

## Complexity

- **Encode / decode:** $O(n \log n)$ with a heap, $O(n)$ with the pointer trick.
- **Space:** $O(n)$.

## Uses

- **Cayley's formula:** $|\{0, \dots, n-1\}^{n-2}| = n^{n-2}$ labeled trees — the bijection is the proof.
- **Random labeled tree:** generate a random length-$(n-2)$ sequence, decode. Uniform over all labeled trees.
- **Count trees with degree constraints:** vertex $v$ occurs $\deg(v) - 1$ times, so the count with degrees $d_1, \dots, d_n$ (each $\ge 1$, summing to $2n - 2$) is the multinomial $\binom{n-2}{d_1 - 1, \dots, d_n - 1}$.
- **Rank / unrank** a labeled tree via its code as a mixed-radix number.
- **[Kirchhoff cross-check](/citadel/cp/kirchhoff-theorem):** for $K_n$, both give $n^{n-2}$.

## Common pitfalls

- **$n \le 2$.** The code has length $\max(0, n - 2)$: empty for $n = 1$ or $2$. The single edge (for $n = 2$) is implicit.
- **Label range in decode.** Every entry of the input sequence must be in $[0, n-1]$; a valid sequence always decodes to a valid tree, but validate untrusted input.
- **Smallest-label leaf, consistently.** Both directions must pick the minimum-label current leaf. Using a stack or arbitrary order breaks the bijection (you would still get *a* tree, but not *the* one the code encodes).
- **Degree bookkeeping in decode.** `degree[v]` starts at `1 + (count of v in code)`; decrement both endpoints each step; the two left at degree $1$ get the final edge.
- **1-indexed vs 0-indexed.** cp-algorithms uses 1-indexed; be consistent with your I/O.

## The keystone

Repeatedly removing the smallest-labelled leaf and recording its neighbour encodes a labeled tree as an $(n-2)$-sequence; the reverse reads off which vertex is currently a leaf from "not appearing later in the code". The bijection with $\{1, \dots, n\}^{n-2}$ is Cayley's formula and a uniform random-tree generator.
