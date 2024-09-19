---
title: 0-1 BFS - Deque Shortest Paths With Zero and One Weights
description: When every edge weighs 0 or 1, a double-ended queue replaces Dijkstra's heap — push 0-edges to the front and 1-edges to the back — for O(V + E) single-source shortest paths.
date: 2024-09-19
draft: false
slug: /cp/zero-one-bfs
tags:
  - Competitive Programming
  - Graphs
  - Shortest Paths
---

Dijkstra on a graph with edge weights in $\{0, 1\}$ is overkill: the heap only ever holds two distinct distance values at a time (the current $d$ and $d + 1$). Replace it with a **deque** — relax a $0$-weight edge by pushing to the **front**, a $1$-weight edge by pushing to the **back** — and you get $O(V + E)$ instead of $O(E \log V)$.

## The problem

Single-source shortest paths where every edge weight is $0$ or $1$. This comes up more than it seems:

- grid where some moves are free and others cost $1$ (breaking a wall, changing direction, using a portal),
- "minimum number of edges to delete/flip so a path exists",
- state graphs where a transition is either free or costs one unit of a budget,
- $k$-weight generalisation: **0-$k$ BFS** with $k+1$ buckets (or Dial's algorithm).

Example: a maze where moving to an empty cell is free and moving through a wall costs $1$ — the answer is the fewest walls a path must break.

## The idea

BFS works because the queue stays sorted by distance when all weights are $1$. With weights $\{0, 1\}$, when you pop a vertex $u$ at distance $d$ and relax:

- a $0$-edge to $v$ gives $\text{dist}[v] = d$ — $v$ belongs at the **front** (same layer, process before the $d+1$ vertices already queued),
- a $1$-edge to $v$ gives $\text{dist}[v] = d + 1$ — $v$ belongs at the **back**.

The deque therefore always holds vertices of at most two consecutive distances, front $\le$ back — exactly the invariant plain BFS relies on. Each vertex can be pushed a few times; guard with a check that the popped distance is still current.

## Algorithm

```python
from collections import deque

def zero_one_bfs(n, adj, source):
    # adj[u] = list of (v, w) with w in {0, 1}
    INF = float("inf")
    dist = [INF] * n
    dist[source] = 0
    dq = deque([source])
    while dq:
        u = dq.popleft()
        for v, w in adj[u]:
            nd = dist[u] + w
            if nd < dist[v]:
                dist[v] = nd
                if w == 0:
                    dq.appendleft(v)
                else:
                    dq.append(v)
    return dist
```

A common variant pushes `(d, u)` pairs and skips stale pops (`if d > dist[u]: continue`) instead of relying on the relax check — either is fine.

## The 0-k generalisation (Dial's algorithm)

If weights are small integers in $[0, k]$, keep $k \cdot V + 1$ buckets indexed by distance (or a ring of $k + 1$ buckets). Process buckets in increasing order; relaxing an edge of weight $w$ drops $v$ into bucket $d + w$. Total $O(E + kV)$ — better than Dijkstra's $\log$ when $k$ is a small constant.

## Complexity

- **Time:** $O(V + E)$ — each edge is examined $O(1)$ times, each vertex enters the deque $O(1)$ times amortised.
- **Space:** $O(V)$.

## Common pitfalls

- **`appendleft` vs `append` swapped.** $0$-edges to the front, $1$-edges to the back. Reversed, the distances come out wrong (it degenerates to a broken DFS-ish order).
- **Not guarding stale entries.** A vertex can be pushed twice before its first pop; check `nd < dist[v]` on relax (as above) or skip on pop.
- **Weights not actually 0/1.** If any edge has weight $\ge 2$, use Dijkstra or Dial's. 0-1 BFS silently gives wrong answers otherwise.
- **Grid direction changes.** "Cost to turn" problems often need the *state* to include the current direction, so the graph has $4V$ nodes; the $0$-edge is "keep going", the $1$-edge is "turn".
- **`dist[source]` vs re-relaxing.** Initialise `dist[source] = 0`; the loop handles the rest.

## The keystone

With edge weights in $\{0, 1\}$, the frontier only ever spans two distance values, so a deque — $0$-edges to the front, $1$-edges to the back — keeps it sorted and gives $O(V + E)$ shortest paths. For weights up to a small $k$, bucket by distance (Dial's) instead.
