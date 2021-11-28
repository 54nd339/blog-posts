---
title: Letter Combinations of a Phone Number - A Cartesian Product by Recursion
description: Every string formed by choosing one letter per digit on an old phone keypad — a depth-per-digit recursion, or an iterative list that grows by a factor of three or four each step.
date: 2021-11-28
draft: false
slug: /dsa/letter-combinations-of-a-phone-number
tags:
  - Competitive Programming
  - Backtracking
  - String
---

Each digit maps to three or four letters. A combination picks one letter for digit 1, one for digit 2, and so on — the Cartesian product of the per-digit letter sets. There is no pruning here; every branch reaches a leaf.

## Description

Given a string `digits` containing digits `2`–`9`, return all letter combinations the number could spell, using the standard phone keypad mapping (`2` → `abc`, `3` → `def`, …, `7` → `pqrs`, `8` → `tuv`, `9` → `wxyz`). Return an empty list for an empty input.

**Example**

```
Input:  digits = "23"
Output: ["ad","ae","af","bd","be","bf","cd","ce","cf"]

Input:  digits = ""
Output: []
```

**Constraints**

- $0 \le \text{digits.length} \le 4$
- Each digit is in `2`–`9`.

## Prerequisites

- A fixed digit → letters map.
- The recursion tree where depth = number of digits, branching = letters for the current digit.

## Approach 1: Backtracking

### Intuition

Recurse on the digit index `i`. At depth `i`, loop over the letters of `digits[i]`, append one, recurse to `i + 1`, pop. When `i` equals the length, the accumulated string is one combination.

### Algorithm

1. If `digits` is empty, return `[]`.
2. `mapping = {"2": "abc", ..., "9": "wxyz"}`.
3. `dfs(i, path)`: if `i == len(digits)`, append `"".join(path)`; return.
4. For each `ch` in `mapping[digits[i]]`: append `ch`, `dfs(i + 1, path)`, pop.

```python
def letterCombinations(digits: str) -> list[str]:
    if not digits:
        return []
    mapping = {
        "2": "abc", "3": "def", "4": "ghi", "5": "jkl",
        "6": "mno", "7": "pqrs", "8": "tuv", "9": "wxyz",
    }
    res = []
    path = []

    def dfs(i):
        if i == len(digits):
            res.append("".join(path))
            return
        for ch in mapping[digits[i]]:
            path.append(ch)
            dfs(i + 1)
            path.pop()

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n \cdot 4^n)$ — at most $4^n$ combinations, each of length `n` to assemble.
- **Space:** $O(n)$ recursion depth, excluding output.

## Approach 2: Iteration (build the product level by level)

### Intuition

Keep a running list of partial strings. For each digit, replace the list with every partial string extended by every letter of that digit. The list grows by a factor of 3 or 4 per digit.

### Algorithm

1. If `digits` is empty, return `[]`.
2. `res = [""]`.
3. For each `d` in `digits`: `res = [prefix + ch for prefix in res for ch in mapping[d]]`.

```python
def letterCombinations(digits: str) -> list[str]:
    if not digits:
        return []
    mapping = {
        "2": "abc", "3": "def", "4": "ghi", "5": "jkl",
        "6": "mno", "7": "pqrs", "8": "tuv", "9": "wxyz",
    }
    res = [""]
    for d in digits:
        res = [prefix + ch for prefix in res for ch in mapping[d]]
    return res
```

### Complexity

- **Time:** $O(n \cdot 4^n)$.
- **Space:** $O(n \cdot 4^n)$ for the list being rebuilt each round.

## Common Pitfalls

- **Returning `[""]` for empty input.** The expected answer is `[]`; a `[""]` slips through if you skip the early return.
- **Off-by-one in the keypad map.** `7` and `9` have four letters, not three; hardcoding three-letter groups drops `s` and `z`.
- **Reusing a shared mutable `path` without popping.** Every append before recursing needs a matching pop after.
- **Joining at every node instead of the leaf.** Only build the string when `i == len(digits)`; joining mid-way wastes work.

## The keystone

This problem is the pure form of the backtracking tree — a full Cartesian product with no constraint to prune it, so brute force *is* optimal. Add a constraint that can fail a partial choice and you get [Combination Sum](/citadel/dsa/combination-sum) or [N-Queens](/citadel/dsa/n-queens), where the value of the recursion is precisely the branches it never takes.
