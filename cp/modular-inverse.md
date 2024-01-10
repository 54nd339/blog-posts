---
title: Modular Inverse - Division Under a Modulus
description: The number that plays the role of 1/a modulo m — when it exists, the two ways to compute one (extended Euclid for any modulus, Fermat for a prime), and the linear-time table of all inverses up to n.
date: 2024-01-10
draft: false
slug: /cp/modular-inverse
tags:
  - Competitive Programming
  - Algebra
  - Modular Arithmetic
---

Contest answers are asked "modulo $10^9 + 7$", and the arithmetic you do to get there often includes division — a binomial coefficient, an average, a probability. You cannot divide in $\mathbb{Z}/m\mathbb{Z}$ directly, but you can multiply by a **modular inverse**: the element $a^{-1}$ with $a \cdot a^{-1} \equiv 1 \pmod m$.

## The problem

Given $a$ and $m$, find $x \in [0, m)$ with $a x \equiv 1 \pmod m$, or report that none exists.

An inverse exists **iff $\gcd(a, m) = 1$**. Example: modulo $7$, the inverse of $3$ is $5$, because $3 \cdot 5 = 15 \equiv 1$. Modulo $10$, the number $4$ has no inverse ($\gcd(4, 10) = 2$).

## Approach 1: Extended Euclid (any modulus)

### Intuition

$a x \equiv 1 \pmod m$ means $a x + m y = 1$ for some integer $y$ — a [linear Diophantine](/citadel/cp/linear-diophantine-equations) equation. [Extended Euclid](/citadel/cp/extended-euclidean) solves $a x + m y = \gcd(a, m)$; when that GCD is $1$, its $x$ is the inverse.

### Algorithm

1. Run `ext_gcd(a, m)` → $(g, x, y)$.
2. If $g \ne 1$, no inverse.
3. Return `x % m` (normalised into $[0, m)$).

```python
def inv(a: int, m: int) -> int:
    g, x, _ = ext_gcd(a % m, m)
    if g != 1:
        raise ValueError("no inverse")
    return x % m

def ext_gcd(a: int, b: int):
    if b == 0:
        return a, 1, 0
    g, x1, y1 = ext_gcd(b, a % b)
    return g, y1, x1 - (a // b) * y1
```

### Complexity

$O(\log m)$. Works for **composite** $m$, which is its whole advantage.

## Approach 2: Fermat's little theorem (prime modulus)

### Intuition

If $m$ is prime and $a \not\equiv 0$, then $a^{m-1} \equiv 1 \pmod m$, so $a^{-1} \equiv a^{m-2} \pmod m$. One [binary exponentiation](/citadel/cp/binary-exponentiation).

```python
def inv_prime(a: int, m: int) -> int:
    return pow(a, m - 2, m)          # m must be prime, a % m != 0
```

For a prime power $m = p^k$, the same idea with [Euler's theorem](/citadel/cp/eulers-totient-function): $a^{-1} \equiv a^{\varphi(m) - 1}$.

### Complexity

$O(\log m)$, same as extended Euclid, but shorter to write when $m$ is a known prime.

## Approach 3: All inverses 1..n modulo a prime

### Intuition

There is an $O(n)$ recurrence. Write $m = q \cdot i + r$ with $q = \lfloor m/i \rfloor$, $r = m \bmod i$. Reducing mod $m$: $q i + r \equiv 0$, so $i^{-1} \equiv -q \cdot r^{-1} \pmod m$, and $r < i$ so $r^{-1}$ is already known.

### Algorithm

```python
def inverses_upto(n: int, m: int) -> list[int]:
    inv = [0] * (n + 1)
    inv[1] = 1
    for i in range(2, n + 1):
        inv[i] = (m - (m // i) * inv[m % i] % m) % m
    return inv
```

### Complexity

$O(n)$ for the whole table — the way to get $1/1!, 1/2!, \dots$ for binomial coefficients without $n$ separate exponentiations. (In practice one often computes $ (n!)^{-1}$ once and walks down with `invfact[i-1] = invfact[i] * i`, also $O(n)$.)

## Common pitfalls

- **Assuming an inverse exists.** Check $\gcd(a, m) = 1$ first. Under a prime modulus, also check $a \bmod m \ne 0$.
- **Fermat with a composite modulus.** $a^{m-2}$ is meaningless as an inverse unless $m$ is prime. Use extended Euclid.
- **Negative result.** `ext_gcd` can return a negative $x$; always finish with `% m`.
- **`pow(a, -1, m)` availability.** Python 3.8+ supports `pow(a, -1, m)` directly for coprime $a, m$ — clean, but know what it is doing.
- **Dividing inside a modular formula.** Replace every "$/k$" with "$\cdot k^{-1}$". Forgetting one gives a wrong-but-plausible number.

## The keystone

A modular inverse is the solution $x$ of $a x + m y = 1$, so it exists exactly when $\gcd(a, m) = 1$: [extended Euclid](/citadel/cp/extended-euclidean) finds it for any modulus in $O(\log m)$, and Fermat's $a^{m-2}$ is the one-liner when $m$ is prime. Precomputing the table of inverses (or inverse factorials) in $O(n)$ is what makes [binomial coefficients](/citadel/cp/binomial-coefficients) modulo a prime an $O(1)$ lookup.
