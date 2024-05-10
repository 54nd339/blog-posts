---
title: Rabin-Karp - Substring Search by Rolling Hash
description: Sliding a fixed-width window across the text and updating its hash in O(1) per step, so every candidate position is a hash comparison, with a character check only on a match to kill collisions.
date: 2024-05-10
draft: false
slug: /cp/rabin-karp
tags:
  - Competitive Programming
  - String Processing
  - String Matching
---

To find every occurrence of a pattern $P$ (length $m$) in a text $T$ (length $n$), Rabin-Karp compares the hash of $P$ against the hash of each length-$m$ window of $T$. A **rolling hash** updates the window hash in $O(1)$ as it slides, so the whole scan is $O(n + m)$ expected — and a direct character check on each hash match makes it always correct.

## The problem

Report all starting positions $i$ with $T[i \dots i+m-1] = P$. Also handles the multi-pattern case when all patterns share one length, and 2-D pattern search.

Example: $T = \texttt{"abracadabra"}$, $P = \texttt{"abra"}$ → matches at positions $0$ and $7$.

## The idea

Use the [polynomial hash](/citadel/cp/string-hashing) $H(w) = \sum_j w_j p^{j} \bmod M$. If the window $w = T[i \dots i+m-1]$ has hash $h_i$, then sliding to $w' = T[i+1 \dots i+m]$:

$$h_{i+1} = \big(h_i - T[i]\big)\,p^{-1} + T[i+m]\,p^{m-1} \pmod M,$$

i.e. drop the outgoing character's term, shift everything down one power, add the incoming character at the top. (An equivalent "high-power-first" convention multiplies the running hash by $p$ and adds the new char, subtracting $T[i]\,p^{m}$.) Either way, $O(1)$ per step.

Compare $h_i$ to $H(P)$; on a hit, verify $T[i \dots i+m-1] = P$ character by character. The verify makes the algorithm deterministic; hashes only prune.

## How it works

$P = \texttt{"abra"}$. Precompute $H(P)$ and $p^{m-1}$. Window at $i = 0$ is `"abra"` → $h_0 = H(P)$ → verify → match. Slide: $i = 1$ window `"brac"`, $h_1$ computed in $O(1)$, $\ne H(P)$. … $i = 7$ window `"abra"` → $h_7 = H(P)$ → verify → match. Two character-level verifications total, one per real occurrence (plus zero spurious, since no collisions occurred).

## Algorithm

The clean form keeps the high power on the *first* character, so a slide is "subtract the leading term, multiply by the base, add the trailing character" — no modular inverse:

```python
def rabin_karp(text: str, pat: str, mod: int = (1 << 61) - 1, base: int = 131) -> list[int]:
    n, m = len(text), len(pat)
    if m == 0 or m > n:
        return []
    top = pow(base, m - 1, mod)

    def h(s):
        v = 0
        for ch in s:                            # high power ends up on the first char
            v = (v * base + ord(ch)) % mod
        return v

    target, cur = h(pat), h(text[:m])
    hits = []
    for i in range(n - m + 1):
        if cur == target and text[i:i + m] == pat:
            hits.append(i)
        if i + m < n:
            cur = (cur - ord(text[i]) * top) % mod        # drop leading char
            cur = (cur * base + ord(text[i + m])) % mod   # shift up, add trailing char
    return hits
```

## Complexity

- **Time:** $O(n + m)$ expected. Worst case $O(nm)$ only if hashes collide on almost every window — avoided by a random base and a large modulus.
- **Space:** $O(1)$ beyond the output.

## Variations

- **Multi-pattern, equal length.** Hash every pattern into a set; one text scan checks membership per window. Different lengths → group by length, or use [Aho-Corasick](/citadel/cp/aho-corasick).
- **2-D search.** Hash each row of the pattern; roll horizontally to get a per-column vector of row-hashes for the text; then treat that vector as a 1-D Rabin-Karp problem vertically. $O(nm)$ for an $n \times m$ text.
- **Anti-collision.** Two moduli, or one 61-bit prime with a random base, plus the character verify. With the verify, a collision only costs time, never correctness.

## Common pitfalls

- **Skipping the character verify.** Without it, one collision is a wrong answer. Keep it — it is $O(m)$ only on a hash hit.
- **Fixed base, $M = 2^{64}$.** Known anti-hash tests. Random base + prime modulus.
- **Rolling update sign.** After `cur - ord(text[i]) * top`, `cur` can go negative; `% mod` in Python fixes it, C++ needs `+ mod`.
- **`m > n` or `m == 0`.** Return early; the window loop assumes $1 \le m \le n$.
- **Recomputing `pow(base, ...)` each step.** Precompute `top = base^(m-1)` once; do not call `pow` inside the loop.

## The keystone

Rabin-Karp is "hash the pattern once, roll the text window's hash in $O(1)$, compare, verify on a hit." The rolling hash is what makes it linear; the character verify is what makes it exact. It generalises cleanly to many equal-length patterns and to 2-D grids.
