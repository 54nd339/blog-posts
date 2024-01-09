---
title: Number of Divisors and Sum of Divisors - Multiplicative Formulas
description: Reading the divisor count and divisor sum straight off a prime factorization, computing them for one number or sieving them for all numbers up to n, and the multiplicative structure that makes both formulas work.
date: 2024-01-09
draft: false
slug: /cp/divisors-count-and-sum
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

Two functions you reach for constantly: $d(n)$, how many positive divisors $n$ has, and $\sigma(n)$, their sum. Both come directly from the prime factorization, and both are multiplicative, so you can also sieve them for every $n$ up to $10^7$ in one pass.

## The problem

Given $n = p_1^{e_1} \cdots p_k^{e_k}$, compute

$$d(n) = \prod_{i=1}^{k} (e_i + 1), \qquad \sigma(n) = \prod_{i=1}^{k} \frac{p_i^{e_i + 1} - 1}{p_i - 1}.$$

Example: $n = 60 = 2^2 \cdot 3 \cdot 5$. Then $d(60) = 3 \cdot 2 \cdot 2 = 12$ and $\sigma(60) = 7 \cdot 4 \cdot 6 = 168$. The twelve divisors are $1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30, 60$; they sum to $168$.

## The idea

A divisor of $n$ picks an exponent $a_i \in [0, e_i]$ for each prime independently. That is $\prod (e_i + 1)$ choices — the divisor **count**.

For the **sum**, expand the product $\prod_i (1 + p_i + p_i^2 + \dots + p_i^{e_i})$: every term of the expansion is one divisor, so the whole product *is* $\sigma(n)$. Each factor is a geometric series summing to $(p_i^{e_i+1} - 1)/(p_i - 1)$.

Both are multiplicative because the choice per prime is independent: $d(ab) = d(a)d(b)$ and $\sigma(ab) = \sigma(a)\sigma(b)$ whenever $\gcd(a, b) = 1$.

## Algorithm

### One value, by trial division

```python
def divisor_count(n: int) -> int:
    count = 1
    p = 2
    while p * p <= n:
        if n % p == 0:
            e = 0
            while n % p == 0:
                n //= p
                e += 1
            count *= e + 1
        p += 1
    if n > 1:
        count *= 2                 # one leftover prime to the first power
    return count

def divisor_sum(n: int) -> int:
    total = 1
    p = 2
    while p * p <= n:
        if n % p == 0:
            term = 1
            pe = 1
            while n % p == 0:
                n //= p
                pe *= p
                term += pe          # 1 + p + p^2 + ... + p^e
            total *= term
        p += 1
    if n > 1:
        total *= 1 + n
    return total
```

### All values up to n, by sieve

The simplest divisor-count sieve: for each $i$ from $1$ to $n$, add $1$ to `d[j]` for every multiple $j$ of $i$. That is the harmonic sum $\sum n/i = O(n \log n)$.

```python
def divisor_count_sieve(n: int) -> list[int]:
    d = [0] * (n + 1)
    for i in range(1, n + 1):
        for j in range(i, n + 1, i):
            d[j] += 1
    return d

def divisor_sum_sieve(n: int) -> list[int]:
    s = [0] * (n + 1)
    for i in range(1, n + 1):
        for j in range(i, n + 1, i):
            s[j] += i
    return s
```

For a true $O(n)$ version, extend a [linear sieve](/citadel/cp/linear-sieve) that tracks, per number, the exponent of its smallest prime and the divisor function of the cofactor.

## Complexity

- **Single value:** $O(\sqrt n)$ by trial division (dominated by finding the factorization).
- **Harmonic sieve:** $O(n \log n)$ time, $O(n)$ space — fine to $n \approx 10^7$.
- **Linear sieve variant:** $O(n)$.

## Related bounds worth knowing

- $d(n)$ is $n^{o(1)}$ — for $n \le 10^{18}$ the maximum is $103{,}680$. Iterating all divisors of one number is cheap.
- To enumerate the divisors themselves, recurse over the prime powers, or trial-divide to $\sqrt n$ collecting $d$ and $n/d$ in pairs, then sort.
- $\sigma(n)$ relates to perfect numbers ($\sigma(n) = 2n$) and to the divisor-sum identity $\sum_{d \mid n} d = \sigma(n)$.

## Common pitfalls

- **Forgetting the leftover prime.** After trial division to $\sqrt n$, a value $n > 1$ is a prime factor with exponent $1$: multiply the count by $2$ and the sum by $1 + n$.
- **Overflow in $\sigma$.** For $n$ near $10^{18}$, $\sigma(n)$ can exceed $2^{63}$. Fine in Python; in C++ use unsigned or 128-bit, or take it modulo the required prime.
- **Geometric-series division.** $(p^{e+1} - 1)/(p - 1)$ is exact, but under a modulus you need the [modular inverse](/citadel/cp/modular-inverse) of $p - 1$ (or just accumulate $1 + p + \dots + p^e$ termwise, as the code does).
- **Counting $1$ and $n$.** Both are divisors; the formulas already include them. "Proper divisor" sums exclude $n$ — subtract it if that is what the problem wants.

## The keystone

A divisor is an independent exponent choice per prime, so $d(n) = \prod(e_i + 1)$ and $\sigma(n)$ is the expanded product of geometric series $\prod \frac{p_i^{e_i+1}-1}{p_i-1}$. Both are multiplicative, which is exactly what lets a [linear sieve](/citadel/cp/linear-sieve) fill them for every $n \le 10^7$ in one sweep.
