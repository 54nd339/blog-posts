---
title: Longest Substring Without Repeating Characters - A Forward-Only Window
description: The longest run of distinct characters — re-scanning from every start is quadratic, but a window with a last-seen-index map slides its left edge forward and never backtracks.
date: 2021-06-15
draft: false
slug: /dsa/longest-substring-without-repeating-characters
tags:
  - Competitive Programming
  - Sliding Window
  - Hash Table
---

Both pointers move only rightward. The right edge scans once; the left edge jumps forward whenever the new character would create a duplicate. Neither ever backtracks, so the whole thing is linear.

## Description

Given a string `s`, return the length of the longest substring with no repeated character.

**Example**

```
Input:  s = "abcabcbb"
Output: 3          # "abc"

Input:  s = "bbbbb"
Output: 1

Input:  s = "pwwkew"
Output: 3          # "wke"
```

**Constraints**

- $0 \le \text{s.length} \le 5 \times 10^4$
- `s` consists of English letters, digits, symbols, and spaces.

## Prerequisites

- [Sliding window](/citadel/dsa/sliding-window), variable size.
- Hash sets and hash maps for $O(1)$ membership and last-index lookup.

## Approach 1: Brute Force

### Intuition

From each start, extend right while characters stay distinct.

### Algorithm

1. `best = 0`.
2. For each start `i`: create an empty set; move `j` from `i` while `s[j]` is not in the set, adding each; record `j - i`.
3. Return the maximum.

```python
def lengthOfLongestSubstring(s: str) -> int:
    n = len(s)
    best = 0
    for i in range(n):
        seen = set()
        j = i
        while j < n and s[j] not in seen:
            seen.add(s[j])
            j += 1
        best = max(best, j - i)
    return best
```

### Complexity

- **Time:** $O(n \cdot m)$ where `m` is the alphabet size — each start can scan far.
- **Space:** $O(m)$ for the set.

## Approach 2: Sliding Window

### Intuition

Keep one window and one set alive across all starts. When the incoming character is already in the set, drop characters from the left until it is not.

### Algorithm

1. `seen = set()`, `left = 0`, `best = 0`.
2. For each `right`, character `c`:
   - While `c` is in `seen`: remove `s[left]`, `left += 1`.
   - Add `c`; update `best` with `right - left + 1`.
3. Return `best`.

```python
def lengthOfLongestSubstring(s: str) -> int:
    seen = set()
    left = 0
    best = 0
    for right, c in enumerate(s):
        while c in seen:
            seen.remove(s[left])
            left += 1
        seen.add(c)
        best = max(best, right - left + 1)
    return best
```

### Complexity

- **Time:** $O(n)$ — `left` and `right` each advance at most `n` times.
- **Space:** $O(m)$.

## Approach 3: Sliding Window (Optimal)

### Intuition

Instead of removing left characters one at a time, store each character's most recent index and jump the left edge straight past the previous occurrence.

### Algorithm

1. `last = {}` (character → last index), `left = 0`, `best = 0`.
2. For each `right`, character `c`:
   - If `c` is in `last` and `last[c] >= left`, set `left = last[c] + 1`.
   - Set `last[c] = right`; update `best` with `right - left + 1`.
3. Return `best`.

```python
def lengthOfLongestSubstring(s: str) -> int:
    last = {}
    left = 0
    best = 0
    for right, c in enumerate(s):
        if c in last and last[c] >= left:
            left = last[c] + 1
        last[c] = right
        best = max(best, right - left + 1)
    return best
```

### Complexity

- **Time:** $O(n)$ — a single pass.
- **Space:** $O(m)$.

## Common Pitfalls

- **Updating `best` mid-shrink (Approach 2).** While the `while` loop runs, the window still holds the duplicate; record only after it exits.
- **Letting the left edge move backward (Approach 3).** Without the `last[c] >= left` guard, a character last seen *before* the window shoves `left` back and re-admits duplicates. Use `left = max(left, last[c] + 1)`.
- **Not updating `last[c]` every step (Approach 3).** It must record the newest index even when `c` is outside the current window.

## The keystone

A variable-size window works when the property ("all distinct") is monotonic: once broken, only shrinking from the left fixes it, and the left edge advances monotonically for $O(n)$. Swapping "remove one at a time" for "jump using stored positions" is a constant-factor win that recurs in [Minimum Window Substring](/citadel/dsa/minimum-window-substring).
