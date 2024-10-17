---
title: Topological Sorting - Kahn's Queue and DFS Post-Order
description: Ordering the vertices of a DAG so every edge points forward — the indegree-zero queue (Kahn) that also detects cycles and yields the lexicographically smallest order, and the DFS finish-time method.
date: 2024-10-17
draft: false
slug: /cp/topological-sort
tags:
  - Competitive Programming
  - Graphs
  - Graph Traversal
---

A topological order of a directed acyclic graph lists its vertices so that every edge $u \to v$ has $u$ before $v$. It exists **iff the graph has no cycle**, and there are two standard ways to produce it: Kahn's algorithm (repeatedly remove a vertex with no incoming edges) and DFS (reverse of the finish order). The [dsa problems](/citadel/dsa/course-schedule) use it for prerequisite chains; here is the clean reference.

## The problem

Given a directed graph, output a topological order, or report that it has a cycle. Variants: the **lexicographically smallest** topological order, **counting** distinct topological orders (that one is #P-hard in general — only feasible for small $n$ via bitmask DP), and **longest path in a DAG** (DP along the order).

Example: edges $A\to C,\ B\to C,\ C\to D$ → valid orders include $A, B, C, D$ and $B, A, C, D$.

## Approach 1: Kahn's algorithm (BFS on indegrees)

### Intuition

A vertex with indegree $0$ has no prerequisites — output it, remove it (decrementing its neighbours' indegrees), repeat. If you output fewer than $n$ vertices, the rest are in a cycle.

### Algorithm

```python
from collections import deque
import heapq

def kahn(n, adj):
    indeg = [0] * n
    for u in range(n):
        for v in adj[u]:
            indeg[v] += 1
    q = deque(v for v in range(n) if indeg[v] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in adj[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    if len(order) != n:
        return None                         # cycle
    return order

def kahn_lexicographic(n, adj):
    indeg = [0] * n
    for u in range(n):
        for v in adj[u]:
            indeg[v] += 1
    pq = [v for v in range(n) if indeg[v] == 0]
    heapq.heapify(pq)
    order = []
    while pq:
        u = heapq.heappop(pq)
        order.append(u)
        for v in adj[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                heapq.heappush(pq, v)
    return order if len(order) == n else None
```

Swap the `deque` for a min-heap to get the lexicographically smallest order ($O((V + E)\log V)$).

## Approach 2: DFS finish order

### Intuition

Run DFS; when a vertex finishes (all its descendants done), prepend it to the output. A vertex finishes *after* everything it can reach, so prepending puts it before them. Detect a cycle with a grey/black colouring: hitting a grey (on-stack) vertex is a back edge ⇒ cycle.

### Algorithm

```python
import sys

def toposort_dfs(n, adj):
    sys.setrecursionlimit(1 << 25)
    color = [0] * n                          # 0 white, 1 grey, 2 black
    order = []
    ok = [True]

    def dfs(u):
        color[u] = 1
        for v in adj[u]:
            if color[v] == 1:
                ok[0] = False                # back edge -> cycle
            elif color[v] == 0:
                dfs(v)
        color[u] = 2
        order.append(u)

    for s in range(n):
        if color[s] == 0:
            dfs(s)
    if not ok[0]:
        return None
    return order[::-1]                       # reverse of finish order
```

## Uses

- **Prerequisite / dependency resolution** ([Course Schedule](/citadel/dsa/course-schedule), build systems, spreadsheet recalculation).
- **DP on a DAG** — process vertices in topological order so all predecessors are done: longest path, number of paths, reachability, [shortest path with negative edges but no negative cycle](/citadel/cp/finding-negative-cycle) on a DAG in $O(V + E)$.
- **[SCC condensation](/citadel/cp/strongly-connected-components)** is a DAG — topologically sort it for a cyclic-graph DP.
- **Detecting a cycle** in a directed graph — either method reports it.
- **[Alien Dictionary](/citadel/dsa/alien-dictionary)** — infer a letter order from sorted words, then toposort.

## Common pitfalls

- **Assuming a total order.** A DAG usually has many topological orders; if the problem wants a *specific* one (lexicographically smallest, or stable), use the heap variant or a defined tie-break.
- **Cycle detection in DFS.** You must distinguish grey (on the current DFS path ⇒ back edge ⇒ cycle) from black (finished ⇒ fine). A plain visited boolean misses cycles.
- **Kahn's cycle test.** `len(order) < n` means a cycle; the leftover vertices all have indegree $> 0$.
- **Multi-edges.** Two copies of $u \to v$ add $2$ to `indeg[v]`; decrement once per edge instance, consistently.
- **Recursion depth** for the DFS version on long chains — iterate or raise the limit.

## The keystone

Kahn's algorithm repeatedly emits an indegree-zero vertex and removes it; finishing with fewer than $n$ vertices means a cycle, and a min-heap instead of a queue gives the lexicographically smallest order. The DFS version reverses the finish order. Either way, processing a DAG in topological order is what makes DAG dynamic programming linear.
