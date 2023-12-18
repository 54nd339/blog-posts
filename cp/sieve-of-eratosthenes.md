---
title: Sieve of Eratosthenes - Listing Primes up to N
description: The classic prime sieve, why it runs in O(n log log n), the standard constant-factor cuts (start at i squared, odds only), and the segmented version for ranges that do not fit in memory.
date: 2023-12-18
draft: false
slug: /cp/sieve-of-eratosthenes
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

To find every prime below $n$, you could test each number for primality separately. The sieve is faster because it works the other way round: take each prime and cross off its multiples. What is left standing is exactly the primes, and the whole thing costs barely more than $O(n)$.

## The problem

Given $n$ (say up to $10^7$, sometimes $10^8$ with care), produce a boolean array `is_prime[0..n]` or the list of all primes $\le n$. This is preprocessing: contest problems then want $\pi(n)$, prime factorizations, multiplicative-function tables, or "is $x$ prime" in $O(1)$.

## The idea

Start assuming every number $\ge 2$ is prime. Walk $i$ from $2$ upward. When you reach an $i$ still marked prime, it *is* prime (nothing below it divided it), so cross out $2i, 3i, 4i, \dots$ as composite. Every composite $c$ has a prime factor $p \le \sqrt c$, so $c$ gets crossed when $i = p$; nothing prime is ever crossed.

Two standard cuts:

- **Start crossing at $i^2$.** Any smaller multiple $ki$ with $k < i$ was already crossed when $i$ was $k$'s prime factor.
- **Stop the outer loop at $\sqrt n$.** Beyond that, $i^2 > n$ and there is nothing left to cross.

## Why it's O(n log log n)

The work is $\sum_{p \le n,\ p \text{ prime}} \frac{n}{p}$. The sum of reciprocals of primes up to $n$ grows like $\ln \ln n$ (Mertens' theorem), so the total is $n \ln \ln n$. For $n = 10^7$, $\ln \ln n \approx 2.8$ — effectively linear.

## How it works

$n = 30$. Mark $0, 1$ non-prime. $i = 2$: cross $4, 6, 8, \dots, 30$. $i = 3$: cross $9, 15, 21, 27$ (even multiples already gone). $i = 4$: already crossed, skip. $i = 5$: cross $25$. $i = 6 > \sqrt{30}$, stop. Survivors: $2, 3, 5, 7, 11, 13, 17, 19, 23, 29$.

## Algorithm

1. Allocate `is_prime = [True] * (n + 1)`; set indices `0` and `1` to `False`.
2. For `i` from `2` to `floor(sqrt(n))`:
   - If `is_prime[i]`, set `is_prime[j] = False` for `j` in `i*i, i*i + i, ..., n`.
3. The remaining `True` indices are the primes.

```python
def sieve(n: int) -> list[bool]:
    is_prime = bytearray([1]) * (n + 1)
    is_prime[0:2] = b"\x00\x00"
    i = 2
    while i * i <= n:
        if is_prime[i]:
            is_prime[i*i : n+1 : i] = b"\x00" * len(range(i*i, n+1, i))
        i += 1
    return is_prime

def primes_upto(n: int) -> list[int]:
    s = sieve(n)
    return [i for i in range(2, n + 1) if s[i]]
```

The slice assignment `is_prime[i*i::i] = ...` is what makes this fast in Python — it pushes the inner loop into C.

## Variations

- **Odds only.** Store only odd numbers, halving memory and roughly halving time. Index $k$ represents $2k + 1$.
- **Segmented sieve.** To sieve $[L, R]$ with $R$ up to $10^{12}$ but $R - L$ small: first sieve primes up to $\sqrt R$, then for each such prime cross its multiples within $[L, R]$ using a local boolean array of size $R - L + 1$. Time $O((R - L)\log\log R + \sqrt R)$.
- **Smallest prime factor.** Instead of a boolean, store `spf[j] = i` the first time `j` is crossed. Then factorizing any $x \le n$ is repeated `x //= spf[x]` in $O(\log x)$ — but the [linear sieve](/citadel/cp/linear-sieve) computes `spf` without over-writing and in true $O(n)$.
- **Multiplicative function tables.** With `spf`, tabulate Euler's [totient](/citadel/cp/eulers-totient-function), Möbius, divisor counts for all $j \le n$ in $O(n \log n)$ or better.

## Common pitfalls

- **Inner loop from $2i$ instead of $i^2$.** Correct but wastes time re-crossing; more importantly, starting at $2 \cdot i$ with `i` up to `n` (not $\sqrt n$) turns an $O(n \log\log n)$ sieve into something noticeably slower.
- **Memory.** A plain `[bool]` list in Python is ~28 bytes per element for small ints but a `bytearray` is 1 byte; for $n = 10^8$ you need the `bytearray` and probably the odds-only layout.
- **Off-by-one on the bound.** Size `n + 1` so index `n` exists; loop condition `i * i <= n`, not `< n`.
- **Segmented sieve and small primes.** When $L \le p^2$, don't cross $p$ itself. Start crossing at $\max(p^2,\ \lceil L/p \rceil \cdot p)$.

## The keystone

The sieve inverts primality testing: rather than ask "what divides $c$", it lets each prime announce its multiples, so every composite is struck exactly by its prime factors. That gives all primes below $n$ in near-linear time, and the same sweep — recording *which* prime did the striking — turns into an $O(\log x)$ factorization table, which the [linear sieve](/citadel/cp/linear-sieve) sharpens to $O(n)$.
