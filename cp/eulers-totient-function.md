---
title: Euler's Totient Function - Counting Coprime Residues
description: The count of integers up to n coprime to n, its product formula over distinct prime factors, a single-value computation, a sieve for all values up to n, and Euler's theorem for modular exponents.
date: 2023-12-22
draft: false
slug: /cp/eulers-totient-function
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

$\varphi(n)$ counts the integers in $[1, n]$ that share no factor with $n$. It shows up whenever you reduce an exponent modulo something, count fractions in lowest terms, or apply the Chinese Remainder Theorem. The function has a clean formula once you know the prime factorization.

## The problem

Compute $\varphi(n) = |\{k \in [1, n] : \gcd(k, n) = 1\}|$, either for one $n$ up to $10^{12}$ (needs a factorization) or for every $n$ up to $10^7$ (needs a sieve).

Values: $\varphi(1) = 1$, $\varphi(10) = 4$ (the coprimes are $1, 3, 7, 9$), $\varphi(p) = p - 1$ for prime $p$, $\varphi(12) = 4$.

## The idea

$\varphi$ is **multiplicative**: $\varphi(ab) = \varphi(a)\varphi(b)$ when $\gcd(a, b) = 1$. For a prime power, exactly the multiples of $p$ are non-coprime, and there are $p^{k-1}$ of them in $[1, p^k]$, so

$$\varphi(p^k) = p^k - p^{k-1} = p^k\left(1 - \tfrac{1}{p}\right).$$

Multiplying over the distinct primes dividing $n$:

$$\varphi(n) = n \prod_{p \mid n} \left(1 - \frac{1}{p}\right).$$

That product form is what you compute — you never need the exponents, only the set of distinct primes.

## How it works

$n = 12 = 2^2 \cdot 3$. Distinct primes $\{2, 3\}$:

$$\varphi(12) = 12 \left(1 - \tfrac12\right)\left(1 - \tfrac13\right) = 12 \cdot \tfrac12 \cdot \tfrac23 = 4.$$

The coprimes are $1, 5, 7, 11$ — four of them.

## Algorithm

### One value, by trial division

1. Set `result = n`.
2. For each prime $p$ dividing $n$ (found by trial division to $\sqrt n$): set `result -= result // p`, then divide all factors of $p$ out of $n$.
3. If $n > 1$ after the loop, one prime factor remains: `result -= result // n`.

```python
def phi(n: int) -> int:
    result = n
    p = 2
    while p * p <= n:
        if n % p == 0:
            while n % p == 0:
                n //= p
            result -= result // p
        p += 1
    if n > 1:
        result -= result // n
    return result
```

`result -= result // p` applies the factor $(1 - 1/p)$ with integer arithmetic — no fractions.

### All values up to n, by sieve

Modify the [Sieve of Eratosthenes](/citadel/cp/sieve-of-eratosthenes): initialise `phi[i] = i`, and when $p$ is found prime, for every multiple $m$ of $p$ do `phi[m] -= phi[m] // p`.

```python
def phi_sieve(n: int) -> list[int]:
    phi = list(range(n + 1))
    for p in range(2, n + 1):
        if phi[p] == p:                 # p is prime
            for m in range(p, n + 1, p):
                phi[m] -= phi[m] // p
    return phi
```

## Complexity

- **Single value:** $O(\sqrt n)$ by trial division (or $O(n^{1/4})$ with [Pollard's rho](/citadel/cp/integer-factorization) for the factorization).
- **Sieve:** $O(n \log \log n)$ time, $O(n)$ space.

## Euler's theorem and its uses

If $\gcd(a, n) = 1$ then $a^{\varphi(n)} \equiv 1 \pmod n$ — the generalisation of Fermat's little theorem to composite moduli. Consequences:

- **Reducing exponents:** $a^e \equiv a^{e \bmod \varphi(n)} \pmod n$ when $\gcd(a, n) = 1$. For the case $\gcd(a, n) \ne 1$, the safe rule is $a^e \equiv a^{\varphi(n) + (e \bmod \varphi(n))} \pmod n$ for $e \ge \log_2 n$.
- **Modular inverse:** $a^{-1} \equiv a^{\varphi(n) - 1} \pmod n$ — a route to the inverse that does not need $n$ prime (only $\gcd(a,n)=1$).
- **Divisor sum identity:** $\sum_{d \mid n} \varphi(d) = n$, useful for flipping sums over divisors.

## Common pitfalls

- **Applying $a^{e \bmod \varphi(n)}$ when $\gcd(a, n) \ne 1$.** It can be wrong; use the $\varphi(n) + (e \bmod \varphi(n))$ form for large $e$.
- **Multiplying by $(1 - 1/p)$ in floating point.** Rounds badly for large $n$. Use `result -= result // p` on integers.
- **Sieve test `phi[p] == p`.** This identifies primes because a prime's entry is untouched until its own pass. Don't reorder the update before the check.
- **$\varphi(1) = 1$.** The empty product; the trial-division code returns `1` correctly since the loop and the `n > 1` branch are both skipped.

## The keystone

$\varphi(n) = n \prod_{p \mid n}(1 - 1/p)$ follows from multiplicativity plus the prime-power count, so you compute it straight from the distinct prime factors — one value in $O(\sqrt n)$, or all values in a sieve pass. Euler's theorem $a^{\varphi(n)} \equiv 1$ then lets you shrink modular exponents and invert without primality, which the [modular inverse](/citadel/cp/modular-inverse) post leans on.
