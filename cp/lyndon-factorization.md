---
title: Lyndon Factorization - Duval's Algorithm and Minimal Rotation
description: Every string factors uniquely into non-increasing Lyndon words, Duval's O(n) algorithm with O(1) extra space computes it, and one adaptation gives the lexicographically smallest rotation.
date: 2024-05-19
draft: false
slug: /cp/lyndon-factorization
tags:
  - Competitive Programming
  - String Processing
  - String Algorithms
---

A **Lyndon word** is a string strictly smaller than all of its proper rotations (equivalently, strictly smaller than every proper suffix of itself). The Chen-Fox-Lyndon theorem says every string factors *uniquely* into Lyndon words $w_1 w_2 \dots w_k$ with $w_1 \ge w_2 \ge \dots \ge w_k$. Duval's algorithm finds this factorization in $O(n)$ time and $O(1)$ extra space, and a small tweak yields the minimal cyclic rotation.

## The problem

Given $s$, output its Lyndon factorization — the unique list of non-increasing Lyndon words whose concatenation is $s$. Applications: the smallest string rotation, generating necklaces, and the bijective Burrows-Wheeler transform.

Example: $s = \texttt{"banana"}$ → `b | an | an | a` → factors `["b", "an", "an", "a"]` (each is a Lyndon word; `"b" >= "an" >= "an" >= "a"`).

## The idea (Duval)

Maintain a pointer $i$ to the start of the not-yet-factored suffix. From $i$, scan with two pointers:

- $j$ — the current position being read,
- $k$ — the position it is compared against (the start of the current "candidate period").

Cases while $j < n$:

- $s[k] = s[j]$ → the candidate repeats; advance both ($k{+}{+}, j{+}{+}$).
- $s[k] < s[j]$ → the block $s[i \dots j]$ is (a prefix of) a longer Lyndon word; reset $k = i$, advance $j$.
- $s[k] > s[j]$ → the Lyndon word of length $j - k$ ends here and repeats. Emit copies of $s[i \dots i + (j-k) - 1]$ for as long as they fit: while $i \le k$, output that block and do $i \mathrel{+}= j - k$. Then restart the scan from the new $i$.

Each character is visited $O(1)$ amortised times.

## How it works

$s = \texttt{"banana"}$.

- $i = 0$: $j = 1, k = 0$. $s[0]=\texttt{b} > s[1]=\texttt{a}$ → Lyndon length $j - k = 1$; emit `"b"`, $i = 1$.
- $i = 1$: $j = 2, k = 1$. $s[1]=\texttt{a} < s[2]=\texttt{n}$ → reset $k = 1$, $j = 3$. $s[1]=\texttt{a} < s[3]=\texttt{a}$? equal → $k=2, j=4$. $s[2]=\texttt{n} > s[4]=\texttt{a}$ → Lyndon length $j - k = 2$; emit `"an"`, $i = 3$; still $i \le k=2$? no. Restart.
- $i = 3$: same as from index $1$ on `"ana"` → emit `"an"`, $i = 5$.
- $i = 5$: single char → emit `"a"`.

Factors: `["b", "an", "an", "a"]`.

## Algorithm

```python
def duval(s: str) -> list[str]:
    n = len(s)
    i = 0
    factors = []
    while i < n:
        j, k = i + 1, i
        while j < n and s[k] <= s[j]:
            k = i if s[k] < s[j] else k + 1
            j += 1
        while i <= k:
            factors.append(s[i:i + j - k])
            i += j - k
    return factors
```

## Minimal cyclic rotation

The lexicographically smallest rotation of $s$: run Duval on $s + s$ but stop once $i \ge n$; the last factor started at or before position $n$ begins the minimal rotation. Concretely, track the largest $i$ produced that is $< n$ — that index into $s+s$ (mod $n$) is the start of the smallest rotation.

```python
def least_rotation(s: str) -> int:
    t = s + s
    n = len(s)
    i, ans = 0, 0
    while i < n:
        ans = i
        j, k = i + 1, i
        while j < len(t) and t[k] <= t[j]:
            k = i if t[k] < t[j] else k + 1
            j += 1
        while i <= k:
            i += j - k
    return ans % n
```

## Complexity

- **Time:** $O(n)$ — amortised $O(1)$ per character (the classic amortisation: each `while i <= k` iteration attributes $j - k$ progress to output).
- **Space:** $O(1)$ beyond the output list.

## Where it is used

- **Smallest rotation** (above) — also solvable by Booth's algorithm; Duval on $s + s$ is the common choice.
- **Generating all necklaces / Lyndon words up to length $n$** in lexicographic order (the FKM algorithm), used to build de Bruijn sequences.
- **Bijective Burrows-Wheeler transform**, which factors the input into Lyndon words first.
- **Runs / squares theory** — Lyndon roots underpin the linear-time "all runs" algorithms.

## Common pitfalls

- **Comparison direction.** `s[k] <= s[j]` continues the scan; `s[k] < s[j]` resets $k$ to $i$, `s[k] == s[j]` advances $k$. Swapping `<` and `<=` breaks uniqueness.
- **The emit loop `while i <= k`.** It outputs one full Lyndon block per iteration and may run several times (a Lyndon word repeated). Do not replace it with a single append.
- **Minimal rotation index.** It is the *last* factor-start below $n$, not the first; track and overwrite `ans` each outer iteration.
- **Equal characters.** A run of equal characters is a sequence of length-1 Lyndon words (`"aaa"` → `["a","a","a"]`), handled by the `k+1` branch reaching the `s[k] > s[j]` case at the end.
- **Empty string.** Return `[]`; the `while i < n` loop handles it.

## The keystone

Every string has a unique factorization into non-increasing Lyndon words, and Duval's two-pointer scan finds it in $O(n)$ time and $O(1)$ space by tracking the current candidate period and emitting a Lyndon block whenever a smaller character breaks it. Running it on $s + s$ gives the lexicographically minimal rotation.
