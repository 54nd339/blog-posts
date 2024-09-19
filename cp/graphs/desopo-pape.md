---
title: D'Esopo-Pape Algorithm - Fast in Practice, Exponential in the Worst Case
description: A SPFA-relative single-source shortest-path algorithm using a deque with a "seen before" rule, why it is often the fastest on real graphs, and the crafted inputs that blow it up to exponential time.
date: 2024-09-19
draft: false
slug: /cp/desopo-pape
tags:
  - Competitive Programming
  - Graphs
  - Shortest Paths
---

The D'Esopo-Pape algorithm computes single-source shortest paths (with possibly negative edges, no negative cycle) using a deque and one clever rule: a vertex being re-inserted goes to the **front**, a vertex seen for the first time goes to the **back**. On typical graphs it beats Bellman-Ford and often Dijkstra — but there are adversarial inputs that force **exponential** running time, so it is a "know the trap" algorithm.

## The problem

Single-source shortest paths in a directed graph with real edge weights and no negative cycle. Same setting as [Bellman-Ford](/citadel/algorithms/PathFinding).

## The idea

Maintain a deque and a per-vertex state:

- `state = 0` — never been in the deque,
- `state = 1` — currently in the deque,
- `state = 2` — was in the deque and has been popped at least once.

Pop from the front. Relax each outgoing edge; if `dist[v]` improves:

- if `state[v] == 0` (first time), push $v$ to the **back**,
- if `state[v] == 2` (returning), push $v$ to the **front** — it needs urgent re-processing because its earlier "final" value was wrong and it may have already propagated it,
- if `state[v] == 1` (already queued), leave it.

The front-insertion for returning vertices is what makes it fast in practice (it fixes mistakes early) and also what makes the worst case exponential (a returning vertex can cascade).

## Algorithm

```python
from collections import deque

def desopo_pape(n, adj, source):
    INF = float("inf")
    dist = [INF] * n
    dist[source] = 0
    state = [0] * n                          # 0 unseen, 1 in deque, 2 popped
    dq = deque([source])
    state[source] = 1
    while dq:
        u = dq.popleft()
        state[u] = 2
        for v, w in adj[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                if state[v] == 0:
                    dq.append(v)
                    state[v] = 1
                elif state[v] == 2:
                    dq.appendleft(v)
                    state[v] = 1
    return dist
```

## Complexity

- **Practical:** near-linear on random and most structured graphs — frequently the fastest option in a contest for negative-edge shortest paths.
- **Worst case:** **exponential** in $n$. There are known small constructions (a few dozen vertices) where the number of relaxations is $\Omega(2^{n/2})$.
- **Space:** $O(V)$.

## When to use it — and when not

- **Use it:** negative edges present, graph looks "normal" (random, geometric, real-world), and you want something faster than Bellman-Ford's $O(VE)$ without proving a bound.
- **Avoid it:** adversarial problem-setter, or any setting where a wrong-answer/TLE on a crafted test is unacceptable. Then use:
  - **Bellman-Ford** ($O(VE)$, guaranteed) if the graph is small,
  - **Johnson's algorithm** (reweight with Bellman-Ford potentials, then Dijkstra per source) for all-pairs with negative edges,
  - **Dijkstra** if you can rule out negative edges.
- **SPFA** (the queue-based Bellman-Ford, `state ∈ {in, out}` only, always push to the back) has the same "great in practice, killable" reputation; D'Esopo-Pape is its deque cousin.

## Common pitfalls

- **Trusting it on a hard problem.** The exponential case is real and small enough to appear as a deliberate test. If correctness under adversarial input matters, do not use it.
- **The three-state logic.** `0 → back`, `2 → front`, `1 → nothing`. Collapsing to two states (like SPFA) changes the algorithm and its constant.
- **Negative cycles.** Like Bellman-Ford's naive form, plain D'Esopo-Pape does not detect them — it loops forever. Add a relaxation counter or a Bellman-Ford pre-check if cycles are possible.
- **Using it where 0-1 BFS or Dijkstra applies.** Non-negative weights → Dijkstra; weights in $\{0,1\}$ → [0-1 BFS](/citadel/cp/zero-one-bfs). Both have clean bounds.

## The keystone

D'Esopo-Pape is a deque-based label-correcting shortest-path algorithm: first-seen vertices to the back, returning vertices to the front. That front-insertion makes it very fast on ordinary graphs and exponential on crafted ones — so reach for it only when the input is benign, and fall back to Bellman-Ford or Johnson when a worst-case guarantee is required.
