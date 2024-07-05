---
title: Balanced Bracket Sequences - Counting, Ranking, Unranking
description: The Catalan count of balanced sequences, the DP that also handles multiple bracket types, and the digit-by-digit walk that finds the k-th sequence lexicographically or the rank of a given one.
date: 2024-07-05
draft: false
slug: /cp/balanced-bracket-sequences
tags:
  - Competitive Programming
  - Combinatorics
  - Counting
---

Balanced bracket sequences of length $2n$ are counted by the [Catalan number](/citadel/cp/catalan-numbers) $C_n$. Beyond the count, contests ask for the **$k$-th** balanced sequence in lexicographic order, or the **rank** of a given one — both solved by a digit-by-digit walk using a small table of "how many completions from here".

## The problem

- **Count** balanced sequences of $2n$ single-type brackets: $C_n$.
- **Count with $t$ bracket types** (`()`, `[]`, `{}`, …), any properly-nested arrangement: $C_n \cdot t^n$ (choose the shape, then a type for each of the $n$ matched pairs).
- **Unrank:** the $k$-th balanced sequence in lexicographic order (`(` $<$ `)`).
- **Rank:** the lexicographic index of a given balanced sequence.

Example: $n = 3$, the $C_3 = 5$ sequences in order: `((()))`, `(()())`, `(())()`, `()(())`, `()()()`.

## The idea

Let $d[i][j]$ = the number of ways to complete a bracket string that currently has $i$ characters left to place and balance $j$ (opens minus closes so far, $j \ge 0$). Recurrence: the next character is `(` (balance $j+1$, one fewer to place) or `)` (balance $j-1$, only if $j > 0$):

$$d[i][j] = d[i-1][j+1] + [\,j > 0\,]\cdot d[i-1][j-1], \qquad d[0][0] = 1,\ d[0][j>0] = 0.$$

$d[2n][0] = C_n$.

- **Unrank $k$:** walk left to right. At each position with `rem` chars left and balance `bal`, the number of completions starting with `(` is $d[\text{rem}-1][\text{bal}+1]$. If $k <$ that, place `(`; else place `)` and subtract that many from $k$.
- **Rank:** the mirror — every time you place `)` where `(` was also legal, add $d[\text{rem}-1][\text{bal}+1]$ to the rank.

## Algorithm

```python
def bracket_table(n: int) -> list[list[int]]:
    N = 2 * n
    d = [[0] * (N + 2) for _ in range(N + 1)]
    d[0][0] = 1
    for i in range(1, N + 1):
        for j in range(N + 1):
            d[i][j] = d[i - 1][j + 1]
            if j > 0:
                d[i][j] += d[i - 1][j - 1]
    return d

def count(n: int, types: int = 1) -> int:
    return bracket_table(n)[2 * n][0] * types ** n

def unrank(n: int, k: int) -> str:
    d = bracket_table(n)
    res = []
    bal = 0
    for pos in range(2 * n):
        rem = 2 * n - pos - 1
        open_ways = d[rem][bal + 1]              # completions if we put '(' now
        if k < open_ways:
            res.append("(")
            bal += 1
        else:
            k -= open_ways
            res.append(")")
            bal -= 1
    return "".join(res)

def rank(s: str) -> int:
    n = len(s) // 2
    d = bracket_table(n)
    k = 0
    bal = 0
    for pos, ch in enumerate(s):
        rem = len(s) - pos - 1
        if ch == ")":
            if bal > 0:                          # '(' would also have been valid
                k += d[rem][bal + 1]
            bal -= 1
        else:
            bal += 1
    return k
```

## With multiple bracket types

For "$k$-th over all $C_n \cdot t^n$ sequences with $t$ types, ordered lexicographically by (shape, then type assignments)": first `unrank` the shape with $k \bmod C_n$ ... actually order matters — usually you fix a global order where each of the $2n$ positions is drawn from the alphabet `( ) [ ] { } ...` and only valid prefixes count. Then the walk considers each candidate opening bracket type (each contributes $t^{\text{pairs left}} \cdot d[\dots]$-style counts) and the single legal closing bracket (the one matching the top of the stack).

## Complexity

- **Table build:** $O(n^2)$.
- **Count / unrank / rank:** $O(n^2)$ for the table, then $O(n)$ for the walk. (The counts are Catalan-sized — take them modulo a prime if only the count is wanted, but for *unrank* you need exact big integers, so Python's `int` is doing real work.)

## Common pitfalls

- **Unrank needs exact integers.** The completion counts must be exact to compare against $k$; a modular table breaks the walk. Python handles the big integers; in C++ you need `__int128` / bignum or a bounded $k$.
- **Balance never negative.** `)` is only legal when `bal > 0`. The table already encodes this ($d[i][j]$ defined for $j \ge 0$); the walk must also guard it.
- **Off-by-one in `rem`.** After placing the character at `pos`, `rem = 2n - pos - 1` characters remain. Using `2n - pos` counts the current one twice.
- **Lexicographic order convention.** `(` $<$ `)`. If the problem orders `)` first, swap the branches.
- **Multiple types ordering.** Be explicit about whether the order is "all shapes, then vary types" or a single lexicographic order over the full bracket alphabet — they give different $k$-th sequences.

## The keystone

$d[i][j]$ = completions with $i$ characters left and balance $j$ is a small $O(n^2)$ table; balanced sequences number $d[2n][0] = C_n$. Walking left to right and comparing $k$ against $d[\text{rem}-1][\text{bal}+1]$ (the count of completions that start with `(`) unranks the $k$-th sequence, and accumulating those counts on each forced-looking `)` ranks a given one.
