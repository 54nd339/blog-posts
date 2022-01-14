---
title: Redundant Connection - The Edge That Closes the Loop
description: Finding the one extra edge that turns a tree into a graph with a cycle — union-find each edge in order and return the first whose endpoints are already connected.
date: 2022-01-14
draft: false
slug: /dsa/redundant-connection
tags:
  - Competitive Programming
  - Graphs
  - Union Find
---

The input is a tree plus exactly one extra edge, so it has exactly one cycle. Add edges one at a time with union-find; the first edge whose two endpoints are already in the same component is the one that created the cycle. The problem wants the *last* such edge in input order, and processing in order gives exactly that.

## Description

A graph started as a tree with `n` nodes (`1` to `n`) and had one extra edge added. `edges[i] = [a, b]`. Return the edge that can be removed so the result is a tree with `n` nodes. If several answers exist, return the one that appears last in `edges`.

**Example**

```
Input:  edges = [[1,2],[1,3],[2,3]]
Output: [2,3]

Input:  edges = [[1,2],[2,3],[3,4],[1,4],[1,5]]
Output: [1,4]
```

**Constraints**

- $n == \text{edges.length}$, $3 \le n \le 1000$
- No self-loops or repeated edges; the graph is connected.

## Prerequisites

- Union-Find with path compression and union by rank/size.
- The fact that exactly one edge is redundant, so the first "already connected" edge is the answer.

## Approach 1: Union-Find

### Intuition

For each edge in input order, try to union its endpoints. If `find(a) == find(b)` already, this edge is redundant — return it. Because there is exactly one extra edge, this fires exactly once and on the correct (last) edge.

### Algorithm

1. DSU over `n + 1` (1-indexed).
2. For each `(a, b)`: if `find(a) == find(b)`, return `[a, b]`; else `union(a, b)`.

```python
def findRedundantConnection(edges: list[list[int]]) -> list[int]:
    n = len(edges)
    parent = list(range(n + 1))
    rank = [1] * (n + 1)

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    for a, b in edges:
        ra, rb = find(a), find(b)
        if ra == rb:
            return [a, b]
        if rank[ra] < rank[rb]:
            ra, rb = rb, ra
        parent[rb] = ra
        rank[ra] += rank[rb]

    return []
```

### Complexity

- **Time:** $O(n \cdot \alpha(n))$.
- **Space:** $O(n)$.

## Approach 2: DFS per edge

### Intuition

For each edge `(a, b)`, before adding it, check with DFS on the graph built so far whether `a` and `b` are already connected. If so, `(a, b)` is redundant. Otherwise add it.

### Algorithm

1. `adj` = empty adjacency list.
2. For each `(a, b)`: run `dfs(a, b, visited)` over the current `adj`. If it returns `True`, return `[a, b]`. Otherwise add `b` to `adj[a]` and `a` to `adj[b]`.
3. `dfs(cur, target, visited)`: if `cur == target`, `True`; mark visited; recurse into unvisited neighbours.

```python
def findRedundantConnection(edges: list[list[int]]) -> list[int]:
    adj = {}

    def dfs(cur, target, visited):
        if cur == target:
            return True
        visited.add(cur)
        for nei in adj.get(cur, ()):
            if nei not in visited and dfs(nei, target, visited):
                return True
        return False

    for a, b in edges:
        if a in adj and b in adj and dfs(a, b, set()):
            return [a, b]
        adj.setdefault(a, []).append(b)
        adj.setdefault(b, []).append(a)

    return []
```

### Complexity

- **Time:** $O(n^2)$ — up to `n` edges, each an $O(n)$ DFS.
- **Space:** $O(n)$.

## Common Pitfalls

- **Processing edges out of order.** The "last redundant edge" requirement is satisfied only by scanning `edges` front to back and returning on the first cycle-closer.
- **0-indexed DSU.** Nodes are `1 .. n`; size the parent array `n + 1`.
- **Adding the edge before the check in Approach 2.** Test connectivity on the graph *without* the current edge, then add it.
- **Union by nothing.** Skipping union-by-rank/size still works here (small `n`) but path compression alone keeps it fast.

## The keystone

The union-find cycle test — "these two are already connected, so this edge is redundant" — is the atom that [Graph Valid Tree](/citadel/dsa/graph-valid-tree) uses to *reject* and this problem uses to *select*. Same line of code, opposite intent. Advanced graph problems build MSTs ([Min Cost to Connect All Points](/citadel/dsa/min-cost-to-connect-all-points)) on the same primitive.
