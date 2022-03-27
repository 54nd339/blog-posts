---
title: Word Break - Can the String Be Cut Into Dictionary Words
description: Whether a string splits into a sequence of dictionary words — a prefix DP where position i is reachable if some earlier reachable position j has s from j to i in the dictionary.
date: 2022-03-27
draft: false
slug: /dsa/word-break
tags:
  - Competitive Programming
  - Dynamic Programming
  - Strings
---

`dp[i]` means "`s[:i]` can be fully segmented". It is `True` when some `j < i` has `dp[j]` true *and* `s[j:i]` is a dictionary word. Build left to right; `dp[n]` is the answer.

## Description

Given a string `s` and a dictionary `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of one or more dictionary words. Words may be reused.

**Example**

```
Input:  s = "leetcode", wordDict = ["leet","code"]
Output: true

Input:  s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
Output: false
```

**Constraints**

- $1 \le \text{s.length} \le 300$
- $1 \le \text{wordDict.length} \le 1000$; word lengths $\le 20$; lowercase.

## Prerequisites

- Prefix DP: `dp[i]` depends on earlier `dp[j]` plus a slice check.
- A set for $O(1)$ word lookup; optionally bound `j` by the max word length.

## Approach 1: Bottom-up DP

### Intuition

`dp[0] = True` (empty prefix). For each end `i`, scan starts `j < i`; if `dp[j]` and `s[j:i]` is in the word set, set `dp[i] = True` and stop.

### Algorithm

1. `words = set(wordDict)`, `dp = [False] * (n + 1)`, `dp[0] = True`.
2. For `i` from `1` to `n`: for `j` from `0` to `i - 1`: if `dp[j]` and `s[j:i] in words`: `dp[i] = True`, break.
3. Return `dp[n]`.

```python
def wordBreak(s: str, wordDict: list[str]) -> bool:
    words = set(wordDict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in words:
                dp[i] = True
                break
    return dp[n]
```

### Complexity

- **Time:** $O(n^2 \cdot L)$ with slice hashing of length up to `L`; $O(n \cdot m \cdot L)$ if you only try the `m` dictionary words at each `i`.
- **Space:** $O(n)$.

## Approach 2: DP bounded by max word length

### Intuition

`s[j:i]` can only be a word if `i - j <= max_len`. Start `j` at `max(0, i - max_len)` to skip hopeless slices.

### Algorithm

Same as Approach 1 with the inner loop `for j in range(max(0, i - max_len), i)`.

```python
def wordBreak(s: str, wordDict: list[str]) -> bool:
    words = set(wordDict)
    max_len = max(map(len, wordDict))
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    for i in range(1, n + 1):
        for j in range(max(0, i - max_len), i):
            if dp[j] and s[j:i] in words:
                dp[i] = True
                break
    return dp[n]
```

### Complexity

- **Time:** $O(n \cdot \text{max\_len} \cdot L)$.
- **Space:** $O(n)$.

## Approach 3: BFS over reachable indices

### Intuition

Nodes are string positions; an edge `j → i` exists when `s[j:i]` is a word. BFS from 0; if you reach `n`, the string breaks.

### Algorithm

1. `q = deque([0])`, `seen = {0}`.
2. Pop `start`; for `end` from `start + 1` to `n`: if `s[start:end] in words` and `end` unseen: if `end == n` return `True`; enqueue and mark.
3. Return `False` when the queue drains.

```python
from collections import deque

def wordBreak(s: str, wordDict: list[str]) -> bool:
    words = set(wordDict)
    n = len(s)
    q = deque([0])
    seen = {0}
    while q:
        start = q.popleft()
        for end in range(start + 1, n + 1):
            if end in seen:
                continue
            if s[start:end] in words:
                if end == n:
                    return True
                seen.add(end)
                q.append(end)
    return False
```

### Complexity

- **Time:** $O(n^2 \cdot L)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **`dp[0]` not `True`.** The empty prefix is trivially segmentable; without it nothing propagates.
- **Requiring exact dictionary coverage but forgetting reuse.** Words may repeat; nothing in the recurrence forbids it, so don't add a "used" set.
- **Trying every substring as a whole.** The problem is a *sequence* of words, so you need the `dp[j]` prefix condition, not just "is `s` one word".
- **Not deduping BFS indices.** Without `seen`, position `i` is expanded once per path that reaches it.

## The keystone

Segmentation is a reachability DP over prefixes: a position is reachable if a previous reachable position connects to it by a valid piece. It is [Decode Ways](/citadel/dsa/decode-ways) with an arbitrary-length, dictionary-defined piece instead of a one- or two-digit one.
