---
title: Valid Anagram - Counting Beats Sorting
description: Checking whether one string is a rearrangement of another — sorting both works, but comparing character counts is faster and a fixed 26-slot array is faster still.
date: 2021-04-18
draft: false
slug: /dsa/valid-anagram
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Strings
---

Two strings are anagrams exactly when every character appears the same number of times in both. Every approach here is a different way of checking that one fact.

## Description

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s` — the same characters with the same multiplicities, in some order.

**Example**

```
Input:  s = "anagram", t = "nagaram"
Output: true

Input:  s = "rat", t = "car"
Output: false
```

**Constraints**

- $1 \le \text{s.length}, \text{t.length} \le 5 \times 10^4$
- `s` and `t` consist of lowercase English letters.

## Prerequisites

- [Arrays and hashing](/citadel/dsa/arrays-and-hashing).
- Hash maps and `Counter` for frequency tables.
- The idea of a *canonical form*: two objects are equivalent iff they reduce to the same key.

## Approach 1: Sorting

### Intuition

Anagrams have the same multiset of characters, so sorting both strings produces identical sequences.

### Algorithm

1. If the lengths differ, return `false`.
2. Sort both strings.
3. Return whether the sorted results are equal.

```python
def isAnagram(s: str, t: str) -> bool:
    return len(s) == len(t) and sorted(s) == sorted(t)
```

### Complexity

- **Time:** $O(n \log n + m \log m)$ for the two sorts.
- **Space:** $O(n + m)$ — `sorted` returns a new list of characters for each string (Python cannot sort a `str` in place).

## Approach 2: Hash Map

### Intuition

An anagram is fully described by how many of each character it contains. Build a frequency map for each string and compare.

### Algorithm

1. If the lengths differ, return `false`.
2. Count characters of `s` into one map and characters of `t` into another.
3. Return whether the two maps are equal.

```python
from collections import Counter

def isAnagram(s: str, t: str) -> bool:
    return len(s) == len(t) and Counter(s) == Counter(t)
```

### Complexity

- **Time:** $O(n + m)$ to build both maps and compare.
- **Space:** $O(k)$ where `k` is the number of distinct characters — at most 26 here, so $O(1)$.

## Approach 3: Hash Table (Using Array)

### Intuition

With a fixed lowercase alphabet, the frequency map is just 26 integers. One array, incremented for `s` and decremented for `t`, must end all zeros.

### Algorithm

1. If the lengths differ, return `false`.
2. Create `count = [0] * 26`.
3. For each position, increment `count[s[i] - 'a']` and decrement `count[t[i] - 'a']`.
4. Return `true` if every entry of `count` is zero.

```python
def isAnagram(s: str, t: str) -> bool:
    if len(s) != len(t):
        return False
    count = [0] * 26
    for cs, ct in zip(s, t):
        count[ord(cs) - ord("a")] += 1
        count[ord(ct) - ord("a")] -= 1
    return all(c == 0 for c in count)
```

### Complexity

- **Time:** $O(n + m)$ — two constant-time updates per position.
- **Space:** $O(1)$ — 26 slots regardless of input size.

## Common Pitfalls

- **Skipping the length check.** With `Counter` equality it is technically unnecessary, but the single `zip` loop in Approach 3 *requires* it — without it, `zip` stops at the shorter string and `"a"` vs `"aa"` reports `true`.
- **Assuming Approach 1 is $O(1)$ space.** Each `sorted(...)` allocates a list of `n` characters.
- **`ord(c) - ord("a")` on non-lowercase input.** An uppercase letter or space yields a negative index, which in Python wraps around and silently corrupts a different counter rather than raising. For an arbitrary or Unicode alphabet, use Approach 2.

## The keystone

A frequency table *is* a canonical form for a multiset, and you build it in one pass — $O(n)$ versus sorting's $O(n \log n)$. That same table becomes a dictionary key in [Group Anagrams](/citadel/dsa/group-anagrams) and the tool of choice in [Top K Frequent Elements](/citadel/dsa/top-k-frequent-elements).
