---
title: Clone Graph - Copy Nodes Once, Then Wire the Edges
description: A deep copy of a connected undirected graph — traverse from the entry node, create each clone the first time you see the original, and use a map to avoid cloning it twice.
date: 2021-12-18
draft: false
slug: /dsa/clone-graph
tags:
  - Competitive Programming
  - Graphs
  - Hash Table
---

The whole problem is "don't clone the same node twice". Keep a map from original node to its clone. On first sight of a node, make its clone and record it; then recurse into neighbours, appending their clones to the current clone's neighbour list.

## Description

Given a reference to a node in a connected undirected graph, return a deep copy. Each node holds a value and a list of neighbours.

**Example**

```
Input:  adjList = [[2,4],[1,3],[2,4],[1,3]]
Output: [[2,4],[1,3],[2,4],[1,3]]   (an isomorphic, independent copy)
```

**Constraints**

- $0 \le \text{number of nodes} \le 100$
- Node values are unique, `1..100`; no self-loops, no repeated edges; the graph is connected.

## Prerequisites

- DFS or BFS over an adjacency list.
- A hash map keyed by node identity (not value) to memoize clones.

## Approach 1: Depth-First Search

### Intuition

`clone(node)`: if `node` is already in the map, return its clone. Otherwise create the clone, store it *before* recursing (so a cycle back to `node` terminates), then fill `clone.neighbors` with `clone(nei)` for each neighbour.

### Algorithm

1. `old_to_new = {}`.
2. `dfs(node)`: if `node in old_to_new`, return it. Create `copy = Node(node.val)`, set `old_to_new[node] = copy`. For each `nei` in `node.neighbors`, `copy.neighbors.append(dfs(nei))`. Return `copy`.
3. Return `dfs(node)` if `node` else `None`.

```python
def cloneGraph(node: "Node") -> "Node":
    old_to_new = {}

    def dfs(cur):
        if cur in old_to_new:
            return old_to_new[cur]
        copy = Node(cur.val)
        old_to_new[cur] = copy
        for nei in cur.neighbors:
            copy.neighbors.append(dfs(nei))
        return copy

    return dfs(node) if node else None
```

### Complexity

- **Time:** $O(V + E)$ — each node and edge is processed once.
- **Space:** $O(V)$ for the map and recursion stack.

## Approach 2: Breadth-First Search

### Intuition

Clone the entry node, enqueue the original. For each dequeued original, iterate its neighbours: clone any not yet in the map (and enqueue it), then link the current clone to the neighbour's clone.

### Algorithm

1. If `node` is `None`, return `None`. `old_to_new = {node: Node(node.val)}`, `q = deque([node])`.
2. While `q`: pop `cur`. For each `nei`: if `nei not in old_to_new`, `old_to_new[nei] = Node(nei.val)` and enqueue `nei`. Append `old_to_new[nei]` to `old_to_new[cur].neighbors`.

```python
from collections import deque

def cloneGraph(node: "Node") -> "Node":
    if not node:
        return None
    old_to_new = {node: Node(node.val)}
    q = deque([node])
    while q:
        cur = q.popleft()
        for nei in cur.neighbors:
            if nei not in old_to_new:
                old_to_new[nei] = Node(nei.val)
                q.append(nei)
            old_to_new[cur].neighbors.append(old_to_new[nei])
    return old_to_new[node]
```

### Complexity

- **Time:** $O(V + E)$.
- **Space:** $O(V)$ for the map and queue.

## Common Pitfalls

- **Storing the clone *after* recursing.** With a cycle, the recursion re-enters `node` before the map has it and loops forever. Insert into the map immediately after constructing the clone.
- **Keying the map by `node.val`.** Values are unique here so it happens to work, but the correct key is node identity — key by the node object.
- **Not handling the empty graph.** `node is None` must return `None`; a single node with no neighbours must return a lone clone.
- **Sharing neighbour lists.** Build a fresh list on each clone; never alias the original's `neighbors`.

## The keystone

A deep copy is a traversal plus a memo table from original to copy — the same shape as memoized recursion, where the table stops you recomputing (here, re-cloning) a node you have already handled. [Course Schedule](/citadel/dsa/course-schedule) keeps the traversal but swaps the copy for a cycle check.
