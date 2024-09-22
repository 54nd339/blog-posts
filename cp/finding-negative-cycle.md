---
title: Finding a Negative Cycle - Bellman-Ford With Parent Pointers
description: Detecting a negative-weight cycle by running Bellman-Ford one extra round and, when a vertex still relaxes, walking parent pointers n steps into the cycle and extracting it.
date: 2024-09-22
draft: false
slug: /cp/finding-negative-cycle
tags:
  - Competitive Programming
  - Graphs
  - Shortest Paths
---

Bellman-Ford converges after $n - 1$ rounds of edge relaxation **iff** there is no negative cycle reachable from the source. Run one more round: if any edge still relaxes, a negative cycle exists — and following parent pointers backward from that edge, then $n$ more steps, lands you inside the cycle so you can print it.

## The problem

- **Detect** whether a directed weighted graph has a negative-weight cycle (anywhere, or reachable from a given source).
- **Report** one such cycle as a vertex sequence.

Example: edges $A \to B$ ($1$), $B \to C$ ($-3$), $C \to A$ ($1$) — cycle weight $-1$, negative. The routine should return $[A, B, C]$ (or a rotation).

## The idea

Bellman-Ford relaxes all $m$ edges $n - 1$ times; after that, shortest paths are final *unless* a negative cycle keeps making some distance smaller. So do an **$n$-th round**:

- If no edge relaxes → no negative cycle (reachable from the sources used).
- If edge $(u, v)$ relaxes → $v$ is "on or after" a negative cycle. Set `parent[v] = u` as usual, then take the vertex $x = v$ and follow `parent` **$n$ times**. Now $x$ is guaranteed to be *on* the cycle (any prefix leading into it has length $< n$). Walk `parent` from $x$ until you return to $x$, collecting vertices — that is the cycle, reversed.

To catch a negative cycle **anywhere** (not just reachable from one source), initialise every `dist[v] = 0` (equivalent to a virtual super-source with a $0$-edge to every vertex).

## Algorithm

```python
def find_negative_cycle(n, edges):
    # edges: list of (u, v, w)
    dist = [0] * n                            # virtual super-source -> detect anywhere
    parent = [-1] * n
    x = -1
    for i in range(n):
        x = -1
        for (u, v, w) in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                parent[v] = u
                x = v
    if x == -1:
        return None                          # no negative cycle

    for _ in range(n):                        # walk n steps to land on the cycle
        x = parent[x]

    cycle = []
    cur = x
    while True:
        cycle.append(cur)
        cur = parent[cur]
        if cur == x and len(cycle) > 1:
            break
    cycle.reverse()
    return cycle                              # a vertex sequence forming a negative cycle
```

The outer loop runs $n$ times (not $n - 1$); if the last iteration set `x`, a cycle was found.

## Variants

- **Reachable from a specific source $s$:** set `dist[s] = 0` and all others to $\infty$; a relaxation in round $n$ then means a negative cycle reachable from $s$.
- **SPFA-based detection:** the queue-based Bellman-Ford flags a negative cycle when any vertex is relaxed $\ge n$ times — faster on average, same worst case.
- **[Floyd-Warshall](/citadel/algorithms/PathFinding) detection:** after the algorithm, a negative `dist[i][i]` means $i$ is on a negative cycle; reconstruct via the `next`/`parent` matrix. $O(n^3)$, good when you already need all-pairs.
- **Minimum mean cycle** (Karp's algorithm): $\min_C \frac{\text{weight}(C)}{|C|}$ in $O(nm)$ — a different question ("most negative average edge"), used in cycle-cancelling [min-cost flow](/citadel/cp/min-cost-flow).

## Complexity

- **Time:** $O(nm)$ — the Bellman-Ford relaxation. Cycle extraction is $O(n)$.
- **Space:** $O(n + m)$.

## Common pitfalls

- **Only $n - 1$ rounds.** You need the $n$-th round to *detect*. Running $n$ rounds (as above) and checking whether the last one relaxed anything is the clean form.
- **Not walking $n$ steps before extracting.** The vertex that relaxed in round $n$ may be on a *path into* the cycle, not the cycle itself. Following `parent` $n$ times guarantees you are on the cycle.
- **Missing cycles unreachable from the source.** Initialise all `dist` to $0$ (virtual super-source) to detect a negative cycle anywhere.
- **Positive-weight-only graphs.** No negative edges ⇒ no negative cycle; skip the whole thing and use Dijkstra.
- **Cycle direction / rotation.** The extracted cycle is correct up to rotation and is emitted in forward edge order after the `reverse()`.

## The keystone

Bellman-Ford stops improving after $n - 1$ rounds unless a negative cycle exists; an $n$-th relaxing round proves one is present. Follow `parent` $n$ steps from the relaxed vertex to land on the cycle, then loop `parent` until you return — that vertex list is the negative cycle.
