---
title: Finding Bridges Online - Incremental 2-Edge-Connectivity
description: Maintaining the bridge count of a graph while edges are added one at a time, using two disjoint-set structures - one for 2-edge-connected components and one for connected components - plus a parent forest and a march-to-LCA merge.
date: 2024-09-16
draft: false
slug: /cp/finding-bridges-online
tags:
  - Competitive Programming
  - Graphs
  - Connectivity
---

The [offline bridge algorithm](/citadel/cp/bridges-and-cut-vertices) needs the whole graph up front. When edges arrive one at a time and you must report the number of bridges after each insertion, you need an **online** structure. The trick: collapse every 2-edge-connected component into a single node, so the graph is always a forest of these components, and a bridge is exactly a tree edge of that forest.

## The problem

Start with $n$ isolated vertices. Process a stream of edge insertions; after each one, answer "how many bridges does the graph have right now?" Edges are never deleted (that is the much harder fully dynamic problem).

Example: add $(0,1)$ → 1 bridge. Add $(1,2)$ → 2 bridges. Add $(2,0)$ → the cycle $0\text{-}1\text{-}2$ merges into one 2-edge-connected component → 0 bridges. Add $(2,3)$ → 1 bridge.

## The idea

Keep two disjoint-set-union structures over the vertices:

- **`dsu_2ecc`** — groups vertices that are in the same 2-edge-connected component (2ECC). Its representatives are the "super-nodes".
- **`dsu_cc`** — groups super-nodes that are in the same connected component.

On the super-nodes, maintain a **parent pointer `par[]`**, so each connected component is an explicitly rooted tree. Every edge of that tree is a bridge, so `bridges` = (number of super-nodes) − (number of connected components) is maintained incrementally.

When you add an edge between vertices $a$ and $b$, reduce to their super-nodes $A = \text{find\_2ecc}(a)$, $B = \text{find\_2ecc}(b)$:

1. **$A = B$** — both endpoints already in the same 2ECC. Nothing changes.
2. **$A, B$ in different connected components** — the edge is a new bridge. Re-root the smaller component's tree at $A$, then hang $A$ under $B$. `bridges += 1`.
3. **$A, B$ in the same component but different 2ECCs** — the tree path from $A$ to $B$ plus this edge forms a cycle, so every super-node on that path merges into one 2ECC. Walk up from both ends to their LCA, union them in `dsu_2ecc`, and `bridges -= 1` for each tree edge swallowed.

## How it works: the path merge

To merge the path in case 3 without knowing the LCA in advance, march upward from $A$ and $B$ **one step at a time, alternately**, stamping each visited super-node with the current query id. The first already-stamped node you hit is the LCA. Then relabel every node on both collected paths (up to and including the LCA) to point at the LCA in `dsu_2ecc`, decrementing `bridges` once per edge collapsed.

Re-rooting a tree (case 2) reverses every parent pointer on the path from the chosen node to the old root — $O(\text{size})$, but by always re-rooting the **smaller** component each insertion contributes $O(\log n)$ amortised, so the whole sequence is near-linear.

## Algorithm

```python
class OnlineBridges:
    def __init__(self, n):
        self.n = n
        self.par = [-1] * n                     # parent in the super-node forest
        self.dsu_2ecc = list(range(n))          # 2-edge-connected components
        self.dsu_cc = list(range(n))            # connected components (over super-nodes)
        self.cc_size = [1] * n
        self.last_visit = [0] * n               # LCA stamp per super-node
        self.iteration = 0
        self.bridges = 0

    def find_2ecc(self, v):
        if v == -1:
            return -1
        while self.dsu_2ecc[v] != v:
            self.dsu_2ecc[v] = self.dsu_2ecc[self.dsu_2ecc[v]]
            v = self.dsu_2ecc[v]
        return v

    def find_cc(self, v):
        v = self.find_2ecc(v)
        while self.dsu_cc[v] != v:
            self.dsu_cc[v] = self.dsu_cc[self.dsu_cc[v]]
            v = self.dsu_cc[v]
        return v

    def make_root(self, v):
        v = self.find_2ecc(v)
        root, child = v, -1
        while v != -1:
            nxt = self.find_2ecc(self.par[v])
            self.par[v] = child
            self.dsu_cc[v] = root
            child, v = v, nxt
        self.cc_size[root] = self.cc_size[child]

    def merge_path(self, a, b):
        self.iteration += 1
        path_a, path_b, lca = [], [], -1
        while lca == -1:
            if a != -1:
                a = self.find_2ecc(a)
                path_a.append(a)
                if self.last_visit[a] == self.iteration:
                    lca = a
                    break
                self.last_visit[a] = self.iteration
                a = self.par[a]
            if b != -1:
                b = self.find_2ecc(b)
                path_b.append(b)
                if self.last_visit[b] == self.iteration:
                    lca = b
                    break
                self.last_visit[b] = self.iteration
                b = self.par[b]
        for path in (path_a, path_b):
            for v in path:
                self.dsu_2ecc[v] = lca
                if v == lca:
                    break
                self.bridges -= 1

    def add_edge(self, a, b):
        a, b = self.find_2ecc(a), self.find_2ecc(b)
        if a == b:
            return                              # already 2-edge-connected
        ca, cb = self.find_cc(a), self.find_cc(b)
        if ca != cb:
            self.bridges += 1
            if self.cc_size[ca] > self.cc_size[cb]:
                a, b, ca, cb = b, a, cb, ca
            self.make_root(a)
            self.par[a] = self.dsu_cc[a] = b
            self.cc_size[cb] += self.cc_size[ca]
        else:
            self.merge_path(a, b)
```

## Complexity

- **Per insertion:** amortised $O(\log n)$ for the re-rooting (small-to-large) plus near-constant DSU work; the path merge is charged to the bridges it destroys, and a bridge is destroyed at most once.
- **Total for $m$ insertions:** $O(n + m \log n)$, effectively linear.
- **Space:** $O(n)$.

## Common pitfalls

- **`find_2ecc(-1)`.** Parent pointers use $-1$ for "no parent". In Python, `array[-1]` is a valid index (the last element), so an unchecked `find_2ecc(par[v])` silently corrupts the structure or loops forever. Guard the $-1$ explicitly.
- **Re-rooting the larger side.** Always `make_root` the smaller connected component; re-rooting the larger one turns the amortised bound into $O(nm)$.
- **Two DSUs, not one.** `dsu_cc` operates on super-nodes, so `find_cc` must call `find_2ecc` first. Collapsing both into one structure loses the "which super-nodes form a tree" information.
- **LCA march must alternate.** Advancing $a$ fully to the root before touching $b$ still finds a common ancestor, but not the *lowest* one, so you merge nodes that should stay separate. Step the two pointers one at a time.
- **Deletions.** This structure is insert-only. Edge deletions need a fully dynamic 2-edge-connectivity structure (Holm-de Lichtenberg-Thorup) or an offline divide-and-conquer over the timeline.

## The keystone

Contract every 2-edge-connected component to a point and the graph becomes a forest whose every edge is a bridge. Two DSUs — one for the contraction, one for connectivity — plus a re-rootable parent forest let you add an edge, collapse the cycle it closes by marching both endpoints to their LCA, and keep an exact bridge count in near-linear total time.
