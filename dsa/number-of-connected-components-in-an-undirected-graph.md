---
title: Number of Connected Components - Start at n, Subtract One Per Merge
description: Counting connected components of an undirected graph — union-find starting from n singletons and decrementing on each real merge, or a visited-marking DFS sweep.
date: 2022-01-13
draft: false
slug: /dsa/number-of-connected-components-in-an-undirected-graph
tags:
  - Competitive Programming
  - Graphs
  - Union Find
---

Begin with `n` components, one per node. Every edge that joins two *different* components reduces the count by one; an edge inside a component changes nothing. Equivalently, sweep the nodes and run one DFS per unvisited node.

## Description

Given `n` nodes labelled `0` to `n - 1` and a list of undirected `edges`, return the number of connected components.

**Example**

```
Input:  n = 5, edges = [[0,1],[1,2],[3,4]]
Output: 2

Input:  n = 5, edges = [[0,1],[1,2],[2,3],[3,4]]
Output: 1
```

**Constraints**

- $1 \le n \le 2000$
- $0 \le \text{edges.length} \le 5000$; no self-loops, no duplicates.

## Prerequisites

- Union-Find with a live component counter.
- Or DFS/BFS with a visited set.

## Approach 1: Union-Find

### Intuition

`count = n`. For each edge, if its endpoints have different roots, union them and `count -= 1`. Redundant edges (same root) are ignored.

### Algorithm

1. DSU over `n`, `count = n`.
2. For each `(u, v)`: `ru, rv = find(u), find(v)`; if `ru != rv`, attach one root to the other and `count -= 1`.
3. Return `count`.

```python
def countComponents(n: int, edges: list[list[int]]) -> int:
    parent = list(range(n))
    rank = [1] * n
    count = n

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    for u, v in edges:
        ru, rv = find(u), find(v)
        if ru == rv:
            continue
        if rank[ru] < rank[rv]:
            ru, rv = rv, ru
        parent[rv] = ru
        rank[ru] += rank[rv]
        count -= 1

    return count
```

### Complexity

- **Time:** $O((n + e) \cdot \alpha(n))$.
- **Space:** $O(n)$.

## Approach 2: DFS sweep

### Intuition

Build an adjacency list. Walk nodes `0 .. n-1`; each time you find an unvisited node, that is a new component — run a DFS to mark its whole component, then `count += 1`.

### Algorithm

1. Build `adj`. `visited = set()`, `count = 0`.
2. For `i` in `range(n)`: if `i not in visited`, `dfs(i)`, `count += 1`.
3. `dfs(u)`: add `u` to `visited`; recurse into unvisited neighbours.

```python
def countComponents(n: int, edges: list[list[int]]) -> int:
    adj = [[] for _ in range(n)]
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)

    visited = set()

    def dfs(u):
        visited.add(u)
        for nei in adj[u]:
            if nei not in visited:
                dfs(nei)

    count = 0
    for i in range(n):
        if i not in visited:
            dfs(i)
            count += 1
    return count
```

### Complexity

- **Time:** $O(n + e)$.
- **Space:** $O(n + e)$.

## Approach 3: BFS sweep

### Intuition

Identical to the DFS sweep with a queue — avoids recursion depth on a long chain.

### Algorithm

For each unvisited node, BFS its component (mark on enqueue), then `count += 1`.

```python
from collections import deque

def countComponents(n: int, edges: list[list[int]]) -> int:
    adj = [[] for _ in range(n)]
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)

    visited = set()
    count = 0
    for i in range(n):
        if i in visited:
            continue
        count += 1
        visited.add(i)
        q = deque([i])
        while q:
            u = q.popleft()
            for nei in adj[u]:
                if nei not in visited:
                    visited.add(nei)
                    q.append(nei)
    return count
```

### Complexity

- **Time:** $O(n + e)$.
- **Space:** $O(n + e)$.

## Common Pitfalls

- **Decrementing on every edge.** Only edges that merge distinct roots reduce the count; skip when `find(u) == find(v)`.
- **Starting the count at 0 for Union-Find.** It starts at `n` and goes down.
- **Forgetting isolated nodes.** Nodes with no edges are their own components; the `0 .. n-1` sweep (or the initial `count = n`) accounts for them.
- **Marking visited on dequeue in BFS.** Mark on enqueue to avoid a node entering the queue from two neighbours.

## The keystone

"How many components" is the counting form of the same union-find / flood sweep that [Number of Islands](/citadel/dsa/number-of-islands) runs on a grid. The abstraction — nodes and an edge list instead of cells and adjacency — is the only difference.
