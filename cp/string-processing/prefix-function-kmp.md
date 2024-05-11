---
title: Prefix Function - Knuth-Morris-Pratt Pattern Matching
description: The prefix function pi[i] as the longest proper border of every prefix, its O(n) computation by reusing previous values, and how "pattern plus separator plus text" turns it into linear substring search.
date: 2024-05-11
draft: false
slug: /cp/prefix-function-kmp
tags:
  - Competitive Programming
  - String Processing
  - String Matching
---

The prefix function of a string $s$ records, for every prefix, how much of it is also a suffix of itself — its longest **border**. That one array computed in $O(n)$ gives deterministic linear-time substring search (KMP), the period of a string, and the count of each prefix's occurrences.

## The problem

For $s$ of length $n$, compute $\pi[0 \dots n-1]$ where

$$\pi[i] = \max\{\,k < i+1 : s[0 \dots k-1] = s[i-k+1 \dots i]\,\}$$

— the length of the longest proper prefix of $s[0 \dots i]$ that is also a suffix of it ($\pi[0] = 0$).

Example: $s = \texttt{"abacaba"}$ → $\pi = [0, 0, 1, 0, 1, 2, 3]$. At $i = 6$, `"aba"` is both a prefix and a suffix, so $\pi[6] = 3$.

## The idea

Compute $\pi$ left to right, reusing what you have. Suppose $\pi[0 \dots i-1]$ are known and let $j = \pi[i-1]$ (length of the current best border).

- If $s[i] = s[j]$, the border extends: $\pi[i] = j + 1$.
- Otherwise fall back to the next candidate border length, which is $\pi[j-1]$ (the longest border of the border), and retry. Repeat until a match or $j = 0$.

Each character increments $j$ at most once, and every fallback strictly decreases $j$, so total work is $O(n)$ amortised.

## How it works

$s = \texttt{"abacaba"}$:

| $i$ | $s[i]$ | $j$ start | steps | $\pi[i]$ |
| --- | --- | --- | --- | --- |
| 1 | b | 0 | $s[1]\ne s[0]$ | 0 |
| 2 | a | 0 | $s[2]=s[0]$ | 1 |
| 3 | c | 1 | $s[3]\ne s[1]$, fall to $j=0$, $s[3]\ne s[0]$ | 0 |
| 4 | a | 0 | $s[4]=s[0]$ | 1 |
| 5 | b | 1 | $s[5]=s[1]$ | 2 |
| 6 | a | 2 | $s[6]=s[2]$ | 3 |

## Algorithm

```python
def prefix_function(s: str) -> list[int]:
    n = len(s)
    pi = [0] * n
    for i in range(1, n):
        j = pi[i - 1]
        while j > 0 and s[i] != s[j]:
            j = pi[j - 1]
        if s[i] == s[j]:
            j += 1
        pi[i] = j
    return pi

def kmp_search(text: str, pat: str) -> list[int]:
    if not pat:
        return list(range(len(text) + 1))
    pi = prefix_function(pat)
    res, j = [], 0
    for i, ch in enumerate(text):
        while j > 0 and ch != pat[j]:
            j = pi[j - 1]
        if ch == pat[j]:
            j += 1
        if j == len(pat):
            res.append(i - j + 1)
            j = pi[j - 1]
    return res
```

`kmp_search` runs the same recurrence with the pattern's $\pi$ against the text, without building `pat + sep + text` explicitly. The classic teaching version does build $\texttt{pat} + \texttt{\#} + \texttt{text}$ and reports every position where $\pi$ reaches $|\texttt{pat}|$.

## What else the prefix function gives

- **Period / smallest repeating block.** The shortest period of $s$ is $n - \pi[n-1]$; $s$ is a full repetition of it iff $n \bmod (n - \pi[n-1]) = 0$.
- **All borders of $s$.** They are $\pi[n-1],\ \pi[\pi[n-1]-1],\ \dots$ down to $0$.
- **Count of occurrences of each prefix** in $s$: `cnt[pi[i]-1] += 1` for all $i$, then propagate `cnt[pi[i]-1] += cnt[i]` for $i$ from $n-1$ down; add $1$ to every prefix for itself.
- **Number of distinct substrings**, incrementally as characters are appended (add $i + 1 - \max \pi$ over the reversed new prefix).
- **Automaton.** Precompute $\text{aut}[j][c]$ = the state after reading char $c$ in KMP state $j$ — an explicit DFA, useful for DP over "does the text contain the pattern".

## Complexity

- **Prefix function / KMP search:** $O(n)$ time, $O(n)$ space for $\pi$.
- **KMP automaton:** $O(m \cdot |\Sigma|)$ to build.

## Common pitfalls

- **Fallback target.** On a mismatch with border length $j$, the next candidate is $\pi[j-1]$, **not** $\pi[i-1] - 1$ or $j - 1$.
- **Resetting after a full match.** In `kmp_search`, after `j == len(pat)` set `j = pi[j-1]` to keep finding overlapping occurrences; setting `j = 0` misses them.
- **Separator character.** If you use the `pat + sep + text` trick, `sep` must not appear in either string.
- **`pi[0]`.** Always $0$ — a single character has no proper border. Start the loop at $i = 1$.
- **Confusing with the Z-function.** [Z](/citadel/cp/z-function) measures match-with-prefix *starting at* each position; $\pi$ measures border *ending at* each position. Convertible, but don't mix the recurrences.

## The keystone

$\pi[i]$ is the longest border of $s[0 \dots i]$, computed in $O(n)$ by extending the previous border on a match and falling back through $\pi[j-1]$ on a mismatch. Running that recurrence with a pattern's $\pi$ against a text is KMP — linear, deterministic substring search — and the same array yields a string's period and its border chain.
