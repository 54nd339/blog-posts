---
title: Permutation in String - A Fixed Window and a Match Counter
description: Whether one string contains a permutation of another — a fixed-size window of character counts slides across the haystack, and a single "how many counts match" tally keeps each step O(1).
date: 2021-06-27
draft: false
slug: /dsa/permutation-in-string
tags:
  - Competitive Programming
  - Sliding Window
  - Strings
---

A permutation of `s1` is any string with exactly `s1`'s character counts. So: does any window of `s2` of length `len(s1)` have the same 26-letter count vector as `s1`? The window size is fixed, which makes the sliding clean.

## Description

Given strings `s1` and `s2`, return `true` if `s2` contains a substring that is a permutation of `s1` (equivalently, an anagram of `s1`).

**Example**

```
Input:  s1 = "ab", s2 = "eidbaooo"
Output: true          # "ba"

Input:  s1 = "ab", s2 = "eidboaoo"
Output: false
```

**Constraints**

- $1 \le \text{s1.length}, \text{s2.length} \le 10^4$
- `s1` and `s2` consist of lowercase English letters.

## Prerequisites

- [Valid Anagram](/citadel/dsa/valid-anagram) — comparing count vectors.
- [Sliding window](/citadel/dsa/sliding-window), fixed size.

## Approach 1: Brute Force

### Intuition

Slide a window of length `len(s1)` and check each by sorting.

### Algorithm

1. Sort `s1`.
2. For each start in `s2` and each end from that start, sort the substring and compare to sorted `s1`.
3. Return `true` on a match.

```python
def checkInclusion(s1: str, s2: str) -> bool:
    target = sorted(s1)
    m = len(s1)
    for i in range(len(s2) - m + 1):
        if sorted(s2[i:i + m]) == target:
            return True
    return False
```

### Complexity

- **Time:** $O(n \cdot m \log m)$ (the NeetCode write-up bounds the fully naive version at $O(n^3 \log n)$).
- **Space:** $O(m)$.

## Approach 2: Hash Table

### Intuition

Build `s1`'s frequency map. For each start in `s2`, extend a window character by character, bailing the moment any character exceeds its allowed count; a full match means a permutation.

### Algorithm

1. Build `need = Counter(s1)`; `required = len(need)`.
2. For each start `i` in `s2`: empty window map, `matched = 0`.
3. Extend `j` from `i`: increment `window[s2[j]]`. If it exceeds `need[s2[j]]`, stop this start. If it equals `need[s2[j]]`, `matched += 1`. If `matched == required`, return `true`.
4. Return `false`.

```python
from collections import Counter

def checkInclusion(s1: str, s2: str) -> bool:
    need = Counter(s1)
    required = len(need)
    for i in range(len(s2)):
        window = Counter()
        matched = 0
        for j in range(i, len(s2)):
            c = s2[j]
            window[c] += 1
            if window[c] > need[c]:
                break
            if window[c] == need[c]:
                matched += 1
            if matched == required:
                return True
    return False
```

### Complexity

- **Time:** $O(n \cdot m)$.
- **Space:** $O(1)$ — at most 26 letters.

## Approach 3: Sliding Window

### Intuition

Maintain the window's 26-count vector and `matches`, the number of letters whose window count equals its `s1` count. A single-letter change flips at most one match; the answer is `matches == 26`.

### Algorithm

1. If `len(s1) > len(s2)`, return `false`.
2. Build `need` and `window` counts for `s1` and the first `len(s1)` characters of `s2`. Set `matches` to the number of equal positions among the 26.
3. Slide: for each new right character, adjust its count and `matches`; for the left character leaving, adjust its count and `matches`. Return `true` whenever `matches == 26`.

```python
def checkInclusion(s1: str, s2: str) -> bool:
    m, n = len(s1), len(s2)
    if m > n:
        return False
    need = [0] * 26
    window = [0] * 26
    for c in s1:
        need[ord(c) - ord("a")] += 1
    for i in range(m):
        window[ord(s2[i]) - ord("a")] += 1

    matches = sum(need[x] == window[x] for x in range(26))

    for i in range(m, n):
        if matches == 26:
            return True
        add = ord(s2[i]) - ord("a")
        window[add] += 1
        if window[add] == need[add]:
            matches += 1
        elif window[add] == need[add] + 1:
            matches -= 1

        rem = ord(s2[i - m]) - ord("a")
        window[rem] -= 1
        if window[rem] == need[rem]:
            matches += 1
        elif window[rem] == need[rem] - 1:
            matches -= 1

    return matches == 26
```

### Complexity

- **Time:** $O(n)$ — a constant number of updates per step.
- **Space:** $O(1)$.

## Common Pitfalls

- **The character leaving the window is at index `i - m`,** not `i - m - 1` or `i - 1`.
- **Updating `matches` by checking only post-change equality.** Catch both transitions: a count reaching equality (`+1`) and a count leaving equality by exactly one — `need[x] + 1` on the way up, `need[x] - 1` on the way down.
- **Forgetting the final `return matches == 26`.** The loop checks at the top, so the last window's state needs one more check after the loop (or check before the slide as shown).

## The keystone

A fixed-length window turns "is there an anagram here?" into "does this count vector match?", updated $O(1)$ per slide. Replacing a full vector comparison with an incrementally maintained match counter is the standard optimisation for "window equals a target profile" — with a threshold instead of strict equality it becomes [Minimum Window Substring](/citadel/dsa/minimum-window-substring).
