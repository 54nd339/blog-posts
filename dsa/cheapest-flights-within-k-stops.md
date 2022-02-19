---
title: Cheapest Flights Within K Stops - Bellman-Ford With a Hop Budget
description: The cheapest route from source to destination using at most k intermediate stops — relax all edges k plus one times from a frozen snapshot, or run a layered BFS by hop count.
date: 2022-02-19
draft: false
slug: /dsa/cheapest-flights-within-k-stops
tags:
  - Competitive Programming
  - Graphs
  - Shortest Path
---

Plain Dijkstra minimises cost but ignores the stop limit. Bellman-Ford fits perfectly: after `i` relaxation rounds it knows the cheapest cost using at most `i` edges, so `k + 1` rounds answers "at most `k` stops". The catch is relaxing from a *snapshot* of the previous round so one round cannot chain several edges.

## Description

`flights[i] = [from, to, price]`. Given `n` cities, a `src`, a `dst`, and `k`, return the cheapest price from `src` to `dst` with at most `k` stops in between, or `-1` if there is no such route.

**Example**

```
Input:  n = 4, flights = [[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]],
        src = 0, dst = 3, k = 1
Output: 700

Input:  n = 3, flights = [[0,1,100],[1,2,100],[0,2,500]], src = 0, dst = 2, k = 0
Output: 500
```

**Constraints**

- $1 \le n \le 100$; $0 \le \text{flights.length} \le n(n-1)/2$
- $0 \le \text{src}, \text{dst}, k < n$; $1 \le \text{price} \le 10^4$

## Prerequisites

- Bellman-Ford edge relaxation and its "at most `i` edges after round `i`" invariant.
- Why the previous round must be snapshotted.

## Approach 1: Bellman-Ford (k + 1 rounds)

### Intuition

`dist[v]` = cheapest cost to `v` so far. Each round, relax every edge using `prev`, a frozen copy of `dist` from before the round. After `k + 1` rounds, `dist[dst]` is the cheapest using at most `k + 1` edges, i.e. at most `k` stops.

### Algorithm

1. `dist = [inf] * n`, `dist[src] = 0`.
2. Repeat `k + 1` times: `prev = dist[:]`; for each `(u, v, w)`, if `prev[u] + w < dist[v]`, set `dist[v] = prev[u] + w`.
3. Return `dist[dst]` if finite, else `-1`.

```python
def findCheapestPrice(n: int, flights: list[list[int]], src: int, dst: int, k: int) -> int:
    dist = [float("inf")] * n
    dist[src] = 0

    for _ in range(k + 1):
        prev = dist[:]
        for u, v, w in flights:
            if prev[u] + w < dist[v]:
                dist[v] = prev[u] + w

    return dist[dst] if dist[dst] != float("inf") else -1
```

### Complexity

- **Time:** $O(k \cdot E)$.
- **Space:** $O(n)$.

## Approach 2: Layered BFS by stop count

### Intuition

BFS where each layer adds one flight. Carry `(city, cost)`; process exactly `k + 1` layers. Keep `best[city]` and only enqueue a `(next_city, new_cost)` when `new_cost` beats the best seen for `next_city`, to prune.

### Algorithm

1. `adj[u] = [(v, w)]`. `q = deque([(src, 0)])`. `best = [inf] * n`, `best[src] = 0`.
2. Repeat up to `k + 1` times while `q` is non-empty: for each of the `len(q)` items, pop `(u, cost)`; for each `(v, w)`, if `cost + w < best[v]`, update `best[v]` and enqueue `(v, cost + w)`.
3. Return `best[dst]` if finite else `-1`.

```python
from collections import deque, defaultdict

def findCheapestPrice(n: int, flights: list[list[int]], src: int, dst: int, k: int) -> int:
    adj = defaultdict(list)
    for u, v, w in flights:
        adj[u].append((v, w))

    best = [float("inf")] * n
    best[src] = 0
    q = deque([(src, 0)])
    stops = 0

    while q and stops <= k:
        for _ in range(len(q)):
            u, cost = q.popleft()
            for v, w in adj[u]:
                if cost + w < best[v]:
                    best[v] = cost + w
                    q.append((v, cost + w))
        stops += 1

    return best[dst] if best[dst] != float("inf") else -1
```

### Complexity

- **Time:** $O(k \cdot E)$ worst case.
- **Space:** $O(n + E)$.

## Approach 3: Dijkstra on (cost, city, stops)

### Intuition

A min-heap ordered by cost, with each state carrying how many stops it used. Pop the cheapest; if it is `dst`, return its cost. Expand a neighbour only if `stops < k + 1`. Because a node can be reached optimally with different stop counts, track the best stop count seen per node and allow re-expansion when the new state uses fewer stops.

### Algorithm

1. `heap = [(0, src, 0)]`. `min_stops = {}`.
2. Pop `(cost, u, stops)`. If `u == dst`, return `cost`. If `stops > k` or `stops >= min_stops.get(u, inf)`, skip. Set `min_stops[u] = stops`. For each `(v, w)`, push `(cost + w, v, stops + 1)`.
3. Return `-1` if the heap empties.

```python
import heapq
from collections import defaultdict

def findCheapestPrice(n: int, flights: list[list[int]], src: int, dst: int, k: int) -> int:
    adj = defaultdict(list)
    for u, v, w in flights:
        adj[u].append((v, w))

    heap = [(0, src, 0)]
    min_stops = {}
    while heap:
        cost, u, stops = heapq.heappop(heap)
        if u == dst:
            return cost
        if stops > k or stops >= min_stops.get(u, float("inf")):
            continue
        min_stops[u] = stops
        for v, w in adj[u]:
            heapq.heappush(heap, (cost + w, v, stops + 1))

    return -1
```

### Complexity

- **Time:** $O(E \cdot k \log(E \cdot k))$.
- **Space:** $O(E \cdot k)$.

## Common Pitfalls

- **Relaxing in place in Bellman-Ford.** Without the `prev` snapshot, a single round can chain two or more flights and undercount stops.
- **Off-by-one on rounds.** `k` stops means `k + 1` edges, so `k + 1` relaxation rounds.
- **Plain Dijkstra by cost only.** It finalizes a node the first time it is popped, which may use too many stops; you must let a cheaper-in-stops path re-expand.
- **Pruning by cost alone in BFS.** A pricier path with fewer stops can still be the one that reaches `dst` within budget — the stop dimension matters.

## The keystone

Add a resource budget (stops, moves, uses) to a shortest-path problem and the state gains a dimension. Bellman-Ford handles it most naturally because its rounds *are* the edge-count axis. This closes [Advanced Graphs](/citadel/dsa/advanced-graphs); the "state = position + budget" idea carries straight into [1-D Dynamic Programming](/citadel/dsa/1-d-dynamic-programming).
