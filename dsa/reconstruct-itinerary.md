---
title: Reconstruct Itinerary - Eulerian Path in Lexical Order
description: Ordering flight tickets into one trip from JFK using every ticket once — a Hierholzer depth-first walk over lexicographically sorted destinations, appending airports as the walk dead-ends.
date: 2022-01-19
draft: false
slug: /dsa/reconstruct-itinerary
tags:
  - Competitive Programming
  - Graphs
  - Eulerian Path
---

Each ticket is a directed edge; a valid itinerary uses every edge exactly once — an Eulerian path. Hierholzer's algorithm walks greedily until stuck, then unwinds, prepending each stuck airport. Visiting neighbours in sorted order makes the result the lexicographically smallest such path.

## Description

Given a list of airline `tickets` `[from, to]`, reconstruct the itinerary that uses all tickets exactly once and starts at `"JFK"`. If several are valid, return the one with the smallest lexical order when read as a single string. A valid itinerary is guaranteed.

**Example**

```
Input:  tickets = [["MUC","LHR"],["JFK","MUC"],["SFO","SJC"],["LHR","SFO"]]
Output: ["JFK","MUC","LHR","SFO","SJC"]

Input:  tickets = [["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],["ATL","JFK"],["ATL","SFO"]]
Output: ["JFK","ATL","JFK","SFO","ATL","SFO"]
```

**Constraints**

- $1 \le \text{tickets.length} \le 300$
- All airports are 3 uppercase letters; a valid itinerary exists.

## Prerequisites

- Eulerian path: a trail using every edge once.
- Hierholzer's algorithm — walk until stuck, backtrack while appending.

## Approach 1: Hierholzer's algorithm (post-order DFS)

### Intuition

Sort each airport's destination list so the smallest is taken first. DFS: from the current airport, repeatedly fly to (and remove) the smallest unused destination. When an airport has no destinations left, prepend it to the result. The recursion naturally handles cycles that must be traversed before a dead-end branch.

### Algorithm

1. Build `adj[src]` as a min-heap (or sorted deque) of destinations.
2. `dfs(u)`: while `adj[u]` is non-empty, pop its smallest destination `v` and `dfs(v)`. Then append `u` to `route`.
3. `dfs("JFK")`; return `route` reversed.

```python
import heapq
from collections import defaultdict

def findItinerary(tickets: list[list[str]]) -> list[str]:
    adj = defaultdict(list)
    for src, dst in tickets:
        heapq.heappush(adj[src], dst)

    route = []

    def dfs(u):
        heap = adj[u]
        while heap:
            v = heapq.heappop(heap)
            dfs(v)
        route.append(u)

    dfs("JFK")
    return route[::-1]
```

### Complexity

- **Time:** $O(E \log E)$ — each edge is used once, heap ops are logarithmic.
- **Space:** $O(E)$ for the adjacency heaps and recursion.

## Approach 2: Backtracking

### Intuition

Treat it as a search: try destinations in sorted order, mark the ticket used, recurse; if the branch fails to consume all tickets, undo and try the next. The first complete path found is lexicographically smallest.

### Algorithm

1. Build `adj[src]` as a sorted list with a per-edge `used` flag (or a Counter of `(src, dst)`).
2. `dfs(u, path)`: if `len(path) == len(tickets) + 1`, return `True`. For each unused ticket `u → v` in sorted order: mark used, append `v`, if `dfs(v, path)` return `True`, else undo.
3. Start `dfs("JFK", ["JFK"])`.

```python
from collections import defaultdict

def findItinerary(tickets: list[list[str]]) -> list[str]:
    adj = defaultdict(list)
    for src, dst in sorted(tickets):
        adj[src].append([dst, False])

    target = len(tickets) + 1
    path = ["JFK"]

    def dfs(u):
        if len(path) == target:
            return True
        for entry in adj[u]:
            if entry[1]:
                continue
            entry[1] = True
            path.append(entry[0])
            if dfs(entry[0]):
                return True
            path.pop()
            entry[1] = False
        return False

    dfs("JFK")
    return path
```

### Complexity

- **Time:** exponential worst case, acceptable for `E ≤ 300`; Hierholzer is strictly better.
- **Space:** $O(E)$.

## Common Pitfalls

- **Appending pre-order instead of post-order in Hierholzer.** You must append an airport *after* its outgoing edges are exhausted, then reverse; appending on entry gives a wrong order when cycles are present.
- **Not removing the edge when you take it.** Each ticket is used once; pop it from the adjacency structure.
- **Forgetting to sort destinations.** Lexical order requires taking the smallest available destination first.
- **Assuming a simple greedy walk works.** A naive "always take smallest" without backtracking or Hierholzer's post-order can strand tickets on a branch you entered too early.

## The keystone

An itinerary using every ticket once is an Eulerian path, and Hierholzer's "walk, dead-end, unwind" is its linear-time construction. The post-order-then-reverse trick reappears wherever a traversal must finish a subtree before committing its node — you saw it in [Course Schedule II](/citadel/dsa/course-schedule-ii).
