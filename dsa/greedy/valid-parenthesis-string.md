---
title: Valid Parenthesis String - Track a Range of Possible Open Counts
description: Whether a string with wildcards can be a balanced parenthesis sequence — carry the minimum and maximum number of unclosed open brackets, treating each star as any of open, close, or empty.
date: 2022-05-29
draft: false
slug: /dsa/valid-parenthesis-string
tags:
  - Competitive Programming
  - Greedy
  - Dynamic Programming
---

Each `*` could be `(`, `)`, or empty, so the count of unclosed `(` is not a single number but a range. Track `lo` (if every `*` is `)` or empty) and `hi` (if every `*` is `(`). Clamp `lo` at 0. The string is valid iff `hi` never dips below 0 and `lo` ends at 0.

## Description

Given a string `s` of `'('`, `')'` and `'*'`, return `true` if it can be interpreted as a valid parenthesis string, where `*` may stand for `(`, `)`, or the empty string.

**Example**

```
Input:  s = "(*))"
Output: true

Input:  s = "(*)"
Output: true

Input:  s = ")("
Output: false
```

**Constraints**

- $1 \le \text{s.length} \le 100$
- `s` contains only `'('`, `')'`, `'*'`.

## Prerequisites

- The open-count range `[lo, hi]` and how each character shifts it.
- Why clamping `lo` at 0 is safe (extra `)` interpretations are simply discarded).

## Approach 1: Greedy open-count range

### Intuition

`lo` = fewest possible unclosed `(`, `hi` = most. `'('` bumps both up. `')'` drops both. `'*'` drops `lo` and raises `hi`. If `hi < 0` at any point, even the most generous reading has an unmatched `)` — fail. Keep `lo >= 0`. Valid iff `lo == 0` at the end.

### Algorithm

1. `lo = hi = 0`.
2. For each `ch`: if `'('`, `lo += 1`, `hi += 1`. If `')'`, `lo -= 1`, `hi -= 1`. If `'*'`, `lo -= 1`, `hi += 1`.
3. If `hi < 0`, return `False`. Set `lo = max(lo, 0)`.
4. Return `lo == 0`.

```python
def checkValidString(s: str) -> bool:
    lo = hi = 0
    for ch in s:
        if ch == "(":
            lo += 1
            hi += 1
        elif ch == ")":
            lo -= 1
            hi -= 1
        else:
            lo -= 1
            hi += 1
        if hi < 0:
            return False
        lo = max(lo, 0)
    return lo == 0
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Two-pass balance check

### Intuition

Scan left to right treating every `*` as `(`; if the running balance ever goes negative, there are too many `)`. Scan right to left treating every `*` as `)`; if that balance goes negative, too many `(`. Passing both means a valid assignment exists.

### Algorithm

1. `bal = 0`; for `ch` left to right: `bal += 1` if `ch != ')'` else `bal -= 1`; if `bal < 0`, return `False`.
2. `bal = 0`; for `ch` right to left: `bal += 1` if `ch != '('` else `bal -= 1`; if `bal < 0`, return `False`.
3. Return `True`.

```python
def checkValidString(s: str) -> bool:
    bal = 0
    for ch in s:
        bal += 1 if ch != ")" else -1
        if bal < 0:
            return False

    bal = 0
    for ch in reversed(s):
        bal += 1 if ch != "(" else -1
        if bal < 0:
            return False

    return True
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 3: Dynamic programming

### Intuition

`dp[i][open]` = can `s[i:]` be valid with `open` unclosed `(` currently. Branch on the three meanings of `*`.

### Algorithm

1. `f(i, open)`: if `open < 0`, `False`. If `i == n`, `open == 0`.
2. `'('` → `f(i+1, open+1)`; `')'` → `f(i+1, open-1)`; `'*'` → any of `f(i+1, open+1)`, `f(i+1, open-1)`, `f(i+1, open)`.
3. Memoize on `(i, open)`.

```python
from functools import lru_cache

def checkValidString(s: str) -> bool:
    n = len(s)

    @lru_cache(None)
    def f(i, open_):
        if open_ < 0:
            return False
        if i == n:
            return open_ == 0
        if s[i] == "(":
            return f(i + 1, open_ + 1)
        if s[i] == ")":
            return f(i + 1, open_ - 1)
        return f(i + 1, open_ + 1) or f(i + 1, open_ - 1) or f(i + 1, open_)

    return f(0, 0)
```

### Complexity

- **Time:** $O(n^2)$.
- **Space:** $O(n^2)$.

## Common Pitfalls

- **Letting `lo` go negative.** Clamp it at 0 each step — those readings (an unmatched `)`) are just not pursued; `hi < 0` is the real failure.
- **Returning `hi == 0`.** Validity is `lo == 0` at the end; `hi` can be positive (some `*` chose to be `(` unnecessarily).
- **One-pass balance treating `*` as always `(` or always `)`.** You need *both* directions, or a case like `"(*"` (should be false — wait, `(*` → `*` empty leaves `(` unmatched) slips through one pass.
- **Forgetting `*` can be empty.** The `f(i+1, open)` branch and the `[lo, hi]` width both encode it.

## The keystone

When a symbol has several meanings, carry the *interval* of reachable states instead of enumerating assignments — `[lo, hi]` collapses an exponential search to one pass. This closes [Greedy](/citadel/dsa/greedy); the interval-of-state idea leads naturally into [Intervals](/citadel/dsa/intervals), where the objects themselves are ranges.
