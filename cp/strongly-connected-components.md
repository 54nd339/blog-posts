---
title: Strongly Connected Components and the Condensation Graph
description: Partitioning a directed graph into maximal mutually-reachable groups — Kosaraju's two DFS passes and Tarjan's one-pass low-link method — and why contracting each group yields a DAG you can DP on.
date: 2024-09-19
draft: false
slug: /cp/strongly-connected-components
tags:
  - Competitive Programming
  - Graphs
  - Connectivity
---

In a directed graph, two vertices are **strongly connected** if each can reach the other. The equivalence classes are the strongly connected components (SCCs). Contract each SCC to a single node and you get the **condensation** — a directed acyclic graph — which turns cyclic reachability problems into DAG dynamic programming. Both Kosaraju and Tarjan find all SCCs in $O(V + E)$.

## The problem

Partition the vertices of a directed graph into SCCs, and (usually) build the condensation DAG, noting which topological order the algorithm hands you the components in.

Example: edges $1\to2,\ 2\to3,\ 3\to1,\ 3\to4,\ 4\to5,\ 5\to4$. SCCs: $\{1,2,3\}$, $\{4,5\}$. Condensation: $\{1,2,3\} \to \{4,5\}$.

## Approach 1: Kosaraju (two passes)

### Intuition

1. DFS the graph, pushing each vertex onto a stack when it **finishes** (post-order).
2. Reverse every edge.
3. Pop vertices from the stack; each unvisited one starts a DFS in the reversed graph — the vertices it reaches form one SCC.

Why it works: the last vertex to finish in pass 1 is in a **source** SCC of the condensation; in the reversed graph, that SCC can only reach itself. Processing vertices in *decreasing* finish time therefore peels SCCs in topological order (a source SCC becomes component 0).

### Algorithm

```python
import sys

def kosaraju(n, adj):
    sys.setrecursionlimit(1 << 25)
    radj = [[] for _ in range(n)]
    for u in range(n):
        for v in adj[u]:
            radj[v].append(u)

    visited = [False] * n
    order = []

    def dfs1(u):
        visited[u] = True
        for v in adj[u]:
            if not visited[v]:
                dfs1(v)
        order.append(u)

    for s in range(n):
        if not visited[s]:
            dfs1(s)

    comp = [-1] * n
    c = 0

    def dfs2(u):
        comp[u] = c
        for v in radj[u]:
            if comp[v] == -1:
                dfs2(v)

    for u in reversed(order):
        if comp[u] == -1:
            dfs2(u)
            c += 1
    return comp, c                       # comp[v] in [0, c); components in TOPOLOGICAL order (sources first)
```

## Approach 2: Tarjan (one pass)

### Intuition

A single DFS with a stack of "vertices whose SCC is not yet closed" and the [`low` value](/citadel/cp/depth-first-search): `low[u]` = the smallest `tin` reachable from $u$ using tree/forward/back edges **and cross edges to still-on-stack vertices**. When DFS finishes a vertex $u$ with `low[u] == tin[u]`, $u$ is the **root** of its SCC — pop the stack down to $u$; those vertices are one component.

### Algorithm

```python
import sys

def tarjan(n, adj):
    sys.setrecursionlimit(1 << 25)
    tin = [-1] * n
    low = [0] * n
    on_stack = [False] * n
    stack = []
    timer = [0]
    comp = [-1] * n
    c = [0]

    def dfs(u):
        tin[u] = low[u] = timer[0]
        timer[0] += 1
        stack.append(u)
        on_stack[u] = True
        for v in adj[u]:
            if tin[v] == -1:
                dfs(v)
                low[u] = min(low[u], low[v])
            elif on_stack[v]:
                low[u] = min(low[u], tin[v])
        if low[u] == tin[u]:
            while True:
                w = stack.pop()
                on_stack[w] = False
                comp[w] = c[0]
                if w == u:
                    break
            c[0] += 1

    for s in range(n):
        if tin[s] == -1:
            dfs(s)
    return comp, c[0]                    # Tarjan numbers components in REVERSE topological order (sinks first)
```

Tarjan is one DFS and no reverse graph — usually preferred. Kosaraju is arguably easier to remember.

## The condensation and what it buys

Map each vertex to `comp[v]`; add a condensation edge $\text{comp}[u] \to \text{comp}[v]$ for every original edge $u \to v$ with $\text{comp}[u] \ne \text{comp}[v]$ (dedupe). The result is a DAG. Now:

- **Longest path / DP over reachability** becomes a DAG DP.
- **"Add fewest edges to make the whole graph strongly connected"** = $\max(\text{sources}, \text{sinks})$ of the condensation (if it has more than one component).
- **[2-SAT](/citadel/cp/two-sat)** — variable $x$ is forced iff $x$ and $\lnot x$ land in the same SCC; otherwise assign by component order.
- **Reachability queries** — precompute per-component reachable sets (bitsets) on the DAG.

## Complexity

- **Time:** $O(V + E)$ for both.
- **Space:** $O(V + E)$; Kosaraju also stores the reversed graph.

## Common pitfalls

- **Tarjan cross-edge rule.** Update `low[u]` from `tin[v]` **only if `v` is on the stack**. A finished vertex not on the stack belongs to a completed SCC and must not contribute.
- **`low` from `tin`, not `low`, on a back/cross edge** — same rule as in [bridges](/citadel/cp/bridges-and-cut-vertices).
- **Component order.** Tarjan numbers components in **reverse** topological order (a sink SCC is component 0); this Kosaraju implementation numbers them in topological order (a source is component 0). If your DAG DP needs a specific order, know which you have, or reverse.
- **Recursion depth.** $10^5$-vertex chains overflow Python's stack; convert to an explicit stack.
- **Self-loops and parallel edges.** Harmless for SCCs; just skip duplicates when building the condensation.

## The keystone

SCCs are the maximal mutually-reachable vertex sets; Kosaraju finds them with a DFS post-order then a DFS on the reversed graph, Tarjan with a single DFS that pops an SCC whenever `low[u] == tin[u]`. Contracting each SCC gives a DAG, converting cyclic problems (2-SAT, reachability DP, "make it strongly connected") into acyclic ones.
