---
title: Bridges and Cut Vertices - Low-Link Values in O(n + m)
description: Finding the edges and vertices whose removal disconnects an undirected graph in one DFS, using the low value that measures how far a subtree can climb back up the tree, plus the strong-orientation application.
date: 2024-09-15
draft: false
slug: /cp/bridges-and-cut-vertices
tags:
  - Competitive Programming
  - Graphs
  - Connectivity
---

A **bridge** is an edge whose removal increases the number of connected components; a **cut vertex** (articulation point) is a vertex whose removal does. Both are found in a single [DFS](/citadel/cp/depth-first-search) using the `low` value — the earliest vertex a subtree can reach by going down tree edges and up at most one back edge. If a child's subtree cannot reach past its parent, the connecting edge is a bridge.

## The problem

Given a connected undirected graph, list all bridges and all cut vertices. Follow-ups: the **2-edge-connected components** (maximal subgraphs with no bridge), the **biconnected components** (no cut vertex), and the **block-cut tree**.

Example: a path $1 - 2 - 3 - 4$ has bridges $\{1,2\}, \{2,3\}, \{3,4\}$ and cut vertices $2, 3$. Add edge $\{1, 4\}$ and there are no bridges and no cut vertices — the cycle is 2-connected.

## The idea

Run DFS, computing `tin[v]` (discovery time) and

$$\text{low}[v] = \min\Big(\text{tin}[v],\ \min_{(v, w)\ \text{back edge}} \text{tin}[w],\ \min_{(v, c)\ \text{tree child}} \text{low}[c]\Big).$$

Then, for a tree edge $(u, v)$ with $u$ the parent:

- **Bridge:** $\text{low}[v] > \text{tin}[u]$ — $v$'s subtree has *no* back edge to $u$ or above, so removing $(u, v)$ cuts it off.
- **Cut vertex ($u$ not the root):** $\text{low}[v] \ge \text{tin}[u]$ for **some** child $v$ — that subtree cannot bypass $u$.
- **Cut vertex (root):** the root is a cut vertex iff it has **two or more** tree children.

## How it works

Graph $1 - 2 - 3$, $2 - 4 - 5$, $5 - 2$ (so $2, 4, 5$ form a triangle). DFS from $1$: `tin = [_,1,2,3,5,4]` (1-indexed sketch). The child $3$ of $2$ has `low[3] = tin[3] > tin[2]` ⇒ edge $\{2,3\}$ is a bridge, and $2$ is a cut vertex (it separates $3$ and $1$ from the triangle... actually $1$ too). The triangle $2$–$4$–$5$ has `low` values reaching back to $2$, so none of its edges are bridges.

## Algorithm

```python
import sys

def bridges_and_cutpoints(n, adj):
    sys.setrecursionlimit(1 << 25)
    tin = [-1] * n
    low = [0] * n
    timer = [0]
    bridges = []
    cutpoints = set()

    def dfs(u, parent_edge):
        tin[u] = low[u] = timer[0]
        timer[0] += 1
        children = 0
        for (v, eid) in adj[u]:
            if eid == parent_edge:
                continue
            if tin[v] != -1:                       # back edge
                low[u] = min(low[u], tin[v])
            else:
                children += 1
                dfs(v, eid)
                low[u] = min(low[u], low[v])
                if low[v] > tin[u]:
                    bridges.append(eid)
                if low[v] >= tin[u] and parent_edge != -1:
                    cutpoints.add(u)
        if parent_edge == -1 and children >= 2:
            cutpoints.add(u)

    for s in range(n):
        if tin[s] == -1:
            dfs(s, -1)
    return bridges, cutpoints
```

Store edges with an id (`adj[u]` holds `(neighbour, edge_id)`) so the parent edge is skipped exactly once — this handles **multi-edges** correctly (a doubled edge is never a bridge).

## Extensions

- **2-edge-connected components:** remove the bridges; each remaining connected piece is one component. Contracting them gives the **bridge tree**, a tree whose edges are the bridges.
- **Biconnected components:** maintain a stack of edges during DFS; when `low[v] >= tin[u]`, pop edges down to $(u, v)$ — that set is one biconnected component. The block-cut tree connects components to their shared cut vertices.
- **Strong orientation (Robbins' theorem):** a connected undirected graph has an orientation making it strongly connected **iff it has no bridge**. The orientation: direct tree edges away from the root, back edges toward the root.
- **[Online bridge finding](/citadel/cp/finding-bridges-online):** support edge insertions with a DSU-of-2-edge-connected-components.

## Common pitfalls

- **`low` from `tin`, not `low`, for back edges.** `low[u] = min(low[u], tin[v])`. Using `low[v]` overshoots and can miss bridges.
- **Parent edge by id, not by vertex.** Skipping "the neighbour equal to parent" fails on multi-edges; skip the specific `edge_id` you came in on, once.
- **Cut-vertex condition is `>=`, bridge is `>`.** A subtree that reaches *exactly* its parent ($\text{low}[v] = \text{tin}[u]$) does not make the edge a bridge but does make $u$ a cut vertex.
- **Root special case.** The root is a cut vertex only with $\ge 2$ DFS children — the `>=` test would wrongly flag it with one child.
- **Recursion depth.** Iterative DFS for $n \gtrsim 10^5$ in Python.

## The keystone

One DFS computes `low[v]` = the earliest vertex $v$'s subtree can reach via one back edge. A tree edge $(u, v)$ is a bridge when `low[v] > tin[u]` (the subtree is sealed off), and $u$ is a cut vertex when `low[v] >= tin[u]` for some child (or the root has two children). Removing the bridges gives the 2-edge-connected components.
