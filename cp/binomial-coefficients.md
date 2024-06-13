---
title: Binomial Coefficients - Pascal, Factorial Tables, and Lucas
description: Computing n choose k many ways — Pascal's triangle for small n, precomputed factorials and inverse factorials modulo a prime for O(1) queries, Lucas for n beyond the modulus, and Legendre for prime powers.
date: 2024-06-13
draft: false
slug: /cp/binomial-coefficients
tags:
  - Competitive Programming
  - Combinatorics
  - Number Theory
---

$\binom{n}{k}$ — "$n$ choose $k$" — is the single most common quantity in counting problems, and contests always want it modulo a prime. The right method depends on the sizes: Pascal's triangle for tiny $n$, factorial + inverse-factorial tables for $n$ up to $\sim 10^7$, and Lucas' theorem when $n$ exceeds the modulus.

## The problem

Compute $\binom{n}{k} \bmod m$ (usually $m = 10^9 + 7$ or $998244353$, prime). Sub-cases: many queries with $n, k \le N$; a single query with astronomically large $n$; a prime-power modulus.

Recall $\binom{n}{k} = \dfrac{n!}{k!\,(n-k)!}$, $\binom{n}{k} = 0$ for $k < 0$ or $k > n$, and Pascal's rule $\binom{n}{k} = \binom{n-1}{k-1} + \binom{n-1}{k}$.

## Approach 1: Pascal's triangle

### When

$n \le \sim 5000$, or you need *every* $\binom{i}{j}$ for $i \le n$, or the modulus is composite/absent.

### Algorithm

```python
def pascal(n: int, mod: int | None = None):
    C = [[0] * (n + 1) for _ in range(n + 1)]
    for i in range(n + 1):
        C[i][0] = 1
        for j in range(1, i + 1):
            C[i][j] = C[i - 1][j - 1] + C[i - 1][j]
            if mod:
                C[i][j] %= mod
    return C
```

$O(n^2)$ time and space; no division, so any modulus works.

## Approach 2: Factorial and inverse-factorial tables (prime modulus)

### Intuition

Precompute $\text{fact}[i] = i! \bmod p$ and $\text{invfact}[i] = (i!)^{-1} \bmod p$. Then every query is

$$\binom{n}{k} \equiv \text{fact}[n] \cdot \text{invfact}[k] \cdot \text{invfact}[n-k] \pmod p$$

in $O(1)$. Build `invfact` in $O(n)$ by one [modular inverse](/citadel/cp/modular-inverse) of $n!$ and the recurrence $\text{invfact}[i-1] = \text{invfact}[i] \cdot i$.

### Algorithm

```python
MOD = 10 ** 9 + 7

def build_factorials(n: int):
    fact = [1] * (n + 1)
    for i in range(1, n + 1):
        fact[i] = fact[i - 1] * i % MOD
    invfact = [1] * (n + 1)
    invfact[n] = pow(fact[n], MOD - 2, MOD)
    for i in range(n, 0, -1):
        invfact[i - 1] = invfact[i] * i % MOD
    return fact, invfact

def C(n: int, k: int, fact, invfact) -> int:
    if k < 0 or k > n:
        return 0
    return fact[n] * invfact[k] % MOD * invfact[n - k] % MOD
```

### Complexity

$O(N)$ precompute, $O(1)$ per query.

## Approach 3: Lucas' theorem (n larger than p)

### Intuition

For prime $p$, write $n$ and $k$ in base $p$: $n = (n_t \dots n_1 n_0)_p$, $k = (k_t \dots k_1 k_0)_p$. Then

$$\binom{n}{k} \equiv \prod_{i=0}^{t} \binom{n_i}{k_i} \pmod p,$$

and each digit binomial has arguments $< p$, computable from a size-$p$ factorial table. If any $k_i > n_i$, the whole product is $0$.

### Algorithm

```python
def lucas(n: int, k: int, p: int, fact, invfact) -> int:
    result = 1
    while n or k:
        ni, ki = n % p, k % p
        if ki > ni:
            return 0
        result = result * (fact[ni] * invfact[ki] % p * invfact[ni - ki] % p) % p
        n //= p
        k //= p
    return result
```

### Complexity

$O(\log_p n)$ digit binomials per query, after an $O(p)$ factorial table. For a prime *power* modulus, use **Andrew Granville / Lucas-with-carries** or factor the modulus and CRT.

## Legendre's formula: the prime power in n!

The exponent of prime $p$ in $n!$ is

$$\nu_p(n!) = \sum_{i \ge 1} \left\lfloor \frac{n}{p^i} \right\rfloor = \frac{n - s_p(n)}{p - 1},$$

where $s_p(n)$ is the digit sum of $n$ in base $p$. Use it to find how many trailing zeros $n!$ has (base $10$: $\min(\nu_2, \nu_5) = \nu_5$), or whether $\binom{n}{k}$ is divisible by $p$ (**Kummer's theorem**: the exponent equals the number of carries when adding $k$ and $n-k$ in base $p$).

## Common pitfalls

- **`invfact` built the wrong direction.** Compute `invfact[n]` once by Fermat, then walk *down* with `invfact[i-1] = invfact[i] * i`. Building each independently is $O(n \log p)$ instead of $O(n)$.
- **Negative or out-of-range $k$.** Return $0$ for $k < 0$ or $k > n$ before indexing the tables.
- **Lucas with composite modulus.** Lucas needs $p$ prime. For $m = p^a$ or general $m$, factor and CRT, or use the generalised algorithm.
- **Pascal overflow.** Without a modulus, $\binom{60}{30} \approx 1.18 \times 10^{17}$ fits in 64 bits but $\binom{70}{35}$ does not — Python is fine, C++ needs care.
- **Modulus not prime for Approach 2.** The inverse of $k!$ may not exist. Use Pascal, or Lucas-style per-prime handling.

## The keystone

For repeated $\binom{n}{k} \bmod p$ queries, precompute factorials and inverse factorials in $O(n)$ and answer in $O(1)$; when $n$ outgrows $p$, Lucas' theorem multiplies the per-digit binomials in base $p$. Legendre's and Kummer's formulas handle divisibility and prime-power moduli.
