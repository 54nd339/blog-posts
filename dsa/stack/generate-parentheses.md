---
title: Generate Parentheses - Prune Before You Build
description: Listing every valid combination of n pairs of parentheses — generating all binary strings and filtering is exponential waste, while backtracking that never lets closers overtake openers builds only valid strings.
date: 2021-07-13
draft: false
slug: /dsa/generate-parentheses
tags:
  - Competitive Programming
  - Stack
  - Backtracking
---

A string of parentheses is valid iff at every prefix the count of `)` never exceeds the count of `(`, and the totals are equal at the end. Build one character at a time and only take a step that keeps those invariants possible.

## Description

Given `n`, return all strings formed by `n` well-formed pairs of parentheses, in any order.

**Example**

```
Input:  n = 3
Output: ["((()))","(()())","(())()","()(())","()()()"]

Input:  n = 1
Output: ["()"]
```

**Constraints**

- $1 \le n \le 8$

The number of valid strings is the `n`-th Catalan number $C_n = \binom{2n}{n}/(n+1)$: `1, 2, 5, 14, 42, ...`.

## Prerequisites

- [Backtracking](/citadel/dsa/backtracking) — extend a partial solution along legal moves only.
- The parentheses balance as a stack depth.

## Approach 1: Brute Force

### Intuition

Generate all $2^{2n}$ strings of `(` and `)`, keep the valid ones.

### Algorithm

1. DFS over positions, appending `(` or `)` at each.
2. When a string reaches length `2n`, validate it with a balance counter (increment on `(`, decrement on `)`, reject if negative; accept if it ends at zero).

```python
def generateParenthesis(n: int) -> list[str]:
    res = []

    def valid(s: str) -> bool:
        bal = 0
        for c in s:
            bal += 1 if c == "(" else -1
            if bal < 0:
                return False
        return bal == 0

    def dfs(s: str) -> None:
        if len(s) == 2 * n:
            if valid(s):
                res.append(s)
            return
        dfs(s + "(")
        dfs(s + ")")

    dfs("")
    return res
```

### Complexity

- **Time:** $O(2^{2n} \cdot n)$ — $4^n$ strings, each validated in $O(n)$.
- **Space:** $O(2^{2n} \cdot n)$ including the output.

## Approach 2: Backtracking

### Intuition

Track openers and closers used. Place `(` only while `open < n`; place `)` only while `close < open`. Every leaf is valid, so nothing is filtered.

### Algorithm

1. `cur = []`, counters `open_count = close_count = 0`.
2. If `len(cur) == 2n`, record `"".join(cur)`.
3. If `open_count < n`: append `"("`, recurse with `open_count + 1`, pop.
4. If `close_count < open_count`: append `")"`, recurse with `close_count + 1`, pop.

```python
def generateParenthesis(n: int) -> list[str]:
    res = []
    cur = []

    def backtrack(open_count: int, close_count: int) -> None:
        if len(cur) == 2 * n:
            res.append("".join(cur))
            return
        if open_count < n:
            cur.append("(")
            backtrack(open_count + 1, close_count)
            cur.pop()
        if close_count < open_count:
            cur.append(")")
            backtrack(open_count, close_count + 1)
            cur.pop()

    backtrack(0, 0)
    return res
```

### Complexity

- **Time:** $O(4^n / \sqrt{n})$ — proportional to the number of valid strings.
- **Space:** $O(n)$ recursion depth plus the buffer.

## Approach 3: Dynamic Programming

### Intuition

Every valid string of `n` pairs is uniquely `( A ) B`, where `A` has `i` pairs and `B` has `n - 1 - i` pairs. Combine all such `A` and `B`.

### Algorithm

1. `dp[0] = [""]`.
2. For `k` from `1` to `n`: for each split `i` in `0 .. k - 1`, for each `a` in `dp[i]` and `b` in `dp[k - 1 - i]`, add `"(" + a + ")" + b` to `dp[k]`.
3. Return `dp[n]`.

```python
def generateParenthesis(n: int) -> list[str]:
    dp = [[""]] + [[] for _ in range(n)]
    for k in range(1, n + 1):
        for i in range(k):
            for a in dp[i]:
                for b in dp[k - 1 - i]:
                    dp[k].append(f"({a}){b}")
    return dp[n]
```

### Complexity

- **Time:** $O(4^n / \sqrt{n})$ — output-proportional.
- **Space:** $O(4^n / \sqrt{n})$ to store all `dp[k]`.

## Common Pitfalls

- **The closer condition is `close < open`, not `close < n`.** `close < n` re-admits invalid strings like `"())("`.
- **Forgetting `cur.pop()` after each recursive call (Approach 2).** The shared buffer must be restored for the sibling branch.
- **The DP split arithmetic.** `A` sits inside the first pair, `B` follows it; sizes add as `i + (n - 1 - i) = n - 1`.

## The keystone

When a brute force generates a huge space and filters it, look for a constraint you can check *incrementally* and let it prune the recursion tree — $O(4^n)$ becomes output-proportional. "Extend a partial solution only along legal moves" is exactly [Backtracking](/citadel/dsa/backtracking).
