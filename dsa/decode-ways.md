---
title: Decode Ways - Count the Splits of a Digit String
description: Counting ways to decode a digit string into letters where A maps to 1 through Z maps to 26 — ways to decode up to position i is ways from a valid one-digit step plus a valid two-digit step.
date: 2022-03-21
draft: false
slug: /dsa/decode-ways
tags:
  - Competitive Programming
  - Dynamic Programming
  - Strings
---

To decode the prefix ending at index `i`, the last letter came from either one digit (`s[i]`, valid unless it is `'0'`) or two digits (`s[i-1:i+1]`, valid if between `"10"` and `"26"`). So `dp[i] = dp[i-1]·(one ok) + dp[i-2]·(two ok)` — a Fibonacci-shaped recurrence with validity gates.

## Description

A message of digits is encoded with `'A' -> "1"`, …, `'Z' -> "26"`. Given a digit string `s`, return the number of ways to decode it. `'0'` has no letter, so `"06"` cannot be decoded.

**Example**

```
Input:  s = "12"
Output: 2   ("AB" = 1 2, "L" = 12)

Input:  s = "226"
Output: 3   ("BZ", "VF", "BBF")

Input:  s = "06"
Output: 0
```

**Constraints**

- $1 \le \text{s.length} \le 100$
- `s` contains only digits and may have leading zeros.

## Prerequisites

- The two-predecessor recurrence with per-step validity checks.
- Careful handling of `'0'` and the `10..26` two-digit window.

## Approach 1: Recursion with memoization

### Intuition

`f(i)` = ways to decode `s[i:]`. If `i == len(s)`, one way (empty). If `s[i] == '0'`, zero ways. Otherwise `f(i) = f(i+1)`, plus `f(i+2)` when `s[i:i+2]` is in `10..26`.

### Algorithm

1. `f(i)`: `i == n` → `1`; `s[i] == '0'` → `0`.
2. `res = f(i + 1)`. If `i + 1 < n` and `10 <= int(s[i:i+2]) <= 26`, `res += f(i + 2)`.
3. Memoize on `i`.

```python
from functools import lru_cache

def numDecodings(s: str) -> int:
    n = len(s)

    @lru_cache(None)
    def f(i):
        if i == n:
            return 1
        if s[i] == "0":
            return 0
        res = f(i + 1)
        if i + 1 < n and 10 <= int(s[i:i + 2]) <= 26:
            res += f(i + 2)
        return res

    return f(0)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 2: Bottom-up array

### Intuition

`dp[i]` = ways to decode `s[i:]`, filled from the end. `dp[n] = 1`.

### Algorithm

1. `dp = [0] * (n + 1)`, `dp[n] = 1`.
2. For `i` from `n - 1` down to `0`: if `s[i] != '0'`, `dp[i] = dp[i+1]`; if `i + 1 < n` and `s[i:i+2]` in `10..26`, `dp[i] += dp[i+2]`.
3. Return `dp[0]`.

```python
def numDecodings(s: str) -> int:
    n = len(s)
    dp = [0] * (n + 1)
    dp[n] = 1
    for i in range(n - 1, -1, -1):
        if s[i] == "0":
            continue
        dp[i] = dp[i + 1]
        if i + 1 < n and 10 <= int(s[i:i + 2]) <= 26:
            dp[i] += dp[i + 2]
    return dp[0]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 3: Two rolling variables

### Intuition

`dp[i]` reads only `dp[i+1]` and `dp[i+2]`; keep those two.

### Algorithm

1. `after1 = 1` (`dp[n]`), `after2 = 0` (`dp[n+1]`).
2. For `i` from `n - 1` down to `0`: `cur = 0` if `s[i] == '0'` else `after1`; add `after2` if the two-digit window is valid. Shift `after2, after1 = after1, cur`.
3. Return `after1`.

```python
def numDecodings(s: str) -> int:
    n = len(s)
    after1, after2 = 1, 0
    for i in range(n - 1, -1, -1):
        cur = 0 if s[i] == "0" else after1
        if i + 1 < n and 10 <= int(s[i:i + 2]) <= 26:
            cur += after2
        after2, after1 = after1, cur
    return after1
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Treating `'0'` as decodable alone.** `'0'` has no letter; `dp[i]` is 0 unless a preceding `'1'` or `'2'` pairs with it.
- **Two-digit window bounds.** Valid only for `"10"`–`"26"`; `"27"`, `"09"`, `"30"` are not.
- **Leading zero.** `"0..."` decodes to 0 ways; the `s[i] == '0'` check at `i = 0` covers it.
- **Base case `dp[n] = 1`.** The empty suffix is one valid decoding; setting it to 0 zeroes everything.

## The keystone

A counting DP over string prefixes is Climbing Stairs with a gate on each transition — the recurrence shape is identical, the digits just decide which of the two terms are live. [Word Break](/citadel/dsa/word-break) is the same prefix DP where the gate is "is this slice a dictionary word".
