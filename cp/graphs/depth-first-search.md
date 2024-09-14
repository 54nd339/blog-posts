---
title: Depth-First Search - The DFS Tree, Edge Types, and Timestamps
description: What DFS builds beyond a visited set — a spanning tree that classifies every other edge, entry and exit times that answer ancestor queries in O(1), and the low-link value the connectivity algorithms run on.
date: 2024-09-14
draft: false
slug: /cp/depth-first-search
tags:
  - Competitive Programming
  - Graphs
  - Graph Traversal
---

DFS is not just "visit everything". The order it visits in imposes a **tree** on the graph, and relative to that tree every remaining edge falls into one of a few types. Recording an entry and exit time per vertex makes "is $u$ an ancestor of $v$" an $O(1)$ check, and the derived `low` value is what [bridges](/citadel/cp/bridges-and-cut-vertices), [articulation points](/citadel/cp/bridges-and-cut-vertices), and [SCCs](/citadel/cp/strongly-connected-components) are computed from.

## The DFS tree

Run DFS from a root. The edges you traverse to reach a new vertex form the **DFS tree** (a forest if the graph is disconnected). Every other edge $(u, v)$ is classified by the timestamps:

- **Tree edge** — used to discover $v$.
- **Back edge** — $v$ is an ancestor of $u$ in the tree (`tin[v] < tin[u]` and $v$ still on the stack). Back edges are exactly the edges that create cycles.
- **Forward edge** (directed only) — $v$ is a descendant of $u$ discovered earlier via another path.
- **Cross edge** (directed only) — neither is an ancestor of the other; $v$'s subtree was fully processed before $u$ was reached.

For an **undirected** graph there are only tree and back edges (each undirected edge is one or the other). This is why "does an undirected graph have a cycle" is simply "does DFS find a back edge".

## Entry / exit times

Assign `tin[v]` when DFS enters $v$ and `tout[v]` when it leaves. Then:

$$u \text{ is an ancestor of } v \iff \text{tin}[u] \le \text{tin}[v] \text{ and } \text{tout}[v] \le \text{tout}[u].$$

The interval $[\text{tin}[v], \text{tout}[v]]$ of a vertex contains exactly the intervals of its subtree — an **Euler-tour** flattening that turns "sum/update over a subtree" into a range operation on an array indexed by `tin` (feed it to a [Fenwick tree](/citadel/cp/fenwick-tree)).

## The low-link value

$$\text{low}[v] = \min\Big(\text{tin}[v],\ \min_{(v,w)\text{ back edge}} \text{tin}[w],\ \min_{(v,c)\text{ tree edge}} \text{low}[c]\Big).$$

`low[v]` is the earliest vertex (by `tin`) reachable from $v$'s subtree using tree edges downward and **at most one** back edge. It measures how far "up" the subtree can escape — the core of the connectivity algorithms.

## Algorithm

```python
import sys

def dfs_forest(n, adj):
    sys.setrecursionlimit(1 << 25)
    tin = [0] * n
    tout = [0] * n
    low = [0] * n
    parent = [-1] * n
    timer = [0]
    edge_type = {}                                  # (u, v) -> 'tree' | 'back' | 'forward' | 'cross'
    state = [0] * n                                 # 0 = unseen, 1 = on stack, 2 = done

    def dfs(u):
        state[u] = 1
        timer[0] += 1
        tin[u] = low[u] = timer[0]
        for v in adj[u]:
            if state[v] == 0:
                parent[v] = u
                edge_type[(u, v)] = 'tree'
                dfs(v)
                low[u] = min(low[u], low[v])
            elif state[v] == 1:
                edge_type.setdefault((u, v), 'back')
                low[u] = min(low[u], tin[v])
            else:  # state[v] == 2
                edge_type.setdefault((u, v),
                                     'forward' if tin[u] < tin[v] else 'cross')
        state[u] = 2
        timer[0] += 1
        tout[u] = timer[0]

    for s in range(n):
        if state[s] == 0:
            dfs(s)
    return tin, tout, low, parent, edge_type

def is_ancestor(tin, tout, u, v):
    return tin[u] <= tin[v] and tout[v] <= tout[u]
```

For large graphs, convert `dfs` to an explicit stack to avoid Python's recursion limit.

## What it powers

- **Cycle detection** — a back edge exists (undirected: any; directed: a back edge to an on-stack vertex).
- **[Topological sort](/citadel/cp/topological-sort)** — reverse of DFS finish order (`tout` descending).
- **[Bridges / articulation points](/citadel/cp/bridges-and-cut-vertices)** — compare `low[child]` to `tin[u]`.
- **[SCCs](/citadel/cp/strongly-connected-components)** — Tarjan uses `low` on the directed DFS; Kosaraju uses two DFS passes.
- **Subtree aggregates / path-to-root updates** — Euler tour on `tin`/`tout`.
- **Finding a cycle explicitly** — walk `parent` pointers from the back edge's endpoint.

## Common pitfalls

- **Recursion depth.** A path graph of $10^5$ vertices blows Python's default limit. Raise it, or iterate with an explicit stack.
- **`low` uses `tin` of the back-edge target, not `low`.** `low[u] = min(low[u], tin[v])` for a back edge; using `low[v]` there is a classic bug that breaks bridge detection in some cases.
- **Parent edge in undirected graphs.** When iterating neighbours, skip the edge back to `parent` **once** (multi-edges: skip only one copy), or you will treat it as a back edge.
- **Directed vs undirected edge types.** Undirected graphs have no forward/cross edges; do not add that logic there.
- **`tin`/`tout` scale.** Using one counter incremented on both enter and exit gives disjoint, nested intervals; incrementing only on enter also works but then use `tin[v] in [tin[u], tout[u]]` with `tout` = max `tin` in subtree.

## The keystone

DFS builds a tree in which every non-tree edge is a back edge (undirected) or back/forward/cross (directed), and `tin`/`tout` intervals encode ancestry in $O(1)$. The `low` value — earliest vertex reachable from a subtree via one back edge — is the single quantity the bridge, articulation-point, and SCC algorithms all compute.
