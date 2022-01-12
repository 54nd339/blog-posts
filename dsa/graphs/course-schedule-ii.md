---
title: Course Schedule II - Emit the Order, Not Just Yes or No
description: A valid course order given prerequisite pairs — Kahn's algorithm outputs the topological order as it peels in-degree-zero nodes, or DFS post-order reversed does the same.
date: 2022-01-12
draft: false
slug: /dsa/course-schedule-ii
tags:
  - Competitive Programming
  - Graphs
  - Topological Sort
---

[Course Schedule](/citadel/dsa/course-schedule) with the answer upgraded from a boolean to the ordering. Kahn's algorithm records each node as it is removed; DFS records each node when it finishes and reverses. Either detects the cycle that makes an order impossible.

## Description

Given `numCourses` and `prerequisites[i] = [a, b]` (take `b` before `a`), return any valid order in which to take all courses. Return `[]` if impossible.

**Example**

```
Input:  numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: [0,1,2,3]   (or [0,2,1,3])

Input:  numCourses = 2, prerequisites = [[0,1],[1,0]]
Output: []
```

**Constraints**

- $1 \le \text{numCourses} \le 2000$
- $0 \le \text{prerequisites.length} \le \text{numCourses} \cdot (\text{numCourses} - 1)$; pairs distinct.

## Prerequisites

- Topological sort by Kahn's algorithm or DFS post-order.
- Detecting that a cycle leaves nodes unordered.

## Approach 1: Kahn's algorithm (BFS)

### Intuition

Same peeling as Course Schedule, but append each node to `order` as it leaves the queue. If `order` ends up shorter than `numCourses`, a cycle blocked some nodes — return `[]`.

### Algorithm

1. Build `adj[b] → a` and `indeg[a]`.
2. Queue all `indeg == 0` nodes.
3. Pop `u`, append to `order`; for each `v` in `adj[u]`, decrement `indeg[v]`, enqueue at 0.
4. Return `order` if `len(order) == numCourses` else `[]`.

```python
from collections import deque

def findOrder(numCourses: int, prerequisites: list[list[int]]) -> list[int]:
    adj = [[] for _ in range(numCourses)]
    indeg = [0] * numCourses
    for a, b in prerequisites:
        adj[b].append(a)
        indeg[a] += 1

    q = deque(u for u in range(numCourses) if indeg[u] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in adj[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)

    return order if len(order) == numCourses else []
```

### Complexity

- **Time:** $O(V + E)$.
- **Space:** $O(V + E)$.

## Approach 2: DFS post-order

### Intuition

Run DFS following prerequisite edges. When a node's entire subtree of prerequisites is done, push the node. The reversed push order (or building it back-to-front) is a valid topological order. Track a grey set to catch cycles.

### Algorithm

1. Build `adj[a] = prerequisites of a`.
2. `visited` (black) set, `on_path` (grey) set, `order = []`.
3. `dfs(u)`: if `u in on_path`, signal a cycle; if `u in visited`, return. Add to `on_path`; `dfs` each prerequisite; move `u` from `on_path` to `visited`; append `u` to `order`.
4. `dfs` every node. `order` is already prerequisite-first because a node is appended only after its prerequisites.

```python
def findOrder(numCourses: int, prerequisites: list[list[int]]) -> list[int]:
    adj = [[] for _ in range(numCourses)]
    for a, b in prerequisites:
        adj[a].append(b)

    visited, on_path = set(), set()
    order = []
    ok = True

    def dfs(u):
        nonlocal ok
        if u in on_path:
            ok = False
            return
        if u in visited:
            return
        on_path.add(u)
        for v in adj[u]:
            dfs(v)
        on_path.discard(u)
        visited.add(u)
        order.append(u)

    for u in range(numCourses):
        dfs(u)
        if not ok:
            return []
    return order
```

### Complexity

- **Time:** $O(V + E)$.
- **Space:** $O(V + E)$.

## Common Pitfalls

- **Returning a partial order on a cycle.** If `len(order) != numCourses` (Kahn) or a grey node is hit (DFS), the answer is `[]`, not what you have so far.
- **Reversing the DFS order the wrong way.** With `adj[a] = prerequisites of a` and appending on finish, `order` is already correct; with `adj[b] → a` you must reverse. Pick one convention and stay with it.
- **Using one visited set in DFS.** You still need grey vs black to detect the cycle.
- **Isolated nodes.** Courses with no edges appear anywhere in the order; make sure every node is a DFS root / initial queue candidate.

## The keystone

A topological order falls out of the same machinery that detects the cycle — Kahn emits it as it peels, DFS emits it in finish order. [Alien Dictionary](/citadel/dsa/alien-dictionary) is this exact algorithm once you extract the letter-ordering edges from the word list.
