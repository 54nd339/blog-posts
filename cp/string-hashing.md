---
title: String Hashing - Polynomial Hashes for O(1) Substring Comparison
description: Treating a string as a base-p number modulo m so a prefix-hash array gives any substring's hash in constant time, plus the collision math that dictates two moduli or a 64-bit random base.
date: 2024-05-09
draft: false
slug: /cp/string-hashing
tags:
  - Competitive Programming
  - String Processing
  - Hashing
---

Comparing two length-$\ell$ substrings character by character is $O(\ell)$. Polynomial hashing precomputes one array so that any substring's hash — and therefore an equality test between any two substrings — is $O(1)$. It is the Swiss-army knife of string problems, at the price of a (tiny, controllable) chance of a false match.

## The problem

Preprocess a string $s$ so that `hash(s[l..r])` is $O(1)$, then use it for: substring equality, longest common substring (binary search the length + a hash set), counting distinct substrings of a fixed length, palindrome tests, and [Rabin-Karp](/citadel/cp/rabin-karp) search.

## The idea

Map $s = s_0 s_1 \dots s_{n-1}$ to the number

$$H(s) = \sum_{i=0}^{n-1} s_i \cdot p^{\,i} \pmod m,$$

with $p$ a base larger than the alphabet (e.g. $p = 131$ or a random $\sim 10^9$ value) and $m$ a large prime ($\sim 10^{18}$, or two separate $\sim 10^9$ primes). Precompute prefix hashes $\text{pre}[k] = \sum_{i<k} s_i p^i \bmod m$ and powers $p^k \bmod m$. Then

$$H(s[l..r]) = \big(\text{pre}[r+1] - \text{pre}[l]\big) \cdot p^{-l} \bmod m,$$

or, to avoid the [modular inverse](/citadel/cp/modular-inverse), compare two substrings of equal length by scaling: $s[l_1..r_1] = s[l_2..r_2]$ iff

$$(\text{pre}[r_1{+}1] - \text{pre}[l_1]) \cdot p^{\,l_2} \equiv (\text{pre}[r_2{+}1] - \text{pre}[l_2]) \cdot p^{\,l_1} \pmod m.$$

## How it works

$s = \texttt{"ababa"}$, $p = 131$, $m$ large. $\text{pre}[0..5]$ accumulates $a\cdot p^0,\ +b\cdot p^1,\ +a\cdot p^2, \dots$. The substring $s[0..2] = \texttt{"aba"}$ has hash $(\text{pre}[3] - \text{pre}[0]) \cdot p^{0}$; $s[2..4] = \texttt{"aba"}$ has $(\text{pre}[5] - \text{pre}[2]) \cdot p^{-2}$. Both reduce to $a + b p + a p^2$, so they compare equal — correctly.

## Algorithm

```python
class Hashing:
    def __init__(self, s: str, mod: int = (1 << 61) - 1, base: int | None = None):
        import random
        self.mod = mod
        self.base = base or random.randrange(256, mod - 1)
        n = len(s)
        self.pw = [1] * (n + 1)
        self.pre = [0] * (n + 1)
        for i, ch in enumerate(s):
            self.pw[i + 1] = self.pw[i] * self.base % mod
            self.pre[i + 1] = (self.pre[i] + (ord(ch) + 1) * self.pw[i]) % mod

    def sub(self, l: int, r: int) -> int:            # hash of s[l..r] inclusive, unscaled
        return (self.pre[r + 1] - self.pre[l]) % self.mod

    def equal(self, l1: int, r1: int, l2: int, r2: int) -> bool:
        if r1 - l1 != r2 - l2:
            return False
        # scale both to a common power so no inverse is needed
        a = self.sub(l1, r1) * self.pw[l2] % self.mod
        b = self.sub(l2, r2) * self.pw[l1] % self.mod
        return a == b
```

Add `+ 1` to `ord(ch)` so a leading `'\0'`-valued character (or `'a'` = 0 in a shifted alphabet) does not make prefixes ambiguous.

## Collision probability and how to kill it

For a single modulus $m$ and $q$ pairwise comparisons, the chance of at least one false positive is roughly $q / m$ (birthday-style if you hash $k$ strings into a set: $\approx k^2 / (2m)$). Practical choices:

- **One 64-bit prime** $m = 2^{61} - 1$ with a random base — collision odds $\sim q \cdot 2^{-61}$, safe for almost everything, and multiplication fits in 128-bit (Python: free).
- **Two independent $\sim 10^9$ primes**, compare both — effective modulus $\sim 10^{18}$, standard for adversarial problems.
- **Randomised base**, chosen at runtime, so a problem-setter cannot precompute a colliding test.

Never use $m = 2^{64}$ (unsigned overflow) with a fixed base — there is a known family of anti-hash tests that force collisions.

## Complexity

- **Build:** $O(n)$.
- **Substring hash / equality:** $O(1)$.
- **Longest common substring of two strings:** binary search the length, hash all windows of that length in each string, intersect — $O((n + m)\log)$.

## Common pitfalls

- **Fixed base with $m = 2^{64}$.** Breakable. Randomise the base and use a prime modulus.
- **Forgetting to scale on comparison.** `sub(l1,r1) == sub(l2,r2)` is only valid when `l1 == l2`. Otherwise multiply by the missing power (as `equal` does) or by the inverse.
- **Zero-valued characters.** Map to `ord(ch) + 1` (or `ord(ch) - ord('a') + 1`) so `"a"`, `"aa"`, `"aaa"` do not all hash to $0$.
- **Single modulus in a hash set of many strings.** With $k \sim 10^5$ strings the $k^2/m$ term bites for $m \sim 10^9$; use the double hash or the 64-bit prime.
- **Overflow in `pre[i] + val * pw[i]`.** Fine in Python; in C++ with $m \approx 2^{61}$ you need `__int128` or careful `mulmod`.

## The keystone

A string is a base-$p$ number mod $m$; a prefix-hash array turns any substring's hash — and any substring equality test — into $O(1)$. Pick a random base and a 61-bit (or double $10^9$) modulus and the false-match probability is negligible, which is what makes hashing the default first attempt at most string problems.
