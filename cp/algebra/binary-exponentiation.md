---
title: Binary Exponentiation - Fast Powers by Squaring
description: Computing a to the n in O(log n) multiplications by reading the exponent in binary, with the modular and matrix versions that make it the most reused routine in contest algebra.
date: 2023-11-24
draft: false
slug: /cp/binary-exponentiation
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

You need $a^n \bmod m$ with $n$ up to $10^{18}$. Multiplying $a$ by itself $n$ times is $O(n)$ and hopeless. Binary exponentiation does it in about 60 multiplications, by noticing that squaring gets you to large powers fast and the binary digits of $n$ tell you which squares to keep.

## The problem

Given $a$, a non-negative integer $n$, and a modulus $m$, compute $a^n \bmod m$. The exponent is large enough that any loop running $n$ times is too slow, and $a^n$ itself has far too many digits to store, so the modulus has to be applied at every step.

Small example: $3^{13}$. Write $13 = 1101_2 = 8 + 4 + 1$. Then $3^{13} = 3^8 \cdot 3^4 \cdot 3^1$, and the powers $3^1, 3^2, 3^4, 3^8$ each come from squaring the previous one. Four squarings and two extra multiplications instead of twelve.

## The idea

Every exponent splits along its binary representation:

$$a^n = \prod_{i \,:\, \text{bit } i \text{ of } n \text{ is } 1} a^{2^i}.$$

The values $a^{2^0}, a^{2^1}, a^{2^2}, \dots$ form a chain where each term is the square of the one before. So walk the bits of $n$ from least significant to most: keep a running `base` that you square every step, and whenever the current bit is set, fold `base` into the `result`. There are $\lfloor \log_2 n \rfloor + 1$ bits, hence $O(\log n)$ multiplications.

Doing it modulo $m$ changes nothing structurally — you reduce after every multiplication so the operands never exceed $m^2$.

## How it works

Trace $3^{13} \bmod 1000$:

| step | bit of 13 | `result` before | `base` | `result` after |
| --- | --- | --- | --- | --- |
| 0 | 1 | 1 | 3 | 3 |
| 1 | 0 | 3 | 9 | 3 |
| 2 | 1 | 3 | 81 | 243 |
| 3 | 1 | 243 | 561 ($81^2 \bmod 1000$) | 243 · 561 mod 1000 = 323 |

$13$ has no more bits, so the answer is $323$. Check: $3^{13} = 1{,}594{,}323$, and $1{,}594{,}323 \bmod 1000 = 323$.

## Algorithm

1. Set `result = 1` and reduce `a` modulo `m`.
2. While `n > 0`:
   a. If `n` is odd, set `result = result * base % m`.
   b. Set `base = base * base % m`.
   c. Integer-divide `n` by 2 (drop the low bit).
3. Return `result`.

```python
def power(a: int, n: int, m: int) -> int:
    a %= m
    result = 1
    while n > 0:
        if n & 1:
            result = result * a % m
        a = a * a % m
        n >>= 1
    return result
```

Python has this built in as `pow(a, n, m)`; the hand-rolled version matters when you swap the scalar for a matrix or a polynomial.

## Complexity

- **Time:** $O(\log n)$ multiplications. Each modular multiply of numbers below $m$ is $O((\log m)^2)$ with schoolbook arithmetic, so $O(\log n \cdot (\log m)^2)$ overall.
- **Space:** $O(1)$.

## Variations

- **Modular inverse.** If $m$ is prime and $a$ is not a multiple of it, Fermat's little theorem gives $a^{-1} \equiv a^{m-2} \pmod m$ — one call to this routine. See [modular inverse](/citadel/cp/modular-inverse).
- **Matrix power.** Replace the scalar `*` with matrix multiplication and `1` with the identity matrix. This is how you get the $n$-th [Fibonacci number](/citadel/cp/fibonacci-numbers) or count length-$n$ walks in a graph in $O(k^3 \log n)$.
- **Multiplication modulo a value near $2^{63}$.** The intermediate product $a \cdot a$ overflows 64-bit integers; in C++ you need `__int128` or a `mulmod`. Python integers are unbounded, so this is a non-issue here.

## Common pitfalls

- **Exponent zero.** $a^0 = 1$ for every $a$, including $a = 0$ by the usual contest convention. Starting `result` at `1` handles it; don't special-case it wrong.
- **Negative exponent.** Only meaningful modulo $m$ when $a$ is invertible: compute $a^{-1}$ first, then raise it to $|n|$.
- **Forgetting to reduce `a` first.** If `a >= m` (or negative), skipping the initial `a %= m` still works in Python but bloats the operands; in fixed-width languages it can overflow immediately.
- **Non-prime modulus with Fermat.** $a^{m-2}$ is the inverse *only* when $m$ is prime. For composite $m$, use the [extended Euclidean](/citadel/cp/extended-euclidean) inverse.

## The keystone

Binary exponentiation is the observation that repeated squaring reaches $a^{2^k}$ in $k$ steps, and the binary digits of any exponent select which of those squares to multiply together. Every time a contest problem has a huge exponent, a linear recurrence, or the phrase "modulo a prime", this $O(\log n)$ loop is somewhere in the solution — often wrapped around a matrix, as the [Fibonacci](/citadel/cp/fibonacci-numbers) post shows.
