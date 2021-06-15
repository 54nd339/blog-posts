---
title: Longest Repeating Character Replacement - Window Size Minus Its Majority
description: The longest substring you can make uniform with at most k replacements — a window is feasible when its length minus its most common character's count is at most k.
date: 2021-06-15
draft: false
slug: /dsa/longest-repeating-character-replacement
tags:
  - Competitive Programming
  - Sliding Window
  - Strings
---

The key quantity for a window is how many characters you would change to make it uniform: window length minus the count of its most frequent letter. If that is `<= k`, the window is feasible. Slide a window and keep the largest feasible one.

## Description

Given a string `s` of uppercase English letters and an integer `k`, you may replace at most `k` characters with any uppercase letters. Return the length of the longest substring containing a single repeated letter you can obtain.

**Example**

```
Input:  s = "ABAB", k = 2
Output: 4          # replace the two A's or the two B's

Input:  s = "AABABBA", k = 1
Output: 4          # "AABA" -> "AAAA" or "ABBB..." variants
```

**Constraints**

- $1 \le \text{s.length} \le 10^5$
- `s` consists of uppercase English letters.
- $0 \le k \le \text{s.length}$

## Prerequisites

- [Sliding window](/citadel/dsa/sliding-window) with a frequency array.
- The observation that the window's majority count only needs to *grow* for the answer to stay correct.

## Approach 1: Brute Force

### Intuition

Check every substring: it is valid iff `length - max_frequency <= k`.

### Algorithm

1. `best = 0`.
2. For each start `i`: empty frequency map, `max_freq = 0`.
3. For each end `j` from `i`: increment `count[s[j]]`, update `max_freq`; if `(j - i + 1) - max_freq <= k`, update `best`.
4. Return `best`.

```python
from collections import defaultdict

def characterReplacement(s: str, k: int) -> int:
    n = len(s)
    best = 0
    for i in range(n):
        count = defaultdict(int)
        max_freq = 0
        for j in range(i, n):
            count[s[j]] += 1
            max_freq = max(max_freq, count[s[j]])
            if (j - i + 1) - max_freq <= k:
                best = max(best, j - i + 1)
    return best
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(m)$ — at most 26 counters.

## Approach 2: Sliding Window (per target character)

### Intuition

Fix the letter you want the window made of. Slide a window; it is valid while the count of *non-target* characters is `<= k`.

### Algorithm

1. Collect the distinct characters of `s`.
2. For each target `ch`: `left = 0`, `count = 0` (occurrences of `ch` in the window).
3. For each `right`: if `s[right] == ch`, `count += 1`. While `(right - left + 1) - count > k`: if `s[left] == ch`, `count -= 1`; `left += 1`. Update `best`.
4. Return `best`.

```python
def characterReplacement(s: str, k: int) -> int:
    best = 0
    for ch in set(s):
        left = count = 0
        for right in range(len(s)):
            if s[right] == ch:
                count += 1
            while (right - left + 1) - count > k:
                if s[left] == ch:
                    count -= 1
                left += 1
            best = max(best, right - left + 1)
    return best
```

### Complexity

- **Time:** $O(m \cdot n)$ — a full sweep per distinct character.
- **Space:** $O(m)$.

## Approach 3: Sliding Window (Optimal)

### Intuition

Track the whole window's frequency map and `max_count`, the largest single-letter count seen in it. When `window_len - max_count > k`, slide `left` by one. `max_count` is never decreased — a stale value just prevents the window from shrinking, and the window can only *grow* when a genuinely higher `max_count` appears, so the final answer is unaffected.

### Algorithm

1. `count = [0]*26`, `left = 0`, `max_count = 0`, `best = 0`.
2. For each `right`: increment `count[s[right]]`; `max_count = max(max_count, count[s[right]])`.
3. If `(right - left + 1) - max_count > k`: decrement `count[s[left]]`, `left += 1`.
4. Update `best` with `right - left + 1`.
5. Return `best`.

```python
def characterReplacement(s: str, k: int) -> int:
    count = [0] * 26
    left = 0
    max_count = 0
    best = 0
    for right in range(len(s)):
        idx = ord(s[right]) - ord("A")
        count[idx] += 1
        max_count = max(max_count, count[idx])
        if (right - left + 1) - max_count > k:
            count[ord(s[left]) - ord("A")] -= 1
            left += 1
        best = max(best, right - left + 1)
    return best
```

### Complexity

- **Time:** $O(n)$ — one pass, $O(1)$ per step.
- **Space:** $O(1)$ — 26 counters.

## Common Pitfalls

- **The feasibility expression.** `window_len - max_frequency` is the number of replacements needed. Comparing `max_frequency` to `k` directly solves a different problem.
- **Using `while` instead of `if` in Approach 3** and expecting `max_count` to stay valid. Because you only added one character, the window is over budget by at most one, so a single `left` step restores it and the code stays $O(n)$.
- **Recomputing `max_count` from the 26 counters each step.** Legal, but $O(26n)$; the "never decrement" trick avoids it.

## The keystone

For a fixed target letter, "can this window be made uniform with `k` edits?" is `length - count(letter) <= k`, and the best letter to keep is the current majority. Tracking that majority count as the window slides — and never bothering to decrement it — is the specific move. "Compare a window against a frequency requirement" generalises to [Permutation in String](/citadel/dsa/permutation-in-string) and [Minimum Window Substring](/citadel/dsa/minimum-window-substring).
