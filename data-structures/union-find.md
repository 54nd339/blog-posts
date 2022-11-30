---
title: Disjoint Set Union - Union by Rank and Path Compression
description: Union-Find tracks a partition of items into groups under two operations - merge two groups, and ask whether two items share a group. With two small optimizations it runs in almost constant time per operation, which is why it sits inside Kruskal's algorithm and every "are these connected?" query.
date: 2022-11-30
draft: false
slug: /data-structures/union-find
tags:
  - Data Structures
  - Graphs
---

You're processing a stream of "user A friended user B" events and, in between, questions like "are users X and Y in the same friend cluster?" The clusters keep merging as edges arrive. You never split a cluster, only join.

Rebuilding connectivity with a graph traversal after every edge is O(V + E) per query — far too slow. What you want is a structure that maintains a **partition** of the items into disjoint groups and supports exactly two operations: `union(a, b)` merges the groups containing `a` and `b`, and `find(a)` returns a canonical identifier for `a`'s group so that `find(x) == find(y)` answers "same group?". That structure is **Union-Find**, also called a **disjoint set union** (DSU). With two cheap tricks it does both operations in effectively constant time.

## Groups as trees

Represent each group as a tree, where every node points to its parent and the **root** is the group's representative. `find(x)` walks parent pointers up to the root. `union(a, b)` finds both roots and points one at the other.

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))     # everyone starts as their own root

    def find(self, x):
        while self.parent[x] != x:
            x = self.parent[x]
        return x

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra != rb:
            self.parent[ra] = rb
```

Correct, but a naive `union` can build a degenerate chain: `union(0,1), union(1,2), union(2,3), …` makes a linked list, and `find` on the deep end is O(n). Two optimizations fix this, and together they're what make DSU fast.

## Optimization 1: union by rank (or size)

When merging two trees, always hang the **shorter** one under the taller one. If you attach the taller under the shorter, the result is one level deeper than it needed to be; the other way, the height doesn't grow at all unless both trees were the same height.

Track an upper bound on each tree's height, called its **rank** (it's only a bound, because the next optimization flattens trees without updating rank).

```python
def __init__(self, n):
    self.parent = list(range(n))
    self.rank = [0] * n

def union(self, a, b):
    ra, rb = self.find(a), self.find(b)
    if ra == rb:
        return
    if self.rank[ra] < self.rank[rb]:
        ra, rb = rb, ra              # ensure ra is the taller root
    self.parent[rb] = ra
    if self.rank[ra] == self.rank[rb]:
        self.rank[ra] += 1
```

A tree's rank only increases when two equal-rank trees merge, which requires doubling the node count. So a rank-`r` tree has at least `2^r` nodes, which caps rank — and therefore `find` — at O(log n).

**Union by size** is the common alternative: track each tree's node count and hang the smaller tree under the larger. Same O(log n) bound, and the size is often useful on its own (how big is this cluster?).

## Optimization 2: path compression

`find` already walks from a node to its root. On the way back, point every node it passed *directly at the root*. The next `find` on any of them is a single hop.

```python
def find(self, x):
    root = x
    while self.parent[root] != root:
        root = self.parent[root]
    while self.parent[x] != root:        # second pass: re-point the path
        self.parent[x], x = root, self.parent[x]
    return root
```

The one-liner recursive form does the same thing:

```python
def find(self, x):
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])
    return self.parent[x]
```

Each `find` pays a bit extra to shorten the path, and every future `find` through those nodes is cheaper. It's the same amortised bargain as resizing a [hash table](/citadel/data-structures/hash-tables): occasional linear work spread over many fast operations.

## Why it's "almost" O(1)

With **both** optimizations, a sequence of `m` operations on `n` elements runs in `O(m · α(n))`, where `α` is the **inverse Ackermann function**. Ackermann's function grows so violently that its inverse grows unimaginably slowly: `α(n) ≤ 4` for any `n` that could be written down in this universe — `n` up to roughly `2^65536`. So per operation the cost is bounded by a small constant. It is provably *not* a true constant (Tarjan showed the `α` factor is unavoidable for this problem), but no input you will ever run distinguishes it from one.

With only one of the two tricks you get O(log n) per operation; with just path compression and no union rule you get O(log n) amortised. You want both.

## Where it shows up

**Kruskal's MST.** Sort the edges by weight and add each one whose endpoints are in different components — `if find(u) != find(v): union(u, v)`. DSU *is* the cycle check. See [minimum spanning tree](/citadel/algorithms/MinimumSpanningTree).

**Connected components, offline.** Any batch of "connect these, then answer connectivity queries" — network reachability, image flood-fill labelling, maze generation, percolation — is a DSU. It beats BFS/DFS whenever edges arrive incrementally and you never need to *remove* one.

**Cycle detection in an undirected graph.** Process edges; the first edge whose endpoints already share a root closes a cycle.

**Equivalence under constraints.** "Variable `a` equals `b`, `b` equals `c`, `d` is not `a` — is this satisfiable?" Union all the equalities first, then check that no inequality has both sides in one group.

Two limits worth stating. DSU handles **union only** — there is no efficient `split`. And to answer *offline* problems it's unbeatable, but a truly *online* fully-dynamic connectivity structure (with edge deletions) needs heavier machinery like Euler-tour trees. When your problem is "merge forward, query connectivity," DSU is the answer.

## The one idea to keep

Union-Find keeps a family of merging groups and answers "same group?" in effectively constant time — but only with *both* tricks: attach the smaller tree under the bigger one so trees stay shallow, and flatten every path you walk during `find` so it stays that way. Whenever a problem is a stream of "join these two" mixed with "are these joined," reach for DSU before you reach for a graph traversal.
