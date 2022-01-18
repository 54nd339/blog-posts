---
title: Word Ladder - Shortest Path in a Graph of One-Letter Edits
description: The fewest single-letter changes from one word to another through a dictionary — BFS on the implicit graph where words one edit apart are neighbours, using wildcard patterns to find them fast.
date: 2022-01-18
draft: false
slug: /dsa/word-ladder
tags:
  - Competitive Programming
  - Graphs
  - Breadth-First Search
---

Words are nodes; an edge joins two words differing in exactly one letter. The shortest transformation is a shortest path, so BFS. The trick is finding neighbours quickly: precompute buckets keyed by patterns like `h*t`, so all words matching a pattern are neighbours of each other.

## Description

Given `beginWord`, `endWord`, and a `wordList`, return the number of words in the shortest transformation sequence from `beginWord` to `endWord`, changing one letter at a time, where every intermediate word is in `wordList`. Return `0` if no such sequence exists.

**Example**

```
Input:  beginWord = "hit", endWord = "cog",
        wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5   (hit → hot → dot → dog → cog)

Input:  beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log"]
Output: 0
```

**Constraints**

- $1 \le \text{wordLength} \le 10$
- $1 \le \text{wordList.length} \le 5000$; all words the same length, lowercase.

## Prerequisites

- BFS gives the shortest path on an unweighted graph.
- Wildcard pattern buckets (`h*t`) to enumerate one-edit neighbours in $O(L \cdot 26)$ or better.

## Approach 1: BFS with wildcard pattern buckets

### Intuition

Build `patterns[pattern] = [words matching it]`, where each word of length `L` yields `L` patterns with one position starred. From a word, its neighbours are every word sharing any of its `L` patterns. BFS layer by layer from `beginWord`; the layer index on reaching `endWord` is the answer.

### Algorithm

1. If `endWord not in wordList`, return `0`.
2. Build `patterns`: for each word, for each `i`, append the word to `patterns[word[:i] + '*' + word[i+1:]]`.
3. BFS from `beginWord` with `steps = 1`. For each word popped, for each of its `L` patterns, for each neighbour in that bucket not yet visited: if it is `endWord`, return `steps + 1`; else mark visited and enqueue.
4. Increase `steps` once per layer. Return `0` if the queue empties.

```python
from collections import deque, defaultdict

def ladderLength(beginWord: str, endWord: str, wordList: list[str]) -> int:
    words = set(wordList)
    if endWord not in words:
        return 0

    L = len(beginWord)
    patterns = defaultdict(list)
    for word in words:
        for i in range(L):
            patterns[word[:i] + "*" + word[i + 1:]].append(word)

    visited = {beginWord}
    q = deque([beginWord])
    steps = 1
    while q:
        for _ in range(len(q)):
            word = q.popleft()
            if word == endWord:
                return steps
            for i in range(L):
                for nei in patterns[word[:i] + "*" + word[i + 1:]]:
                    if nei not in visited:
                        visited.add(nei)
                        q.append(nei)
        steps += 1
    return 0
```

### Complexity

- **Time:** $O(N \cdot L^2)$ where `N` is the word count (`L` patterns per word, each of length `L` to build).
- **Space:** $O(N \cdot L^2)$ for the pattern buckets.

## Approach 2: Bidirectional BFS

### Intuition

Search from both `beginWord` and `endWord`, always expanding the smaller frontier. The two searches meet in about half the depth, cutting the branching-factor blow-up roughly to its square root.

### Algorithm

1. Same pattern buckets. Keep two sets `front` and `back`, initialised to `{beginWord}` and `{endWord}`.
2. Each round: swap so `front` is the smaller. Build `next_front` by expanding every word in `front` over its patterns. If any expanded word is in `back`, return `steps + 1`. Otherwise add new words to `visited`, set `front = next_front`, `steps += 1`.
3. Return `0` if a frontier becomes empty.

```python
from collections import defaultdict

def ladderLength(beginWord: str, endWord: str, wordList: list[str]) -> int:
    words = set(wordList)
    if endWord not in words:
        return 0

    L = len(beginWord)
    patterns = defaultdict(list)
    for word in words:
        for i in range(L):
            patterns[word[:i] + "*" + word[i + 1:]].append(word)

    front, back = {beginWord}, {endWord}
    visited = {beginWord, endWord}
    steps = 1
    while front and back:
        if len(front) > len(back):
            front, back = back, front
        nxt = set()
        for word in front:
            for i in range(L):
                for nei in patterns[word[:i] + "*" + word[i + 1:]]:
                    if nei in back:
                        return steps + 1
                    if nei not in visited:
                        visited.add(nei)
                        nxt.add(nei)
        front = nxt
        steps += 1
    return 0
```

### Complexity

- **Time:** $O(N \cdot L^2)$ worst case, far less in practice.
- **Space:** $O(N \cdot L^2)$.

## Common Pitfalls

- **Rebuilding neighbours by trying all 26 letters at every position each visit.** Fine for small inputs, but the pattern buckets amortise it; either way, do it once per word, not once per queue pop of a repeat.
- **Off-by-one on the count.** The answer counts *words* in the path, not edges — `hit → ... → cog` of length 4 edges is 5.
- **Not checking `endWord in wordList` up front.** If it is absent, no sequence can end there.
- **Marking visited late.** Add to `visited` on enqueue; otherwise the same word is expanded from multiple predecessors in one layer.

## The keystone

Any "minimum number of steps / moves / transformations" on states with uniform-cost transitions is BFS on an implicit graph — the work is defining the neighbour relation cheaply. That closes the [Graphs](/citadel/dsa/graphs) section; [Advanced Graphs](/citadel/dsa/advanced-graphs) adds edge weights, where BFS gives way to Dijkstra and MST algorithms.
