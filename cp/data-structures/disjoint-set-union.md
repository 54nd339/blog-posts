---
title: Disjoint Set Union - Union by Rank and Path Compression
description: The near-constant-time structure for "are these connected" under a stream of merges — the forest representation, why the two optimisations together give inverse-Ackermann time, and the rollback variant for offline deletion.
date: 2024-03-14
draft: false
slug: /cp/disjoint-set-union
tags:
  - Competitive Programming
  - Data Structures
  - Union-Find
---

You have $n$ elements and a stream of "merge group of $a$ with group of $b$" operations, interleaved with "are $a$ and $b$ in the same group?" queries. Disjoint set union answers both in effectively constant time, using a forest where each tree is one group and each root names it.

## The problem

Maintain a partition of $\{0, 1, \dots, n-1\}$ under:

- `find(x)` — return a canonical representative of $x$'s set,
- `union(a, b)` — merge the sets containing $a$ and $b$.

`find(a) == find(b)` answers "same set?". A component count is maintained by decrementing on each *effective* union. Example: start with $\{0\}\{1\}\{2\}\{3\}$; `union(0,1)`, `union(2,3)`, `union(1,2)` leaves one set $\{0,1,2,3\}$.

## The idea

Represent each set as a rooted tree via a `parent` array; the root is its own parent and is the representative. `find` walks to the root; `union` links one root under the other. Two optimisations make it fast:

- **Union by size (or rank):** always attach the smaller tree under the larger root. Keeps trees shallow — height $O(\log n)$ on its own.
- **Path compression:** during `find`, re-point every node on the path directly to the root. Future `find`s on those nodes are $O(1)$.

Together, $m$ operations on $n$ elements run in $O(m\,\alpha(n))$, where $\alpha$ is the inverse Ackermann function — below $5$ for any $n$ that fits in the universe.

## How it works

`union(0,1)`: roots $0, 1$, equal size, attach $1$ under $0$; sizes $\{0: 2\}$. `union(2,3)`: attach $3$ under $2$. `union(1,2)`: `find(1) = 0`, `find(2) = 2`, both size $2$, attach $2$ under $0$. Now `find(3)`: $3 \to 2 \to 0$; path compression re-points $3$ and $2$ straight to $0$, so the next `find(3)` is one hop.

## Algorithm

```python
class DSU:
    def __init__(self, n: int):
        self.parent = list(range(n))
        self.size = [1] * n
        self.components = n

    def find(self, x: int) -> int:
        root = x
        while self.parent[root] != root:
            root = self.parent[root]
        while self.parent[x] != root:          # path compression
            self.parent[x], x = root, self.parent[x]
        return root

    def union(self, a: int, b: int) -> bool:
        a, b = self.find(a), self.find(b)
        if a == b:
            return False                       # already together
        if self.size[a] < self.size[b]:
            a, b = b, a
        self.parent[b] = a
        self.size[a] += self.size[b]
        self.components -= 1
        return True
```

The iterative two-pass `find` avoids recursion-depth limits; a recursive one-liner `parent[x] = find(parent[x])` is also common.

## Complexity

- **Time:** $O(\alpha(n))$ amortised per `find`/`union` with both optimisations; $O(\log n)$ with only one of them; $O(n)$ worst case with neither.
- **Space:** $O(n)$ for `parent` and `size`.

## Variations

- **DSU with rollback.** Skip path compression, keep only union by size, and push each `parent`/`size` change onto a stack. `rollback()` pops to a saved checkpoint. `find` becomes $O(\log n)$, but you gain undo — the basis of offline dynamic connectivity (segment tree on the timeline) and of ["deletion in $O(T(n)\log n)$"](/citadel/cp/deleting-in-log-n).
- **Weighted / bipartite DSU.** Store the parity (or a group element) of the edge to the parent; `find` accumulates it. Answers "are $a$ and $b$ in the same part / at what relative offset" — used for [bipartiteness under unions](/citadel/cp/bipartite-matching) and difference-constraint problems.
- **DSU on the small-to-large trick.** Merging sets by always iterating the smaller one totals $O(n \log n)$ element moves — the same principle, applied to explicit set contents rather than just roots.
- **Partial persistence / DSU on tree.** Kept around for subtree-merge counting problems.

## Common pitfalls

- **Union without `find` first.** `parent[b] = a` where `a`, `b` are not roots corrupts the forest. Always `a, b = find(a), find(b)`.
- **Comparing raw ids.** "Same set" is `find(a) == find(b)`, never `a == b` or `parent[a] == parent[b]`.
- **Rollback with path compression.** Compression makes changes you cannot cheaply undo; the rollback variant must drop it.
- **Recursion depth.** A recursive `find` on an adversarial chain (no compression yet) can hit $10^5$+ depth. Use the iterative form or raise the limit.
- **Counting effective unions.** Decrement the component count only when `union` actually merges two different sets (return value `True`).

## The keystone

DSU is a forest of "point to your representative" links, kept shallow by attaching small trees under large ones and flattened further by path compression — together, inverse-Ackermann time per operation. Dropping compression for a change-stack buys `rollback`, which is how offline problems simulate edge *deletion*.
