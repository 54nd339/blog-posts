---
title: Maximum Flow - Edmonds-Karp, Dinic, and Push-Relabel
description: Pushing the most flow from source to sink through capacitated edges — the residual-graph idea, BFS-augmenting Edmonds-Karp, the blocking-flow speedup of Dinic, and where push-relabel and the min-cut duality fit.
date: 2024-10-12
draft: false
slug: /cp/maximum-flow
tags:
  - Competitive Programming
  - Graphs
  - Network Flow
---

Maximum flow: given a directed graph with edge capacities, a source $s$, and a sink $t$, push as much flow as possible from $s$ to $t$ without exceeding any capacity or leaving flow stuck at an intermediate vertex. The **max-flow min-cut theorem** ties the answer to the cheapest way to sever $s$ from $t$. Dinic's algorithm is the contest default at $O(V^2 E)$ (much faster in practice, $O(E\sqrt V)$ on unit-capacity graphs).

## The problem

Find the maximum $s \to t$ flow value (and, usually, the flow on each edge or the min cut). Applications: bipartite matching, vertex-disjoint / edge-disjoint paths, project selection, scheduling with capacities, image segmentation, and anything modelled as "route units through a network".

## The core idea: residual graph and augmenting paths

For each directed edge $u \to v$ with capacity $c$ and current flow $f$, the **residual capacity** is $c - f$ forward and $f$ backward (a reverse edge $v \to u$ that lets you *cancel* previously routed flow). An **augmenting path** is any $s \to t$ path in the residual graph with all residual capacities positive; pushing $\min$ residual along it increases the flow. Repeat until no augmenting path exists — then the reachable set from $s$ in the residual graph is one side of a minimum cut, and its outgoing capacity equals the flow (max-flow min-cut).

## Approach 1: Edmonds-Karp (BFS augmenting paths)

### Intuition

Ford-Fulkerson picks *any* augmenting path; choosing the **shortest** (fewest edges, via BFS) bounds the number of augmentations at $O(VE)$, giving $O(VE^2)$ total.

```python
from collections import deque

class MaxFlow:
    def __init__(self, n):
        self.n = n
        self.graph = [[] for _ in range(n)]      # each entry: [to, cap, index of reverse edge]

    def add_edge(self, u, v, cap):
        self.graph[u].append([v, cap, len(self.graph[v])])
        self.graph[v].append([u, 0, len(self.graph[u]) - 1])

    def edmonds_karp(self, s, t):
        flow = 0
        while True:
            parent = [(-1, -1)] * self.n
            parent[s] = (s, -1)
            q = deque([s])
            while q and parent[t][0] == -1:
                u = q.popleft()
                for i, (v, cap, _) in enumerate(self.graph[u]):
                    if cap > 0 and parent[v][0] == -1:
                        parent[v] = (u, i)
                        q.append(v)
            if parent[t][0] == -1:
                return flow
            # bottleneck
            aug = float("inf")
            v = t
            while v != s:
                u, i = parent[v]
                aug = min(aug, self.graph[u][i][1])
                v = u
            v = t
            while v != s:
                u, i = parent[v]
                self.graph[u][i][1] -= aug
                rev = self.graph[u][i][2]
                self.graph[v][rev][1] += aug
                v = u
            flow += aug
```

### Complexity

$O(VE^2)$. Fine for small graphs; superseded by Dinic.

## Approach 2: Dinic (blocking flows on the level graph)

### Intuition

One "phase": BFS from $s$ to assign each vertex a **level** (BFS distance in the residual graph). Then find a **blocking flow** — push flow only along edges that go from level $\ell$ to level $\ell + 1$, using a DFS with a per-vertex "current edge" pointer so dead ends are never revisited. Each phase increases the $s$–$t$ distance, so there are at most $V$ phases.

