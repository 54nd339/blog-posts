---
title: Graph Valid Tree - Connected and Exactly n Minus One Edges
description: Whether an undirected graph is a tree — check it has n minus one edges and is fully connected, or run union-find and reject any edge that joins two already-connected nodes.
date: 2022-01-12
draft: false
slug: /dsa/graph-valid-tree
tags:
  - Competitive Programming
  - Graphs
  - Union Find
---

A graph on `n` nodes is a tree iff it is connected and has no cycle. Two equivalent shortcuts: it has exactly `n - 1` edges *and* is connected; or, adding edges one at a time, no edge ever connects two nodes already in the same component.

## Description

Given `n` nodes labelled `0` to `n - 1` and a list of undirected `edges`, return `true` if the edges form a valid tree.

**Example**

```
Input:  n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]
Output: true

Input:  n = 5, edges = [[0,1],[1,2],[2,3],[1,3],[1,4]]
Output: false
```

**Constraints**

- $1 \le n \le 2000$
- $0 \le \text{edges.length} \le 5000$; no self-loops, no duplicate edges.

## Prerequisites

- Tree fact: connected + `n - 1` edges ⇒ acyclic (and vice versa).
- Union-Find, or DFS/BFS connectivity with parent tracking.

## Approach 1: Union-Find

### Intuition

Process each edge. If its endpoints already share a root, this edge closes a cycle — not a tree. Otherwise union them. At the end, a tree has exactly one component.

### Algorithm

1. If `len(edges) != n - 1`, return `False` immediately.
2. DSU over `n`. For each `(u, v)`: if `find(u) == find(v)`, return `False`; else `union(u, v)`.
3. Return `True` (the edge count already guarantees connectivity).

```python
def validTree(n: int, edges: list[list[int]]) -> bool:
    if len(edges) != n - 1:
        return False
    parent = list(range(n))

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    for u, v in edges:
        ru, rv = find(u), find(v)
        if ru == rv:
            return False
        parent[ru] = rv
    return True
```

### Complexity

- **Time:** $O(n \cdot \alpha(n))$.
- **Space:** $O(n)$.

## Approach 2: DFS connectivity + cycle check

### Intuition

Build an adjacency list. DFS from node 0, passing the parent to ignore the edge you came in on. If you reach an already-visited node that is not the parent, there is a cycle. Afterwards, check every node was visited.

### Algorithm

1. Build `adj`. `visited = set()`.
2. `dfs(node, parent)`: if `node in visited`, return `False`. Add it; for each neighbour `nei != parent`, if `not dfs(nei, node)`, return `False`. Return `True`.
3. Return `dfs(0, -1) and len(visited) == n`.

```python
def validTree(n: int, edges: list[list[int]]) -> bool:
    if len(edges) != n - 1:
        return False
    adj = [[] for _ in range(n)]
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)

    visited = set()

    def dfs(node, parent):
        if node in visited:
            return False
        visited.add(node)
        for nei in adj[node]:
            if nei == parent:
                continue
            if not dfs(nei, node):
                return False
        return True

    return dfs(0, -1) and len(visited) == n
```

### Complexity

- **Time:** $O(n + e)$.
- **Space:** $O(n + e)$.

## Approach 3: BFS connectivity

### Intuition

Same criteria, iterative. BFS from 0; a neighbour already visited that is not the one you came from is a back edge (cycle). Count visited at the end.

### Algorithm

1. Build `adj`. `visited = {0}`, `q = deque([(0, -1)])`.
2. Pop `(node, parent)`; for each `nei`: if `nei == parent`, skip; if `nei in visited`, return `False`; else add and enqueue `(nei, node)`.
3. Return `len(visited) == n`.

```python
from collections import deque

def validTree(n: int, edges: list[list[int]]) -> bool:
    if len(edges) != n - 1:
        return False
    adj = [[] for _ in range(n)]
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)

    visited = {0}
    q = deque([(0, -1)])
    while q:
        node, parent = q.popleft()
        for nei in adj[node]:
            if nei == parent:
                continue
            if nei in visited:
                return False
            visited.add(nei)
            q.append((nei, node))
    return len(visited) == n
```

### Complexity

- **Time:** $O(n + e)$.
- **Space:** $O(n + e)$.

## Common Pitfalls

- **Skipping the edge-count check.** With `n - 1` edges, "connected" alone implies "tree"; without it you must check both connectivity and acyclicity.
- **Not passing the parent in DFS/BFS.** Every undirected edge looks like a 2-cycle; ignore the immediate parent so you don't flag it.
- **Ignoring the parent by identity when there are parallel edges.** The problem forbids duplicate edges, so `nei == parent` skipping exactly one occurrence is fine here.
- **Forgetting `n = 1, edges = []`.** One node, zero edges: a valid tree.

## The keystone

Union-Find shines the moment the question is "does this edge merge two things that were already together" — that is precisely a cycle. The same rejection test *is* the answer to [Redundant Connection](/citadel/dsa/redundant-connection).
