---
title: Alien Dictionary - Recover the Alphabet by Topological Sort
description: Deducing letter order from a list of words sorted in an unknown alphabet — extract one ordering edge from each adjacent word pair, then topologically sort, detecting contradictions.
date: 2022-02-17
draft: false
slug: /dsa/alien-dictionary
tags:
  - Competitive Programming
  - Graphs
  - Topological Sort
---

Each adjacent pair of words gives one fact: at the first position where they differ, the earlier word's letter precedes the later word's letter. Those facts are directed edges over the letters; any topological order of that graph is a valid alphabet. A cycle means the input is inconsistent.

## Description

Given a list of `words` sorted lexicographically by the rules of an unknown language, return a string of its letters in order. If the order is invalid, return `""`. If several orders are possible, return any.

**Example**

```
Input:  words = ["wrt","wrf","er","ett","rftt"]
Output: "wertf"

Input:  words = ["z","x","z"]
Output: ""
```

**Constraints**

- $1 \le \text{words.length} \le 100$; $1 \le \text{words}[i].length \le 100$; lowercase letters.

## Prerequisites

- Deriving edges from adjacent word comparisons.
- Topological sort (Kahn or DFS) with cycle detection.
- The prefix edge case: `["abc", "ab"]` is invalid.

## Approach 1: Kahn's algorithm (BFS)

### Intuition

Build a graph over the letters that appear. For each adjacent word pair, scan to the first differing character and add edge `a → b`. Then peel letters with in-degree 0. If you emit fewer letters than exist, there is a cycle.

### Algorithm

1. `adj = {c: set() for c in all letters}`, `indeg = {c: 0 for c in all letters}`.
2. For each adjacent pair `(w1, w2)`: find the first index `i` where they differ. If none and `len(w1) > len(w2)`, return `""` (bad prefix). Else if `w2[i] not in adj[w1[i]]`, add it and `indeg[w2[i]] += 1`.
3. Queue all in-degree-0 letters; pop, append to `order`, decrement successors, enqueue at 0.
4. Return `order` if it contains every letter, else `""`.

```python
from collections import deque

def alienOrder(words: list[str]) -> str:
    adj = {c: set() for w in words for c in w}
    indeg = {c: 0 for c in adj}

    for w1, w2 in zip(words, words[1:]):
        for a, b in zip(w1, w2):
            if a != b:
                if b not in adj[a]:
                    adj[a].add(b)
                    indeg[b] += 1
                break
        else:
            if len(w1) > len(w2):
                return ""

    q = deque(c for c in indeg if indeg[c] == 0)
    order = []
    while q:
        c = q.popleft()
        order.append(c)
        for nxt in adj[c]:
            indeg[nxt] -= 1
            if indeg[nxt] == 0:
                q.append(nxt)

    return "".join(order) if len(order) == len(adj) else ""
```

### Complexity

- **Time:** $O(C)$ where `C` is the total length of all words.
- **Space:** $O(1)$ — at most 26 letters and 26² edges.

## Approach 2: DFS post-order

### Intuition

Same edge extraction. Then DFS each letter, marking grey on the path and black when done; append to `order` on finish. Reverse `order`. A grey re-visit is a cycle → `""`.

### Algorithm

1. Build `adj` as above (`""` return on the prefix violation).
2. `state = {}` (`1` grey, `2` black). `dfs(c)`: if grey, signal cycle; if black, return. Mark grey; `dfs` each successor; mark black; append `c`.
3. `dfs` every letter. If a cycle was signalled, return `""`. Return the reversed `order`.

```python
def alienOrder(words: list[str]) -> str:
    adj = {c: set() for w in words for c in w}

    for w1, w2 in zip(words, words[1:]):
        for a, b in zip(w1, w2):
            if a != b:
                adj[a].add(b)
                break
        else:
            if len(w1) > len(w2):
                return ""

    state = {}
    order = []
    ok = True

    def dfs(c):
        nonlocal ok
        state[c] = 1
        for nxt in adj[c]:
            if state.get(nxt) == 1:
                ok = False
                return
            if state.get(nxt) is None:
                dfs(nxt)
        state[c] = 2
        order.append(c)

    for c in adj:
        if state.get(c) is None:
            dfs(c)
            if not ok:
                return ""

    return "".join(reversed(order))
```

### Complexity

- **Time:** $O(C)$.
- **Space:** $O(1)$ bounded by the alphabet.

## Common Pitfalls

- **Missing the prefix rule.** `["abc", "ab"]` has no differing character but the longer word comes first — invalid, return `""`.
- **Adding more than one edge per pair.** Only the *first* differing position is a fact; later characters tell you nothing.
- **Dropping letters that have no edges.** Every letter that appears must be in the output; seed `adj` / `indeg` from all characters.
- **Counting a duplicate edge in the in-degree.** Guard with `if b not in adj[a]` before incrementing.

## The keystone

"Reconstruct an order from pairwise constraints" is topological sort once you see the constraints as edges; the only problem-specific work is extracting exactly the right edges — here, one per adjacent word pair. It is [Course Schedule II](/citadel/dsa/course-schedule-ii) wearing a costume.
