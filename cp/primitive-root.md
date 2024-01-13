---
title: Primitive Root - Generators of the Multiplicative Group
description: The element whose powers sweep out every unit modulo m, when one exists, how to find the smallest by checking prime-factor exponents of phi(m), and how it turns x to the k congruent a into a discrete log.
date: 2024-01-13
draft: false
slug: /cp/primitive-root
tags:
  - Competitive Programming
  - Algebra
  - Modular Arithmetic
---

A primitive root modulo $m$ is a number $g$ whose powers $g^1, g^2, \dots, g^{\varphi(m)}$ run through *every* residue coprime to $m$ exactly once. It exists only for special $m$, and when it does it turns multiplication modulo $m$ into addition modulo $\varphi(m)$ — which is what makes equations like $x^k \equiv a \pmod m$ solvable.

## The problem

Given $m$, decide whether a primitive root exists and, if so, return the smallest one. A primitive root exists **iff** $m \in \{1, 2, 4, p^k, 2p^k\}$ for an odd prime $p$. The number of primitive roots, when any exist, is $\varphi(\varphi(m))$.

Example: $m = 7$. $\varphi(7) = 6$. Test $g = 2$: powers are $2, 4, 1, 2, 4, 1$ — order $3$, not a generator. Test $g = 3$: $3, 2, 6, 4, 5, 1$ — all six units, so $3$ is the smallest primitive root mod $7$.

## The idea

$g$ is a primitive root iff its **multiplicative order** — the least $d > 0$ with $g^d \equiv 1$ — equals $\varphi(m)$. Checking every $d$ is too slow, but the order always divides $\varphi(m)$, so:

> $g$ has order $\varphi(m)$ **iff** $g^{\varphi(m)/q} \not\equiv 1 \pmod m$ for every prime $q \mid \varphi(m)$.

If some $g^{\varphi(m)/q} \equiv 1$, the order divides $\varphi(m)/q$ and $g$ fails. If none do, no proper divisor of $\varphi(m)$ can be the order, so it is exactly $\varphi(m)$.

Factorising $\varphi(m)$ once gives you a handful of prime factors $q$; then each candidate $g$ costs $O(\log \varphi(m))$ per factor via [binary exponentiation](/citadel/cp/binary-exponentiation). The smallest primitive root is empirically tiny (below $\sim 100$ for essentially all practical $m$), so trying $g = 2, 3, 4, \dots$ terminates fast.

## How it works

$m = 761$ (prime), $\varphi = 760 = 2^3 \cdot 5 \cdot 19$. Prime factors $\{2, 5, 19\}$, so the exponents to test are $760/2 = 380$, $760/5 = 152$, $760/19 = 40$.

- $g = 2$: need $2^{380}, 2^{152}, 2^{40} \not\equiv 1 \pmod{761}$. $2^{380} \equiv 1$ (it turns out $2$ has order $380$), so $2$ fails.
- $g = 3$: none of $3^{380}, 3^{152}, 3^{40}$ is $1$, so $3$ is a primitive root.

## Algorithm

1. Compute $\phi = \varphi(m)$ (see [Euler's totient](/citadel/cp/eulers-totient-function)).
2. Factor $\phi$ into distinct primes $q_1, \dots, q_t$.
3. For $g = 1, 2, \dots, m-1$ with $\gcd(g, m) = 1$:
   - if $g^{\phi / q_i} \not\equiv 1 \pmod m$ for all $i$, return $g$.
4. If none found, no primitive root exists.

```python
def primitive_root(m: int):
    if m == 1:
        return 0
    phi = euler_phi(m)
    factors = list(distinct_prime_factors(phi))
    for g in range(1, m):
        if gcd(g, m) != 1:
            continue
        if all(pow(g, phi // q, m) != 1 for q in factors):
            return g               # g == 1 only when m in {1, 2}
    return None
```

## Discrete root: solving x^k congruent a (mod m)

A primitive root converts $x^k \equiv a \pmod m$ into a linear congruence in the exponent. Let $g$ be a primitive root and write $x = g^y$, $a = g^t$ (the exponent $t$ is a [discrete log](/citadel/cp/discrete-logarithm), found by baby-step giant-step). Then

$$g^{ky} \equiv g^{t} \pmod m \;\Longleftrightarrow\; k y \equiv t \pmod{\varphi(m)}.$$

That is a [linear congruence](/citadel/cp/linear-congruence-equation): solvable iff $\gcd(k, \varphi(m)) \mid t$, giving $\gcd(k, \varphi(m))$ solutions for $y$ modulo $\varphi(m)$, each mapped back to $x = g^{y} \bmod m$.

```python
def discrete_root(k: int, a: int, m: int) -> list[int]:
    if a % m == 0:
        return [0]
    g = primitive_root(m)
    phi = euler_phi(m)
    t = bsgs(g, a % m, m)                       # discrete log of a
    ys = solve_congruence(k, t, phi)           # k*y == t (mod phi)
    return sorted({pow(g, y, m) for y in ys})
```

## Complexity

- **Finding a primitive root:** $O\!\big(\text{factor}(\varphi(m)) + g_{\min} \cdot t \cdot \log m\big)$, where $t$ is the number of distinct prime factors of $\varphi(m)$ and $g_{\min}$ is the (small) answer.
- **Discrete root:** dominated by the $O(\sqrt m)$ discrete-log call.

## Common pitfalls

- **Assuming every modulus has one.** Only $1, 2, 4, p^k, 2p^k$ do. Modulo $8$ there is no primitive root: the units $\{1, 3, 5, 7\}$ all square to $1$.
- **Testing all $d \mid \varphi(m)$ instead of $\varphi(m)/q$.** Only the maximal proper divisors $\varphi(m)/q$ (one per prime $q$) need checking — that is the whole speed-up.
- **Forgetting $\gcd(g, m) = 1$.** A non-unit cannot be a primitive root; skip it.
- **Discrete root when $\gcd(k, \varphi(m)) \nmid t$.** No solution — return empty, do not force it.
- **$a \equiv 0$.** Then $x \equiv 0$ is the only root (for $k \ge 1$); handle before taking a discrete log.

## The keystone

$g$ is a primitive root iff its order is $\varphi(m)$, and that reduces to checking $g^{\varphi(m)/q} \ne 1$ for each prime $q \mid \varphi(m)$ — a few fast exponentiations per candidate. Once you have $g$, every unit is a power of it, so $x^k \equiv a$ becomes the linear congruence $ky \equiv \log_g a \pmod{\varphi(m)}$.
