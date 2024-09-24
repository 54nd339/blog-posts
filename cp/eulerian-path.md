---
title: Eulerian Path - Existence Conditions and Hierholzer's Algorithm
description: When a graph has a trail using every edge exactly once — the degree-parity conditions for undirected and directed graphs — and Hierholzer's O(E) construction that splices sub-cycles together.
date: 2024-09-24
draft: false
slug: /cp/eulerian-path
tags:
  - Competitive Programming
  - Graphs
  - Graph Traversal
---

An **Eulerian path** uses every edge of a graph exactly once; an **Eulerian circuit** is one that returns to its start. Whether one exists is decided by counting odd-degree vertices (undirected) or in/out-degree imbalance (directed), and Hierholzer's algorithm builds it in $O(E)$ by growing a walk and splicing detours in wherever it gets stuck. The [dsa Reconstruct Itinerary problem](/citadel/dsa/reconstruct-itinerary) is a lexicographic special case.

## The problem

Given a connected graph (ignoring isolated vertices), decide whether an Eulerian path/circuit exists and, if so, output the edge sequence.

## Existence conditions

**Undirected graph** (all edges in one connected component):

- **Eulerian circuit** iff every vertex has **even** degree.
- **Eulerian path (not circuit)** iff **exactly two** vertices have odd degree — the path must start at one and end at the other.
- More than two odd-degree vertices ⇒ none.

**Directed graph** (weakly connected, all edges in one strongly-connected-enough component — precisely: every vertex with a nonzero degree is in one SCC that contains all edges):

- **Eulerian circuit** iff $\deg^+(v) = \deg^-(v)$ for every $v$.
- **Eulerian path** iff exactly one vertex has $\deg^+ - \deg^- = +1$ (the **start**), exactly one has $\deg^- - \deg^+ = +1$ (the **end**), and all others balanced.

## Hierholzer's algorithm

Start at a valid start vertex $s$ (for a path, the odd-degree / surplus-out one; for a circuit, anywhere with an edge). Follow unused edges, marking each used, until you get **stuck** — for a circuit that can only be back at $s$. If unused edges remain, some vertex $v$ on the current walk still has unused edges; start a new sub-walk from $v$, which must also return to $v$, and **splice** it into the main walk at $v$. Repeat until every edge is used.

Implemented with an explicit stack (post-order emission):

```python
def eulerian_path_undirected(n, edges):
    # edges: list of (u, v). Returns a vertex sequence, or None.
    adj = [[] for _ in range(n)]
    used = [False] * len(edges)
    deg = [0] * n
    for i, (u, v) in enumerate(edges):
        adj[u].append((v, i))
        adj[v].append((u, i))
        deg[u] += 1
        deg[v] += 1

    odd = [v for v in range(n) if deg[v] % 2 == 1]
    if len(odd) not in (0, 2):
        return None
    start = odd[0] if odd else next((v for v in range(n) if deg[v] > 0), 0)

    ptr = [0] * n
    stack = [start]
    path = []
    while stack:
        v = stack[-1]
        while ptr[v] < len(adj[v]) and used[adj[v][ptr[v]][1]]:
            ptr[v] += 1
        if ptr[v] == len(adj[v]):
            path.append(v)
            stack.pop()
        else:
            to, eid = adj[v][ptr[v]]
            used[eid] = True
            stack.append(to)

    if len(path) != len(edges) + 1:
        return None                          # graph not connected on its edges
    return path[::-1]

def eulerian_path_directed(n, edges):
    adj = [[] for _ in range(n)]
    outd = [0] * n
    ind = [0] * n
    for (u, v) in edges:
        adj[u].append(v)
        outd[u] += 1
        ind[v] += 1
    start = 0
    plus = minus = 0
    for v in range(n):
        d = outd[v] - ind[v]
        if d == 1:
            plus += 1; start = v
        elif d == -1:
            minus += 1
        elif d != 0:
            return None
    if not ((plus == 0 and minus == 0) or (plus == 1 and minus == 1)):
        return None
    if plus == 0:
        start = next((v for v in range(n) if outd[v] > 0), 0)

    ptr = [0] * n
    stack = [start]
    path = []
    while stack:
        v = stack[-1]
        if ptr[v] < len(adj[v]):
            stack.append(adj[v][ptr[v]])
            ptr[v] += 1
        else:
            path.append(v)
            stack.pop()
    if len(path) != len(edges) + 1:
        return None
    return path[::-1]
```

For the **lexicographically smallest** Eulerian path (as in Reconstruct Itinerary), sort each adjacency list and consume it as a stack/heap — the same post-order emission gives the smallest sequence when reversed.

## Complexity

- **Time:** $O(V + E)$ — each edge is used once; the pointer per vertex never rewinds.
- **Space:** $O(V + E)$.

## Related

- **Chinese Postman / Route Inspection:** if a graph has $2k > 0$ odd-degree vertices, the shortest closed walk covering every edge duplicates a minimum-weight $T$-join (a min-cost perfect matching on the odd vertices via shortest paths). Eulerian on the augmented graph.
- **De Bruijn sequences:** an Eulerian circuit of the de Bruijn graph $B(k, n)$ spells a cyclic sequence containing every length-$n$ string over a $k$-alphabet exactly once.
- **[BEST theorem](/citadel/cp/kirchhoff-theorem):** counts Eulerian circuits of a digraph via a Laplacian cofactor.

## Common pitfalls

- **Connectivity.** The degree conditions are necessary but not sufficient without connectivity of the edge set. Check `len(path) == E + 1` after Hierholzer, or verify connectivity up front (ignoring degree-0 vertices).
- **Recursive Hierholzer stack overflow.** For $E \sim 10^5$, use the explicit-stack post-order version above, not recursion.
- **Which endpoint to start from.** Undirected path: an odd-degree vertex. Directed path: the vertex with $\deg^+ - \deg^- = 1$. Starting elsewhere for a *path* (not circuit) fails.
- **Multi-edges and self-loops.** Handle by edge id (undirected) so each parallel edge is used once; a self-loop adds $2$ to an undirected degree, $1$ to each of in/out for directed.
- **Output order.** The stack emits vertices in reverse; reverse `path` at the end.

## The keystone

An Eulerian circuit exists iff all degrees are even (undirected) / balanced (directed); an Eulerian path iff exactly two vertices are odd / imbalanced by one. Hierholzer walks unused edges until stuck, then splices in a detour from any vertex that still has edges — $O(E)$, emitted as a reversed post-order from an explicit stack.
