---
title: Course Schedule - Can You Finish, or Is There a Cycle
description: Whether all courses can be taken given prerequisite pairs — the question is exactly whether the dependency graph has a cycle, answered by DFS colouring or Kahn's algorithm.
date: 2022-01-12
draft: false
slug: /dsa/course-schedule
tags:
  - Competitive Programming
  - Graphs
  - Topological Sort
---

Model each course as a node with an edge from prerequisite to course. You can finish every course iff this directed graph is acyclic. Two standard tests: DFS with a three-state colour, or repeatedly removing nodes with no remaining prerequisites (Kahn) and checking that all of them go.

## Description

There are `numCourses` courses labelled `0` to `numCourses - 1`. `prerequisites[i] = [a, b]` means you must take `b` before `a`. Return `true` if you can finish all courses.

**Example**

```
Input:  numCourses = 2, prerequisites = [[1,0]]
Output: true

Input:  numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: false
```

**Constraints**

- $1 \le \text{numCourses} \le 2000$
- $0 \le \text{prerequisites.length} \le 5000$; pairs are distinct.

## Prerequisites

- Adjacency-list construction from edge pairs.
- Cycle detection in a directed graph (DFS colours or in-degree peeling).

## Approach 1: DFS cycle detection

### Intuition

Colour each node white (unseen), grey (on the current DFS path), black (fully explored). If DFS reaches a grey node, the path loops — cycle. A node that finishes without hitting grey is black and safe.

### Algorithm

1. Build `adj[course] = list of prerequisites`.
2. `state = [0] * n` (0 white, 1 grey, 2 black).
3. `dfs(u)`: if `state[u] == 1`, return `False` (cycle); if `state[u] == 2`, return `True`. Set `state[u] = 1`; for each `v` in `adj[u]`, if not `dfs(v)`, return `False`. Set `state[u] = 2`; return `True`.
4. Return `all(dfs(u) for u in range(n))`.

```python
def canFinish(numCourses: int, prerequisites: list[list[int]]) -> bool:
    adj = [[] for _ in range(numCourses)]
    for a, b in prerequisites:
        adj[a].append(b)

    state = [0] * numCourses  # 0 = white, 1 = grey, 2 = black

    def dfs(u):
        if state[u] == 1:
            return False
        if state[u] == 2:
            return True
        state[u] = 1
        for v in adj[u]:
            if not dfs(v):
                return False
        state[u] = 2
        return True

    return all(dfs(u) for u in range(numCourses))
```

### Complexity

- **Time:** $O(V + E)$.
- **Space:** $O(V + E)$ for the adjacency list and recursion.

## Approach 2: Kahn's algorithm (BFS topological sort)

### Intuition

Repeatedly take a course with in-degree 0 (no unmet prerequisite), remove it, and decrement its dependents' in-degrees. If you can remove all `numCourses`, the graph is acyclic.

### Algorithm

1. Build `adj[b] = list of courses that need b`, and `indeg[a]` = number of prerequisites of `a`.
2. Queue all courses with `indeg == 0`. `taken = 0`.
3. While the queue is non-empty: pop `u`, `taken += 1`; for each `v` in `adj[u]`, `indeg[v] -= 1`, and enqueue `v` when it hits 0.
4. Return `taken == numCourses`.

```python
from collections import deque

def canFinish(numCourses: int, prerequisites: list[list[int]]) -> bool:
    adj = [[] for _ in range(numCourses)]
    indeg = [0] * numCourses
    for a, b in prerequisites:
        adj[b].append(a)
        indeg[a] += 1

    q = deque(u for u in range(numCourses) if indeg[u] == 0)
    taken = 0
    while q:
        u = q.popleft()
        taken += 1
        for v in adj[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)

    return taken == numCourses
```

### Complexity

- **Time:** $O(V + E)$.
- **Space:** $O(V + E)$.

## Common Pitfalls

- **Two-state visited.** A plain "visited" set cannot tell "on the current path" from "done in an earlier branch". You need the grey/black distinction (or Kahn).
- **Edge direction.** `[a, b]` means `b → a`. Building the list the other way inverts every dependency.
- **Not resetting grey on the way out.** After exploring all of `u`'s successors, mark it black; leaving it grey makes a later, unrelated visit report a false cycle.
- **Disconnected nodes.** Courses with no prerequisites and no dependents still must be counted — iterate all `n` nodes.

## The keystone

"Can this be ordered respecting dependencies" is cycle detection on a directed graph, and both answers come from the same two tools: path-colouring DFS or in-degree peeling. [Course Schedule II](/citadel/dsa/course-schedule-ii) asks for the order itself, which Kahn produces as a side effect.
