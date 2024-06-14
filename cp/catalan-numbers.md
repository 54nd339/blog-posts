---
title: Catalan Numbers - Counting Balanced Structures
description: The sequence 1, 1, 2, 5, 14, 42 that counts balanced brackets, binary trees, triangulations and Dyck paths, its closed form and recurrences, and the reflection argument that proves the formula.
date: 2024-06-14
draft: false
slug: /cp/catalan-numbers
tags:
  - Competitive Programming
  - Combinatorics
  - Counting
---

The Catalan numbers $1, 1, 2, 5, 14, 42, 132, \dots$ are the answer to a startling number of "count the balanced / non-crossing / never-negative structures" problems. Recognising that a problem *is* Catalan turns it from a DP into a one-line formula.

## The problem

Compute $C_n \bmod p$, and — more usefully — recognise when a counting problem has answer $C_n$.

$C_n$ counts, among other things:

- balanced bracket sequences of length $2n$,
- full binary trees with $n + 1$ leaves (equivalently, binary trees with $n$ nodes),
- triangulations of a convex $(n + 2)$-gon,
- monotone lattice paths from $(0,0)$ to $(n,n)$ that never rise above the diagonal (**Dyck paths**),
- ways to fully parenthesise $n + 1$ factors,
- non-crossing perfect matchings of $2n$ points on a circle,
- rooted ordered forests / stack-sortable permutations of length $n$.

## The formulas

$$C_n = \frac{1}{n+1}\binom{2n}{n} = \binom{2n}{n} - \binom{2n}{n+1} = \prod_{k=2}^{n}\frac{n+k}{k}.$$

Recurrences:

$$C_0 = 1, \qquad C_{n+1} = \sum_{i=0}^{n} C_i\, C_{n-i} \quad\text{(convolution)}, \qquad C_{n+1} = \frac{2(2n+1)}{n+2}\, C_n \quad\text{(linear)}.$$

The convolution recurrence is the one to *recognise*: "the structure splits at a first return / a root into a left part of size $i$ and a right part of size $n - i$" is the Catalan signature.

## Why C_n = binom(2n, n) / (n+1) (reflection)

Count Dyck paths: $2n$ steps, $n$ up ($+1$) and $n$ down ($-1$), partial sums never negative. Total paths with $n$ ups and $n$ downs: $\binom{2n}{n}$. Count the **bad** ones (dip to $-1$ somewhere): reflect the path across the line $y = -1$ after its first touch of $-1$. This bijects bad paths with unrestricted paths of $n - 1$ ups and $n + 1$ downs, of which there are $\binom{2n}{n+1}$. So

$$C_n = \binom{2n}{n} - \binom{2n}{n+1} = \binom{2n}{n}\left(1 - \frac{n}{n+1}\right) = \frac{1}{n+1}\binom{2n}{n}.$$

## Algorithm

```python
MOD = 10 ** 9 + 7

def catalan_table(n: int, fact, invfact) -> list[int]:
    # fact/invfact from build_factorials(2n)   -- see the binomial-coefficients post
    C = [0] * (n + 1)
    for i in range(n + 1):
        binom = fact[2 * i] * invfact[i] % MOD * invfact[i] % MOD
        C[i] = binom * pow(i + 1, MOD - 2, MOD) % MOD
    return C

def catalan_linear(n: int) -> list[int]:
    C = [1] * (n + 1)
    for i in range(1, n + 1):
        # C[i] = C[i-1] * 2*(2i-1) / (i+1)
        C[i] = C[i - 1] * (2 * (2 * i - 1)) % MOD * pow(i + 1, MOD - 2, MOD) % MOD
    return C
```

For non-modular exact values, `catalan_linear` with integer division `C[i-1] * 2 * (2*i-1) // (i+1)` is exact.

## Generalisations

- **Ballot numbers / paths with a shifted floor:** paths from $(0,0)$ to $(a, b)$ staying weakly below the diagonal — the cycle lemma gives $\frac{b - a + 1}{b + 1}\binom{a+b}{a}$ (for $a \le b$).
- **$k$-ary trees / Fuss-Catalan:** $\frac{1}{kn + 1}\binom{kn + 1}{n}$ counts $k$-ary trees with $n$ internal nodes.
- **Narayana numbers** $N(n, k) = \frac{1}{n}\binom{n}{k}\binom{n}{k-1}$ refine $C_n$ by the number of peaks: $\sum_k N(n, k) = C_n$.
- **Motzkin, Schröder numbers** — related "non-crossing" families with different step sets.

## Common pitfalls

- **Off-by-one in "$n$".** $C_n$ counts brackets of length $2n$, polygons with $n + 2$ vertices, trees with $n$ internal nodes. Pin down which $n$ the problem's $n$ is.
- **Division by $n + 1$ under a modulus.** Use a [modular inverse](/citadel/cp/modular-inverse); `binom // (n+1)` only works in exact integer arithmetic.
- **Convolution recurrence cost.** $C_{n+1} = \sum C_i C_{n-i}$ is $O(n^2)$; use the linear recurrence or the formula for a single value, or [FFT](/citadel/algorithms/FastFourierTransform) if you need the whole self-convolution.
- **Assuming a structure is Catalan.** Verify the split: "first return to zero" or "the root" must cut it into two independent smaller instances of the *same* kind. Structures with crossings usually are not Catalan.
- **Overflow.** $C_{35} \approx 3.1 \times 10^{18}$ — the last one before 64-bit overflow.

## The keystone

$C_n = \frac{1}{n+1}\binom{2n}{n}$ counts every "balanced, non-crossing, never-negative" family, provable by reflecting the bad lattice paths. The tell that a problem is Catalan is the convolution split $C_{n+1} = \sum_i C_i C_{n-i}$ — a first return or a root that divides the structure into two smaller ones of the same type.
