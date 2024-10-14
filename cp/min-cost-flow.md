---
title: Minimum-Cost Flow - Successive Shortest Paths With Potentials
description: Sending a required amount of flow (or a max flow) at minimum total cost by repeatedly augmenting along the cheapest source-to-sink path, using Johnson potentials so Dijkstra works despite negative-cost residual edges.
date: 2024-10-14
draft: false
slug: /cp/min-cost-flow
tags:
  - Competitive Programming
  - Graphs
  - Network Flow
---

Minimum-cost flow adds a per-unit **cost** to each capacitated edge and asks for the cheapest way to route a given amount of flow (or the cheapest maximum flow). The workhorse is **successive shortest paths**: repeatedly push flow along the minimum-cost $s \to t$ path in the residual graph. Reduced costs (Johnson potentials) keep every shortest-path search a Dijkstra even though residual reverse edges have negative cost.

## The problem

Each edge has capacity $c_e$ and cost $a_e$ per unit of flow. Find the flow of a target value $F$ (or the maximum flow) minimising $\sum_e a_e f_e$. Assume no negative-cost cycle in the original graph.

Applications: [assignment problem](/citadel/cp/assignment-problem), transportation/logistics, min-cost bipartite matching, "buy/sell with capacities", scheduling with penalties, some DP-on-a-DAG problems recast as flow.

## The idea: successive shortest paths

Start with zero flow. Repeat:

1. Find the **minimum-cost** $s \to t$ path in the residual graph (edges with residual capacity $> 0$; a used edge's reverse has cost $-a_e$).
2. Push as much flow as the bottleneck allows (or exactly what remains of the target $F$).
3. Update residual capacities and add the path's cost $\times$ pushed amount to the total.

This yields a min-cost flow of *each* intermediate value — so you can stop at any target $F$, and the cost is convex in $F$.

**Why potentials.** Residual reverse edges carry negative cost, so a plain Dijkstra is invalid. Keep a potential $h[v]$ (initially the shortest-path distances from a Bellman-Ford, or $0$ if all costs are non-negative). Define the **reduced cost** of edge $u \to v$ as $a_e + h[u] - h[v] \ge 0$; Dijkstra on reduced costs is valid, and after each augmentation you update $h[v] \mathrel{+}= \text{dist}[v]$.

## Algorithm

```python
import heapq

class MinCostFlow:
    def __init__(self, n):
        self.n = n
        self.graph = [[] for _ in range(n)]     # entry: [to, cap, cost, rev_index]

    def add_edge(self, u, v, cap, cost):
        self.graph[u].append([v, cap, cost, len(self.graph[v])])
        self.graph[v].append([u, 0, -cost, len(self.graph[u]) - 1])

    def flow(self, s, t, maxf=float("inf")):
        n = self.n
        total_flow = 0
        total_cost = 0
        h = [0] * n                             # potentials; assumes no negative-cost edge initially
        while total_flow < maxf:
            dist = [float("inf")] * n
            dist[s] = 0
            pv = [-1] * n
            pe = [-1] * n
            pq = [(0, s)]
            while pq:
                d, u = heapq.heappop(pq)
                if d > dist[u]:
                    continue
                for i, (v, cap, cost, _) in enumerate(self.graph[u]):
                    if cap > 0 and dist[u] + cost + h[u] - h[v] < dist[v]:
                        dist[v] = dist[u] + cost + h[u] - h[v]
                        pv[v] = u
                        pe[v] = i
                        heapq.heappush(pq, (dist[v], v))
            if dist[t] == float("inf"):
                break                           # no more augmenting paths
            for v in range(n):
                if dist[v] < float("inf"):
                    h[v] += dist[v]
            # bottleneck along the path
            aug = maxf - total_flow
            v = t
            while v != s:
                aug = min(aug, self.graph[pv[v]][pe[v]][1])
                v = pv[v]
            v = t
            while v != s:
                e = self.graph[pv[v]][pe[v]]
                e[1] -= aug
                self.graph[v][e[3]][1] += aug
                v = pv[v]
            total_flow += aug
            total_cost += aug * h[t]            # h[t] is now the true shortest s->t cost
        return total_flow, total_cost
```

If the original graph *does* have negative-cost edges (but no negative cycle), initialise `h` with a Bellman-Ford from $s$ first.

## Variants

- **Min-cost max-flow:** call `flow(s, t)` with `maxf = inf` — it stops when $t$ is unreachable.
- **Min-cost flow of exactly $F$:** pass `maxf = F`; if it returns `total_flow < F`, that much flow is infeasible.
- **Cycle-cancelling:** an alternative — find any max flow, then repeatedly cancel negative-cost cycles in the residual graph ([Bellman-Ford / SPFA](/citadel/cp/finding-negative-cycle) or Karp min-mean-cycle). Simpler to state, usually slower.
- **SPFA instead of Dijkstra:** skip potentials, find the cheapest path with a queue-based Bellman-Ford each round. Easier to code, worse worst case.

## Complexity

- **Successive shortest paths with potentials:** $O(F \cdot E \log V)$ where $F$ is the flow value (number of augmentations $\le F$ for unit capacities, or $\le VE$ in general). For assignment-style problems $F = n$, giving $O(n E \log V)$.
- **Space:** $O(V + E)$.

## Common pitfalls

- **Negative residual costs without potentials.** Plain Dijkstra gives wrong distances. Use reduced costs $a_e + h[u] - h[v]$ and update $h$ each round.
- **Updating `h` for unreachable vertices.** Only add `dist[v]` where `dist[v] < inf`; adding `inf` corrupts the potential.
- **Cost accumulation.** After updating `h`, `h[t]` equals the real shortest $s \to t$ cost for this augmentation — multiply by `aug`. Accumulating raw `dist[t]` (a reduced cost) is wrong.
- **Negative-cost cycle in the input.** Successive shortest paths assumes none. If present, cancel them first or the answer is unbounded/ill-defined.
- **Reverse edge bookkeeping.** Same as [max flow](/citadel/cp/maximum-flow): store the partner index; the reverse edge's cost is $-a_e$.

## The keystone

Min-cost flow augments along the cheapest residual $s \to t$ path each round; Johnson potentials turn each such search into a Dijkstra despite negative reverse-edge costs, and updating the potentials by the round's distances keeps reduced costs non-negative. The cost is convex in the flow value, so you can stop at any target amount.
