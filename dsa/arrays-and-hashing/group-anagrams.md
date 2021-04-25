---
title: Group Anagrams - Picking a Canonical Key
description: Bucketing words so anagrams land together — the whole problem is choosing a dictionary key identical for anagrams and distinct otherwise, either sorted letters or a 26-count signature.
date: 2021-04-25
draft: false
slug: /dsa/group-anagrams
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Strings
---

The whole problem reduces to one design choice: a function mapping every string to a *key* such that two strings share a key exactly when they are anagrams. Then a single dictionary does the grouping.

## Description

Given a list of strings `strs`, group the strings that are anagrams of each other. The groups may be returned in any order.

**Example**

```
Input:  strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
Output: [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]

Input:  strs = [""]
Output: [[""]]
```

**Constraints**

- $1 \le \text{strs.length} \le 10^4$
- $0 \le \text{strs}[i]\text{.length} \le 100$
- `strs[i]` consists of lowercase English letters.

## Prerequisites

- [Valid Anagram](/citadel/dsa/valid-anagram) — the two-string version, and where the "canonical form" idea comes from.
- Hash maps with list values (`defaultdict(list)`), and why keys must be hashable (tuple, not list).

## Approach 1: Sorting

### Intuition

Sorting a word's letters produces a string identical for all of its anagrams: `"eat"`, `"tea"`, `"ate"` all sort to `"aet"`. Use that as the bucket key.

### Algorithm

1. Create a hash map from a sorted-letter key to a list of original strings.
2. For each string, sort its characters and join them into a key.
3. Append the original string to the list at that key.
4. Return the map's values.

```python
from collections import defaultdict

def groupAnagrams(strs: list[str]) -> list[list[str]]:
    buckets: dict[str, list[str]] = defaultdict(list)
    for s in strs:
        key = "".join(sorted(s))
        buckets[key].append(s)
    return list(buckets.values())
```

### Complexity

- **Time:** $O(m \cdot n \log n)$ — `m` strings, each costing an $n \log n$ sort where `n` is the longest length.
- **Space:** $O(m \cdot n)$ for the buckets.

## Approach 2: Hash Table

### Intuition

An anagram is fully described by *how many* of each letter it has, and counting is linear. Replace the $n \log n$ sort in the key with an $O(n)$ count vector.

### Algorithm

1. Create a hash map from a 26-length count tuple to a list of strings.
2. For each string, build a `[0] * 26` array and increment the slot for each character.
3. Convert the array to a tuple and use it as the key; append the string.
4. Return the map's values.

```python
from collections import defaultdict

def groupAnagrams(strs: list[str]) -> list[list[str]]:
    buckets: dict[tuple, list[str]] = defaultdict(list)
    for s in strs:
        count = [0] * 26
        for c in s:
            count[ord(c) - ord("a")] += 1
        buckets[tuple(count)].append(s)
    return list(buckets.values())
```

### Complexity

- **Time:** $O(m \cdot n)$ — one linear pass per string, plus $O(26)$ to hash the tuple.
- **Space:** $O(m)$ auxiliary for the keys; $O(m \cdot n)$ counting the output.

## Common Pitfalls

- **Using a list as a dict key.** `sorted(s)` and the raw count array are lists, which are unhashable. Join the sorted list into a string, or wrap the counts in `tuple(...)`.
- **`dict` instead of `defaultdict`.** `buckets[key].append(...)` raises `KeyError` on the first occurrence of each key; use `defaultdict(list)` or `setdefault`.
- **Approach 2 on a non-lowercase alphabet.** The fixed `[0] * 26` breaks; fall back to `tuple(sorted(Counter(s).items()))`.

## The keystone

Grouping "things equivalent under some relation" becomes a hashing problem the moment you can compute a *canonical form* per equivalence class — here, an anagram class's letter-count vector. That reframing replaces an $O(m^2)$ pairwise comparison with an $O(m)$ pass over keys, the same trick as [Two Sum](/citadel/dsa/two-sum).
