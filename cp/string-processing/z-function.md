---
title: Z-Function - Longest Match From Every Position
description: z[i] as the length of the longest substring starting at i that matches a prefix of the string, computed in O(n) with a sliding [l, r] match window, and its use for pattern search and periods.
date: 2024-05-13
draft: false
slug: /cp/z-function
tags:
  - Competitive Programming
  - String Processing
  - String Matching
---

The Z-function of a string $s$ gives, for each position $i$, the length of the longest substring starting at $i$ that is also a **prefix** of $s$. It is computed in $O(n)$ by a clever reuse of a "rightmost match so far" window, and it does everything KMP's [prefix function](/citadel/cp/prefix-function-kmp) does, often with less fuss.

## The problem

For $s$ of length $n$, compute $z[0 \dots n-1]$ with

$$z[i] = \max\{\,k : s[0 \dots k-1] = s[i \dots i+k-1]\,\}, \qquad z[0] = 0 \text{ by convention}.$$

Example: $s = \texttt{"aabxaabxcaabxaabxay"}$ → $z[4] = 4$ (`"aabx"` matches the prefix), $z[8] = 0$, $z[9] = 8$ (`"aabxaabx"`).

## The idea

Maintain the interval $[l, r]$ that is the match with the prefix reaching **furthest right** among all found so far — meaning $s[l \dots r-1] = s[0 \dots r-l-1]$. For each new $i$:

- **If $i < r$:** position $i$ mirrors position $i - l$ inside the prefix. Start with $z[i] = \min(r - i,\ z[i - l])$ — the mirror value, capped so it does not run past $r$ (beyond $r$ we have no information yet).
- **Then extend naively:** while $i + z[i] < n$ and $s[z[i]] = s[i + z[i]]$, increment $z[i]$.
- **If $i + z[i] > r$:** update $l = i$, $r = i + z[i]$.

Every naive-extension step pushes $r$ right, and $r$ only moves forward, so the extensions total $O(n)$.

## How it works

$s = \texttt{"aaaaa"}$. $i = 1$: $i \ge r$ (window empty), extend naively: matches `"aaaa"`, $z[1] = 4$, set $[l, r] = [1, 5]$. $i = 2$: $i < r$, mirror $z[2 - 1] = z[1] = 4$, capped at $r - i = 3$, so start $z[2] = 3$; try to extend at $s[3]$ vs $s[5]$ — out of range, stop. $z[2] = 3$. $i = 3$: mirror $z[2] = 3$ capped at $r - i = 2$ → $z[3] = 2$. $i = 4$: → $z[4] = 1$. Result $z = [0, 4, 3, 2, 1]$.

## Algorithm

```python
def z_function(s: str) -> list[int]:
    n = len(s)
    z = [0] * n
    l = r = 0
    for i in range(1, n):
        if i < r:
            z[i] = min(r - i, z[i - l])
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        if i + z[i] > r:
            l, r = i, i + z[i]
    return z

def z_search(text: str, pat: str, sep: str = "\x00") -> list[int]:
    s = pat + sep + text
    z = z_function(s)
    m = len(pat)
    return [i - m - 1 for i in range(m + 1, len(s)) if z[i] >= m]
```

## What it gives

- **Pattern search:** build $\texttt{pat} + \texttt{sep} + \texttt{text}$; every position with $z \ge |\texttt{pat}|$ is a match. $sep$ must not occur in either string.
- **Number of distinct substrings**, added one character at a time.
- **String period:** $s$ has period $p$ iff $p + z[p] = n$ for some $p$ (i.e. the suffix from $p$ matches a prefix and reaches the end); the smallest such $p$ is the shortest period.
- **String compression / smallest generating block:** the shortest $t$ with $s = t^k$ corresponds to the smallest $p \mid n$ with $z[p] = n - p$.
- **Counting occurrences of each prefix**, like the prefix function's analogous trick.

## Complexity

- **Time:** $O(n)$ — the naive `while` only ever advances $r$, and $r \le n$.
- **Space:** $O(n)$ for $z$ (plus the concatenation in `z_search`).

## Z-function vs prefix function

They carry the same information (each is computable from the other in $O(n)$). Rules of thumb:

- **Z** is often more intuitive for "match with the prefix from here", pattern search, and period detection.
- **Prefix function** is the one you want for building the **KMP automaton** and for problems phrased around borders.
- Either works for plain substring search; pick the one you can write without bugs.

## Common pitfalls

- **`min(r - i, z[i - l])` cap.** Do not skip the cap — beyond $r$ the mirror is unverified, and using $z[i-l]$ raw can overshoot a real mismatch.
- **Updating $[l, r]$.** Only when $i + z[i] > r$, and then $l = i$, $r = i + z[i]$ (exclusive $r$). Off-by-one here breaks the whole scan.
- **$z[0]$.** Left at $0$ by convention (the whole string trivially matches itself; defining it as $n$ also works but be consistent).
- **Separator in `z_search`.** Must be a character absent from both strings; `"\x00"` or a sentinel outside the alphabet.
- **Index arithmetic in `z_search`.** A match at concatenation index $i$ is text position $i - m - 1$ (accounting for the separator).

## The keystone

$z[i]$ is the longest prefix-match starting at $i$, found in $O(n)$ by keeping the match window that reaches furthest right and seeding each new position from its mirror inside that window. It is interchangeable with the [prefix function](/citadel/cp/prefix-function-kmp) — use whichever phrasing (prefix-match-from-here vs border-ending-here) fits the problem.