```python
class Dinic:
    def __init__(self, n):
        self.n = n
        self.graph = [[] for _ in range(n)]

    def add_edge(self, u, v, cap):
        self.graph[u].append([v, cap, len(self.graph[v])])
        self.graph[v].append([u, 0, len(self.graph[u]) - 1])

    def _bfs(self, s, t):
        self.level = [-1] * self.n
        self.level[s] = 0
        q = deque([s])
        while q:
            u = q.popleft()
            for v, cap, _ in self.graph[u]:
                if cap > 0 and self.level[v] == -1:
                    self.level[v] = self.level[u] + 1
                    q.append(v)
        return self.level[t] != -1

    def _dfs(self, u, t, pushed):
        if u == t or pushed == 0:
            return pushed
        while self.it[u] < len(self.graph[u]):
            e = self.graph[u][self.it[u]]
            v, cap, rev = e
            if cap > 0 and self.level[v] == self.level[u] + 1:
                d = self._dfs(v, t, min(pushed, cap))
                if d > 0:
                    e[1] -= d
                    self.graph[v][rev][1] += d
                    return d
            self.it[u] += 1
        return 0

    def max_flow(self, s, t):
        flow = 0
        while self._bfs(s, t):
            self.it = [0] * self.n
            while True:
                f = self._dfs(s, t, float("inf"))
                if f == 0:
                    break
                flow += f
        return flow

    def min_cut(self, s):
        # after max_flow: vertices with level != -1 in the last BFS are the s-side
        return [v for v in range(self.n) if self.level[v] != -1]
```

### Complexity

$O(V^2 E)$ in general; $O(E \sqrt E)$ on unit-capacity graphs; $O(E \sqrt V)$ for bipartite matching. In practice it is dramatically faster than the bound and is the standard choice.

## Approach 3: Push-relabel (and its variants)

Instead of augmenting paths, maintain a **preflow** (flow conservation relaxed to "inflow $\ge$ outflow") and vertex **heights**; repeatedly *push* excess from a higher vertex to a lower neighbour, or *relabel* (raise) a vertex with excess but no lower neighbour. The **highest-label** selection rule gives $O(V^2 \sqrt E)$; the FIFO rule $O(V^3)$; with gap heuristics it is often the fastest on dense graphs. More moving parts than Dinic; reach for it when Dinic is too slow on a dense instance.

## Modelling cheatsheet

- **Undirected edge of capacity $c$:** add both $u \to v$ and $v \to u$ with capacity $c$ (each with a $0$-capacity reverse).
- **Vertex capacity:** split $v$ into $v_{\text{in}} \to v_{\text{out}}$ with the capacity on that edge.
- **Multiple sources/sinks:** a super-source to each real source (capacity $\infty$ or the source's supply), similarly a super-sink.
- **Lower bounds on edges:** see [flows with demands](/citadel/cp/flows-with-demands).
- **Bipartite matching:** source → left (cap 1), left → right for each allowed pair (cap 1), right → sink (cap 1); max flow = max matching ([Kuhn](/citadel/cp/bipartite-matching) is simpler for pure matching).

## Common pitfalls

- **Reverse edge indexing.** Each `add_edge` pushes two entries; store the index of the partner so pushing flow updates both. Off-by-one here corrupts the residual graph.
- **Integer vs float capacities.** Ford-Fulkerson can fail to terminate on irrational capacities; with integers Edmonds-Karp/Dinic always terminate. Keep capacities integral.
- **`INF` capacity arithmetic.** Use a value larger than the total possible flow but small enough not to overflow when summed; in Python it is a non-issue, in C++ pick $\sim 10^{18}$.
- **Reading the min cut.** After max flow, do one more BFS in the residual graph; vertices reachable from $s$ are one side, and the cut edges are the saturated $s$-side → $t$-side edges.
- **Dinic without the current-edge pointer (`it`).** Re-scanning dead-end edges makes the blocking flow $O(VE)$ per phase instead of $O(VE)$ total — keep `self.it`.

## The keystone

Max flow is repeated augmentation in the residual graph until no $s$–$t$ path remains, at which point the flow equals the minimum cut. Edmonds-Karp augments along BFS shortest paths ($O(VE^2)$); Dinic groups augmentations into blocking flows on the BFS level graph ($O(V^2E)$, far faster in practice); push-relabel abandons paths for local excess pushes on dense graphs.
