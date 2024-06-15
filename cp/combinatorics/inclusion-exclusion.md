---
title: The Inclusion-Exclusion Principle - Counting by Overlap
description: Counting a union by alternately adding and subtracting intersections, the complementary "count the good ones" phrasing, and the Mobius-function form that speeds up divisibility-flavoured sums.
date: 2024-06-15
draft: false
slug: /cp/inclusion-exclusion
tags:
  - Competitive Programming
  - Combinatorics
  - Counting
---

To count elements with **at least one** of several properties, you cannot just add the counts — elements with two properties are counted twice. Inclusion-exclusion fixes it by alternately adding and subtracting the counts of intersections. The contest version is almost always the *complement*: count the elements with **none** of the bad properties.

## The problem

Given sets $A_1, \dots, A_m$ (each "elements having property $i$") inside a universe $U$, compute $|A_1 \cup \dots \cup A_m|$, or its complement $|U| - |A_1 \cup \dots \cup A_m|$ = "elements with no property".

Example: how many integers in $[1, 100]$ are divisible by $2$, $3$, or $5$? $|A_2| = 50$, $|A_3| = 33$, $|A_5| = 20$, $|A_2 \cap A_3| = 16$, $|A_2 \cap A_5| = 10$, $|A_3 \cap A_5| = 6$, $|A_2 \cap A_3 \cap A_5| = 3$. Union $= 50 + 33 + 20 - 16 - 10 - 6 + 3 = 74$.

## The idea

$$\left| \bigcup_{i=1}^{m} A_i \right| = \sum_{\emptyset \ne S \subseteq \{1..m\}} (-1)^{|S| + 1} \left| \bigcap_{i \in S} A_i \right|.$$

Equivalently, the count of elements in **none** of the $A_i$:

$$\left| U \setminus \bigcup A_i \right| = \sum_{S \subseteq \{1..m\}} (-1)^{|S|} \left| \bigcap_{i \in S} A_i \right|,$$

with the empty intersection being $|U|$. Every element in exactly $k$ of the sets contributes $\sum_{j=0}^{k} \binom{k}{j}(-1)^j = [k = 0]$ — so it is counted once iff it has no property, zero times otherwise.

## How it works

Divisible by $2$, $3$, or $5$ in $[1, 100]$, via the complement: not divisible by any is

$$100 - \tfrac{100}{2} - \tfrac{100}{3} - \tfrac{100}{5} + \tfrac{100}{6} + \tfrac{100}{10} + \tfrac{100}{15} - \tfrac{100}{30}$$
$$= 100 - 50 - 33 - 20 + 16 + 10 + 6 - 3 = 26.$$

So $100 - 26 = 74$ are divisible by at least one — matching the direct computation. (Floors: $\lfloor 100/3 \rfloor = 33$, etc.)

## Algorithm

The generic $2^m$ subset loop:

```python
def count_none(m: int, universe: int, intersection):
    """intersection(S) = size of the intersection of A_i for i in bitmask S (S=0 -> universe)."""
    total = 0
    for S in range(1 << m):
        sign = -1 if bin(S).count("1") & 1 else 1
        total += sign * intersection(S)
    return total
```

For the "divisible by a set of primes" case, `intersection(S)` is $\lfloor N / \prod_{i \in S} p_i \rfloor$, computed incrementally.

### Möbius form (divisibility problems)

When the properties are "divisible by $d$" over all $d \mid n$, inclusion-exclusion collapses into a sum against the **Möbius function** $\mu$:

$$\#\{x \le N : \gcd(x, n) = 1\} = \sum_{d \mid n} \mu(d) \left\lfloor \frac{N}{d} \right\rfloor,$$

where $\mu(d) = 0$ if $d$ has a squared prime factor, else $(-1)^{\#\text{distinct primes}}$. This is inclusion-exclusion over the *distinct prime factors* of $n$ — only $2^{\omega(n)}$ non-zero terms, and a [linear sieve](/citadel/cp/linear-sieve) tabulates $\mu$ for all $d \le 10^7$.

```python
def coprime_count(N: int, n: int, primes_of_n: list[int]) -> int:
    total = 0
    m = len(primes_of_n)
    for S in range(1 << m):
        prod = 1
        for i in range(m):
            if S >> i & 1:
                prod *= primes_of_n[i]
        sign = -1 if bin(S).count("1") & 1 else 1
        total += sign * (N // prod)
    return total
```

## Where it shows up

- **Coprime / gcd conditions** — count pairs with $\gcd = 1$, or $\gcd = g$ (scale down by $g$), via Möbius.
- **Surjections** — number of onto functions $[n] \to [k]$ is $\sum_{j=0}^{k} (-1)^j \binom{k}{j}(k - j)^n$.
- **Derangements** — $D_n = n! \sum_{j=0}^{n} \frac{(-1)^j}{j!}$, inclusion-exclusion over "position $i$ is a fixed point".
- **Forbidden positions / rook polynomials**, **permanent of a 0/1 matrix** (Ryser's formula is inclusion-exclusion), **counting integers avoiding digit patterns**.
- **At least $k$ of the properties** — a binomial-weighted variant.

## Common pitfalls

- **Sign convention.** For the union, non-empty subsets with sign $(-1)^{|S|+1}$. For "none of them", *all* subsets (including empty = universe) with sign $(-1)^{|S|}$. Pick one and be consistent.
- **$2^m$ blow-up.** Feasible only for $m \lesssim 20$–$25$. If $m$ is large but the properties are divisibility, switch to the Möbius form over $\omega(n) \le 9$ primes.
- **Intersections must be easy.** Inclusion-exclusion is only useful when $|\bigcap_{i \in S} A_i|$ has a formula. If intersections are as hard as the original problem, it does not help.
- **Overcounting the empty product.** $\prod_{i \in \emptyset} p_i = 1$, contributing $\lfloor N / 1 \rfloor = N$ with sign $+1$. Do not skip $S = 0$ in the "none" form.
- **$\mu$ of a non-squarefree number is $0$** — those subsets vanish, which is why the Möbius sum is short.

## The keystone

$|A_1 \cup \dots \cup A_m|$ alternates $+$ singles, $-$ pairs, $+$ triples, because an element in $k$ sets is counted $\sum \binom{k}{j}(-1)^{j+1} = 1$ times. In practice you count the complement ("none of the bad properties") over all $2^m$ subsets, and when the properties are divisibility you replace the loop with a short Möbius sum over distinct prime factors.
