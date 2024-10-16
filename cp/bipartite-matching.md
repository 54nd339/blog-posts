---
title: Maximum Bipartite Matching - Kuhn's Algorithm and Konig's Theorem
description: Finding the largest set of independent edges in a bipartite graph by repeatedly searching for augmenting paths, and the theorems that convert the result into a minimum vertex cover, maximum independent set, or minimum path cover.
date: 2024-10-16
draft: false
slug: /cp/bipartite-matching
tags:
  - Competitive Programming
  - Graphs
  - Matching
---

A matching is a set of edges with no shared endpoint. In a bipartite graph, the maximum matching is found by Kuhn's algorithm — repeatedly look for an **augmenting path** (an alternating unmatched/matched path between two free vertices) and flip it. König's theorem then turns the matching size into a minimum vertex cover, and via complementation a maximum independent set.

## The problem

Given a bipartite graph with parts $L$ and $R$, find a maximum matching. Derived quantities (all from the same matching):

- **minimum vertex cover** — smallest set of vertices touching every edge; $|{\text{cover}}| = |{\text{matching}}|$ (König).
- **maximum independent set** — $|V| - |{\text{matching}}|$.
- **minimum edge cover** — $|V| - |{\text{matching}}|$ (if no isolated vertices).
- **minimum path cover of a DAG** — $n - |{\text{matching}}|$ in the split bipartite graph.

## The idea (Kuhn)

Start with an empty matching. For each left vertex $u$ in turn, run a DFS trying to match it: for each neighbour $v$, if $v$ is free, match $u \leftrightarrow v$; otherwise, recursively try to re-match $v$'s current partner elsewhere, and if that succeeds, steal $v$ for $u$. A successful DFS is an **augmenting path** and increases the matching by one. Reset the "visited right vertices" set before each left vertex.

Each left vertex triggers one DFS over $O(E)$ edges, so $O(VE)$ total.

## Algorithm

```python
def kuhn(n_left, n_right, adj):
    # adj[u] = list of right-vertices adjacent to left-vertex u
    match_r = [-1] * n_right                      # match_r[v] = left vertex matched to v, or -1
    match_l = [-1] * n_left

    def try_augment(u, visited):
        for v in adj[u]:
            if not visited[v]:
                visited[v] = True
                if match_r[v] == -1 or try_augment(match_r[v], visited):
                    match_r[v] = u
                    match_l[u] = v
                    return True
        return False

    matching = 0
    for u in range(n_left):
        visited = [False] * n_right
        if try_augment(u, visited):
            matching += 1
    return matching, match_l, match_r
```

**Speedups:** a greedy pre-match (match every left vertex to any free neighbour first) often halves the work. **Hopcroft-Karp** finds shortest augmenting paths in BFS phases for $O(E\sqrt V)$ — use it when $V, E$ are large.

## König's theorem: matching → vertex cover

After a maximum matching:

1. Let $Z$ = the set of vertices reachable from **unmatched left vertices** by alternating paths (unmatched edge left→right, matched edge right→left).
2. The **minimum vertex cover** is $(L \setminus Z) \cup (R \cap Z)$.
3. The **maximum independent set** is the complement: $(L \cap Z) \cup (R \setminus Z)$.

```python
def min_vertex_cover(n_left, n_right, adj, match_l, match_r):
    visited_l = [False] * n_left
    visited_r = [False] * n_right

    def dfs(u):
        visited_l[u] = True
        for v in adj[u]:
            if not visited_r[v]:
                visited_r[v] = True
                if match_r[v] != -1 and not visited_l[match_r[v]]:
                    dfs(match_r[v])

    for u in range(n_left):
        if match_l[u] == -1:
            dfs(u)

    cover_left = [u for u in range(n_left) if not visited_l[u]]
    cover_right = [v for v in range(n_right) if visited_r[v]]
    return cover_left, cover_right
```

## Applications

- **Assign tasks to workers** (each edge = "worker can do task"); max matching = most tasks done.
- **Minimum path cover of a DAG:** split each vertex $v$ into $v_L, v_R$; edge $u \to v$ becomes $u_L - v_R$. Answer $= n - \text{max matching}$. Vertex-disjoint paths covering all vertices.
- **Maximum independent set / minimum vertex cover** in a bipartite graph (both NP-hard in general, poly here via König).
- **Latin-square completion, tournament scheduling, resource allocation** — many "pair these up" problems.
- **Hall's theorem check:** a perfect matching of $L$ exists iff every subset $S \subseteq L$ has $|N(S)| \ge |S|$ — the matching either achieves $|L|$ or exposes a violating $S$.

## Complexity

- **Kuhn:** $O(VE)$; with the greedy warm start, near-linear in practice.
- **Hopcroft-Karp:** $O(E\sqrt V)$.
- König reconstruction: one $O(E)$ DFS.

## Common pitfalls

- **Not resetting `visited` per left vertex.** `visited` tracks right vertices used *in the current augmenting search*; carry it across and you miss augmentations.
- **Directed vs undirected adjacency.** Store `adj[u]` only for left→right; `match_r` gives the reverse. Building both directions is wasteful and error-prone.
- **Recursion depth.** Long alternating paths for $V \sim 10^5$ overflow Python's stack — iterate, or use Hopcroft-Karp (BFS-based).
- **König's reachability set.** Alternating from *unmatched* left vertices only. Starting from all left vertices gives the wrong cover.
- **Non-bipartite graph.** Kuhn and König need bipartiteness. General-graph matching is Blossom ($O(V^3)$) — much harder.

## The keystone

Kuhn's algorithm augments the matching one edge at a time by finding an alternating unmatched–matched–unmatched path and flipping it — $O(VE)$. König's theorem then reads a minimum vertex cover (and, by complement, a maximum independent set) directly off which vertices are reachable by alternating paths from the unmatched left side.
