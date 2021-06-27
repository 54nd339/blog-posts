---
title: Minimum Window Substring - Expand to Cover, Contract to Trim
description: The shortest substring containing every character of a target with multiplicity — a window grows until it covers the target, then shrinks from the left while it still covers.
date: 2021-06-27
draft: false
slug: /dsa/minimum-window-substring
tags:
  - Competitive Programming
  - Sliding Window
  - Hash Table
---

The window alternates two moves: the right edge expands until the window *covers* `t`, then the left edge contracts as far as it can while still covering `t`, recording the size at each valid moment. The trick is checking coverage without rescanning.

## Description

Given strings `s` and `t`, return the shortest substring of `s` that contains every character of `t`, counting multiplicity. If none exists, return `""`. The answer is guaranteed unique.

**Example**

```
Input:  s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"

Input:  s = "a", t = "a"
Output: "a"

Input:  s = "a", t = "aa"
Output: ""
```

**Constraints**

- $1 \le \text{s.length}, \text{t.length} \le 10^5$
- `s` and `t` consist of uppercase and lowercase English letters.

## Prerequisites

- [Sliding window](/citadel/dsa/sliding-window), variable size, "shortest window meeting a condition".
- Hash-map counts and a `have`/`need` satisfaction counter.

## Approach 1: Brute Force

### Intuition

Try every substring; keep the shortest that contains all of `t`.

### Algorithm

1. Build `need = Counter(t)`.
2. For each start `i`: grow an end `j`, maintaining a window count map; when the window contains every character of `t` with sufficient count, record `s[i:j+1]` if shorter than the best, then move to the next `i`.
3. Return the best (or `""`).

```python
from collections import Counter

def minWindow(s: str, t: str) -> str:
    if not t or not s:
        return ""
    need = Counter(t)
    best = ""
    for i in range(len(s)):
        window = Counter()
        for j in range(i, len(s)):
            window[s[j]] += 1
            if all(window[c] >= need[c] for c in need):
                cand = s[i:j + 1]
                if best == "" or len(cand) < len(best):
                    best = cand
                break
    return best
```

### Complexity

- **Time:** $O(m + n^2 \cdot u)$ — `u` distinct characters in `t`.
- **Space:** $O(k)$ for the count maps.

## Approach 2: Sliding Window

### Intuition

Let `required` be the number of distinct characters in `t`, and `formed` the number of them whose window count has reached its needed amount. Expand `right` until `formed == required`; while covered, record the window and shrink `left`; a removal that drops a count below its need decrements `formed` and ends the shrink.

### Algorithm

1. `need = Counter(t)`, `required = len(need)`, `window = {}`, `formed = 0`, `left = 0`, best length `= inf`.
2. For each `right`, character `c`: `window[c] += 1`; if `c in need` and `window[c] == need[c]`, `formed += 1`.
3. While `formed == required`: record `[left, right]` if smaller; remove `s[left]` from `window`; if `s[left] in need` and `window[s[left]] < need[s[left]]`, `formed -= 1`; `left += 1`.
4. Return the best slice or `""`.

```python
from collections import Counter

def minWindow(s: str, t: str) -> str:
    if not s or not t or len(t) > len(s):
        return ""
    need = Counter(t)
    required = len(need)
    window = {}
    formed = 0
    left = 0
    best_len = float("inf")
    best_left = 0
    for right, c in enumerate(s):
        window[c] = window.get(c, 0) + 1
        if c in need and window[c] == need[c]:
            formed += 1
        while formed == required:
            if right - left + 1 < best_len:
                best_len = right - left + 1
                best_left = left
            lc = s[left]
            window[lc] -= 1
            if lc in need and window[lc] < need[lc]:
                formed -= 1
            left += 1
    return "" if best_len == float("inf") else s[best_left:best_left + best_len]
```

### Complexity

- **Time:** $O(n + m)$ — `right` and `left` each traverse `s` once.
- **Space:** $O(k)$ for the count maps.

## Common Pitfalls

- **`window[c] == need[c]` vs `>=` when updating `formed`.** With `>=`, the same character bumps `formed` repeatedly and a non-covering window looks complete. Each distinct character must contribute to `formed` exactly once.
- **Recording the best window after shrinking past validity.** Record at the top of the `while`, before `left += 1`.
- **Checking coverage as `window == need`.** The window may hold extra copies or unrelated characters; coverage is `window[c] >= need[c]` for each needed `c`, captured by `formed`.

## The keystone

Expand-to-satisfy then contract-to-minimise is the template for "shortest window meeting a condition", the mirror of "longest window keeping a condition" in [Longest Substring Without Repeating Characters](/citadel/dsa/longest-substring-without-repeating-characters). The `formed`/`required` counter compresses "does the window satisfy a multiset lower bound?" into one integer comparison per pointer move.
