---
title: Manacher's Algorithm - All Sub-Palindromes in O(n)
description: Computing the radius of the longest palindrome centred at every position in linear time by mirroring known radii inside the rightmost palindrome, and the "insert separators" trick that unifies odd and even lengths.
date: 2024-05-26
draft: false
slug: /cp/manachers-algorithm
tags:
  - Competitive Programming
  - String Processing
  - Palindromes
---

The [expand-around-centre](/citadel/dsa/longest-palindromic-substring) approach to palindromes is $O(n^2)$: try every centre, grow while it matches. Manacher's algorithm gets the palindrome radius at *every* centre in $O(n)$ total, by reusing radii already computed inside the currently-rightmost palindrome — the same "sliding window of known information" idea as the [Z-function](/citadel/cp/z-function).

## The problem

For each centre, find the radius of the longest palindrome centred there. From that array you get: the longest palindromic substring, the count of all palindromic substrings, and whether $s[l \dots r]$ is a palindrome in $O(1)$.

Handling odd and even lengths together: transform $s$ by inserting a separator (not in the alphabet) between every pair of characters and at the ends. `"abba"` → `"^#a#b#b#a#$"` (with distinct guards `^`, `$`). Now every palindrome of the original — odd or even — is an odd-length palindrome in the transformed string, centred at some index.

Example: `"abba"` transformed, the centre at the middle `#` has radius $4$, meaning `"abba"` is a palindrome.

## The idea

Let $p[i]$ be the palindrome radius at index $i$ of the transformed string $t$ (so $t[i-p[i] \dots i+p[i]]$ is a palindrome, and $p[i]$ in the transformed string equals the length of the corresponding original palindrome).

Track $(c, r)$ = the centre and right edge of the palindrome found so far that **extends furthest right**. For each $i \le r$:

- its mirror about $c$ is $j = 2c - i$. Initialise $p[i] = \min(r - i,\ p[j])$ — the mirror's radius, capped so it does not claim past $r$ (unverified territory).
- then **expand**: while $t[i - p[i] - 1] = t[i + p[i] + 1]$, increment $p[i]$.
- if $i + p[i] > r$, update $c = i$, $r = i + p[i]$.

The guards `^` and `$` at the ends never match, so the expansion loop stops without a bounds check. Every expansion step pushes $r$ right, so expansions total $O(n)$.

## How it works

$t = \texttt{"\^\#a\#b\#b\#a\#\$"}$ (indices $0 \dots 10$). Processing left to right, at the central `#` (index $5$) the mirror logic plus one expansion gives $p[5] = 4$: $t[1 \dots 9] = \texttt{"\#a\#b\#b\#a\#"}$ is a palindrome, i.e. the original `"abba"`. Elsewhere $p$ picks up `"a"` (radius $1$), `"bb"` (radius $2$), etc.

## Algorithm

```python
def manacher(s: str) -> list[int]:
    t = "^#" + "#".join(s) + "#$"
    n = len(t)
    p = [0] * n
    c = r = 0
    for i in range(1, n - 1):
        if i < r:
            p[i] = min(r - i, p[2 * c - i])
        while t[i - p[i] - 1] == t[i + p[i] + 1]:
            p[i] += 1
        if i + p[i] > r:
            c, r = i, i + p[i]
    return p                                   # p[i] = radius; also = length of the original palindrome at that centre

def longest_palindrome(s: str) -> str:
    p = manacher(s)
    length, centre = max((v, i) for i, v in enumerate(p))
    start = (centre - length) // 2             # map transformed centre back to s
    return s[start:start + length]

def count_palindromic_substrings(s: str) -> int:
    return sum((v + 1) // 2 for v in manacher(s))
```

The mapping back: a transformed index $i$ with radius $p[i]$ corresponds to the original substring starting at $(i - p[i]) // 2$ of length $p[i]$.

## What it answers

- **Longest palindromic substring:** $\max p[i]$ and its centre.
- **Number of palindromic substrings:** $\sum_i \lceil p[i] / 2 \rceil$ — each centre of radius $\rho$ contributes $\lceil \rho / 2 \rceil$ original palindromes... (equivalently $\sum (p[i]+1)//2$).
- **Is $s[l \dots r]$ a palindrome:** with the transform below, $s[k]$ sits at $t[2k + 2]$, so the centre of $s[l \dots r]$ is $t[\,l + r + 2\,]$; check $p[l + r + 2] \ge r - l + 1$.
- **Palindromic factorisation DP, palindromic tree cross-checks**, longest palindromic prefix/suffix.

## Complexity

- **Time:** $O(n)$ — the expansion loop only ever advances $r$, and $r \le 2n$.
- **Space:** $O(n)$ for $t$ and $p$.

## Common pitfalls

- **Missing the `min(r - i, p[mirror])` cap.** Without it you trust the mirror radius past the known boundary and can overshoot a mismatch.
- **Guard characters.** `^` and `$` (or any two distinct out-of-alphabet symbols) at the ends let the `while` loop run without index checks. If you skip them, add bounds tests.
- **Odd/even confusion.** The separator trick makes every original palindrome odd-length in $t$; do **not** also special-case even palindromes.
- **Index mapping.** Original start $= (i - p[i]) // 2$; length $= p[i]$. Get this wrong and `longest_palindrome` returns a shifted slice.
- **Counting formula.** Total palindromic substrings is $\sum \lceil p[i]/2 \rceil$; a common bug is $\sum p[i]$ (overcounts) or $\sum p[i]//2$ (undercounts odd centres).

## The keystone

Insert separators so every palindrome is odd-length, then compute each centre's radius in $O(n)$ by seeding it from its mirror inside the rightmost known palindrome and expanding only past that boundary. The radius array yields the longest palindrome, the count of all palindromic substrings, and $O(1)$ "is this range a palindrome" tests.
