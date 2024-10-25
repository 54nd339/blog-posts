---
title: Heavy-Light Decomposition - Path Queries on a Tree
description: Cutting a tree into vertex-disjoint chains so that any root-to-node path meets O(log n) of them, which turns "sum/min/max on the path between u and v" into O(log^2 n) segment-tree queries.
date: 2024-10-25
draft: false
slug: /cp/heavy-light-decomposition
tags:
  - Competitive Programming
  - Graphs
  - Trees
---

A segment tree answers range queries on an array. Heavy-light decomposition (HLD) makes a tree *look* like an array for the purpose of path queries: it partitions the vertices into chains laid out contiguously, such that the path between any two vertices spans only $O(\log n)$ chain-segments. Path sum, path max, "assign along a path", subtree queries — all become a handful of segment-tree operations.

## The problem

Support, on a rooted tree with values on vertices (or edges):

- **update** a vertex/edge value, or a whole path;
- **query** an aggregate (sum, min, max, gcd, ...) over the path between $u$ and $v$.

Both in $O(\log^2 n)$. The [LCA structures](/citadel/cp/lowest-common-ancestor) handle "where do the paths meet"; HLD also handles "what is *on* the path".

## The idea: heavy edges and chains

For each vertex, its **heavy child** is the child with the largest subtree; every other child edge is **light**. Key fact: walking from the root to any vertex, you cross at most $\lfloor \log_2 n \rfloor$ light edges, because each light edge you descend at least halves the remaining subtree size.

Chain the heavy edges together: a **heavy path** is a maximal downward run of heavy edges. The tree decomposes into vertex-disjoint heavy paths connected by light edges, and any root-to-node path alternates "walk up a heavy path" / "cross one light edge" $O(\log n)$ times.

Now number the vertices by a DFS that **visits the heavy child first**. Then every heavy path occupies a contiguous block of indices, and every subtree occupies a contiguous block too. Build a segment tree over that index array.

## How a path query works

To query the path $u \to v$: repeatedly take whichever of $u, v$ sits in the chain with the **deeper head** (top vertex), query the segment from that head down to the vertex, and jump to the parent of the head. When both are in the same chain, query the segment between them. Each jump crosses one light edge, so $O(\log n)$ segment queries, each $O(\log n)$ — $O(\log^2 n)$ total. (This also computes the LCA: it is the shallower of $u, v$ when they finally share a chain.)

## Algorithm

```python
class HLD:
    def __init__(self, n, adj, root=0):
        self.n, self.adj = n, adj
        self.parent = [-1] * n
        self.depth = [0] * n
        self.size = [1] * n
        self.heavy = [-1] * n
        self.head = [0] * n
        self.pos = [0] * n

        order, stack, seen = [], [root], [False] * n
        seen[root] = True
        while stack:                              # 1. parent, depth, DFS order
            u = stack.pop(); order.append(u)
            for w in adj[u]:
                if not seen[w]:
                    seen[w] = True
                    self.parent[w] = u
                    self.depth[w] = self.depth[u] + 1
                    stack.append(w)

        for u in reversed(order):                 # 2. subtree size, heavy child
            best = 0
            for w in adj[u]:
                if w != self.parent[u]:
                    self.size[u] += self.size[w]
                    if self.size[w] > best:
                        best, self.heavy[u] = self.size[w], w

        cur = 0
        stack = [(root, root)]                    # 3. lay out chains (heavy child first)
        while stack:
            v, h = stack.pop()
            while v != -1:
                self.head[v] = h
                self.pos[v] = cur; cur += 1
                for w in adj[v]:
                    if w != self.parent[v] and w != self.heavy[v]:
                        stack.append((w, w))     # each light child starts a new chain
                v = self.heavy[v]

    def path_query(self, seg, u, v):
        res = seg.identity
        while self.head[u] != self.head[v]:
            if self.depth[self.head[u]] < self.depth[self.head[v]]:
                u, v = v, u
            res = seg.op(res, seg.query(self.pos[self.head[u]], self.pos[u] + 1))
            u = self.parent[self.head[u]]
        lo, hi = (u, v) if self.depth[u] < self.depth[v] else (v, u)
        return seg.op(res, seg.query(self.pos[lo], self.pos[hi] + 1))

    def lca(self, u, v):
        while self.head[u] != self.head[v]:
            if self.depth[self.head[u]] < self.depth[self.head[v]]:
                u, v = v, u
            u = self.parent[self.head[u]]
        return u if self.depth[u] < self.depth[v] else v
```

Pair it with any segment tree exposing `query(l, r)` on $[l, r)$, an associative `op`, and its `identity`. For **edge values**, store each edge's value at its lower (deeper) endpoint and, in `path_query`, skip the LCA itself on the final chain: `seg.query(self.pos[lo] + 1, self.pos[hi] + 1)`.

## Complexity

- **Build:** $O(n)$ (three linear passes) plus the segment tree's $O(n)$.
- **Path query / path update:** $O(\log^2 n)$ — $O(\log n)$ chains, each an $O(\log n)$ segment-tree call.
- **Subtree query:** $O(\log n)$ — one range `[pos[v], pos[v] + size[v])`.
- **Space:** $O(n)$.

## Common pitfalls

- **Not visiting the heavy child first.** If the heavy child is not numbered immediately after its parent, heavy paths are no longer contiguous and every query is wrong.
- **Edge values vs vertex values.** With edge values, the LCA carries no edge on the path — exclude it (`pos[lo] + 1`). Off by one here double-counts or drops an edge.
- **Choosing the shallower-head chain.** You must ascend from the vertex whose *head* is deeper, not whose vertex is deeper. Comparing `depth[u]` vs `depth[v]` instead of `depth[head[u]]` vs `depth[head[v]]` can loop forever.
- **Non-commutative `op` (e.g. matrix product along a path).** The two halves $u \to \text{lca}$ and $\text{lca} \to v$ are traversed in opposite directions; keep separate left/right accumulators and combine them in the right order at the end.
- **Recursion depth.** A path graph has depth $n$; recursive builds overflow. The passes above are iterative.

## The keystone

Mark the largest-subtree child of every vertex as heavy; heavy edges link into chains, and any root-to-node path crosses only $O(\log n)$ light edges between them. Number vertices heavy-child-first so each chain is a contiguous array segment, then a path query is $O(\log n)$ segment-tree range queries — $O(\log^2 n)$ — and the LCA falls out for free.
