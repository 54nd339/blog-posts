---
title: Chinese Remainder Theorem - Merging Congruences
description: Reconstructing an integer from its residues modulo pairwise-coprime numbers, the incremental two-at-a-time merge that also handles non-coprime moduli, and Garner's mixed-radix form for exact big results.
date: 2024-01-10
draft: false
slug: /cp/chinese-remainder-theorem
tags:
  - Competitive Programming
  - Algebra
  - Modular Arithmetic
---

If you know a number's remainder modulo $3$, modulo $5$, and modulo $7$, you know it modulo $105$. That is the Chinese Remainder Theorem: a system of congruences with coprime moduli has a unique solution modulo their product. It is how you dodge overflow (compute modulo several small primes, then reconstruct) and how you solve "smallest $x$ with these remainders" problems.

## The problem

Given congruences

$$x \equiv r_1 \pmod{m_1}, \quad x \equiv r_2 \pmod{m_2}, \quad \dots, \quad x \equiv r_k \pmod{m_k},$$

find the unique $x$ modulo $M = \operatorname{lcm}(m_1, \dots, m_k)$, or report that the system is inconsistent. When the $m_i$ are pairwise coprime, $M = \prod m_i$ and a solution always exists.

Example: $x \equiv 2 \pmod 3$, $x \equiv 3 \pmod 5$, $x \equiv 2 \pmod 7$. The answer is $x \equiv 23 \pmod{105}$.

## The idea: merge two at a time

Combining $x \equiv r_1 \pmod{m_1}$ and $x \equiv r_2 \pmod{m_2}$: write $x = r_1 + m_1 t$ and substitute into the second:

$$r_1 + m_1 t \equiv r_2 \pmod{m_2} \;\Longrightarrow\; m_1 t \equiv r_2 - r_1 \pmod{m_2}.$$

This is a [linear congruence](/citadel/cp/linear-congruence-equation) in $t$. Let $g = \gcd(m_1, m_2)$:

- if $g \nmid (r_2 - r_1)$, the system is **inconsistent** — stop;
- otherwise $t \equiv \dfrac{r_2 - r_1}{g} \cdot \left(\dfrac{m_1}{g}\right)^{-1} \pmod{\dfrac{m_2}{g}}$.

Back-substitute to get the merged congruence $x \equiv r_1 + m_1 t \pmod{\operatorname{lcm}(m_1, m_2)}$. Fold the list left to right and you have handled the general (not necessarily coprime) case for free.

## How it works

Merge $x \equiv 2 \pmod 3$ with $x \equiv 3 \pmod 5$: $3t \equiv 1 \pmod 5$, and $3^{-1} \equiv 2$, so $t \equiv 2 \pmod 5$. Then $x \equiv 2 + 3 \cdot 2 = 8 \pmod{15}$.

Merge $x \equiv 8 \pmod{15}$ with $x \equiv 2 \pmod 7$: $15 t \equiv -6 \pmod 7$, i.e. $t \equiv 1 \cdot (-6) \equiv 1 \pmod 7$ (since $15 \equiv 1$). Then $x \equiv 8 + 15 \cdot 1 = 23 \pmod{105}$.

## Algorithm

1. Start with $(r, m) = (0, 1)$.
2. For each $(r_i, m_i)$:
   - $g \leftarrow \gcd(m, m_i)$; if $(r_i - r) \bmod g \ne 0$, return "inconsistent".
   - $\text{lcm} \leftarrow m / g \cdot m_i$.
   - $t \leftarrow \big((r_i - r) / g\big) \cdot \operatorname{inv}(m/g,\ m_i/g) \bmod (m_i/g)$.
   - $r \leftarrow (r + m \cdot t) \bmod \text{lcm}$; $m \leftarrow \text{lcm}$.
3. Return $(r, m)$ — the solution $r$ modulo $m$.

```python
def crt(congruences: list[tuple[int, int]]):
    r, m = 0, 1
    for ri, mi in congruences:
        g, p, _ = ext_gcd(m, mi)
        if (ri - r) % g != 0:
            return None                       # inconsistent
        lcm = m // g * mi
        t = (ri - r) // g * p % (mi // g)
        r = (r + m * t) % lcm
        m = lcm
    return r % m, m

def ext_gcd(a, b):
    if b == 0:
        return a, 1, 0
    g, x1, y1 = ext_gcd(b, a % b)
    return g, y1, x1 - (a // b) * y1
```

## Garner's algorithm: mixed-radix reconstruction

When the moduli $p_1, \dots, p_k$ are fixed coprimes (typically small primes) and you want the **exact** integer $x < \prod p_i$ — possibly a big integer far past 64 bits — Garner's form is cleaner than repeated merging. Represent

$$x = a_1 + a_2 p_1 + a_3 p_1 p_2 + \dots + a_k p_1 p_2 \cdots p_{k-1},$$

and solve for the digits $a_j$ one at a time:

$$a_j = \left(\big(r_j - a_1 - a_2 p_1 - \dots\big) \cdot (p_1 p_2 \cdots p_{j-1})^{-1}\right) \bmod p_j.$$

```python
def garner(remainders: list[int], primes: list[int]) -> int:
    k = len(primes)
    # inv[i][j] = (p_i)^{-1} mod p_j, defined only for i < j
    inv = [[pow(primes[i], -1, primes[j]) if i < j else 0
            for j in range(k)] for i in range(k)]
    x = [0] * k
    for j in range(k):
        cur = remainders[j] % primes[j]
        for i in range(j):
            cur = (cur - x[i]) * inv[i][j] % primes[j]
        x[j] = cur % primes[j]
    result, mul = 0, 1
    for j in range(k):
        result += x[j] * mul
        mul *= primes[j]
    return result
```

The value of Garner's method: the digits $a_j$ are small (below $p_j$), so the only big-integer work is the final positional sum — good when the true answer genuinely needs arbitrary precision.

## Common pitfalls

- **Assuming coprimality.** The pairwise-coprime case always has a solution; the general case can be inconsistent. Always run the $g \mid (r_i - r)$ check.
- **Overflow in `m * t`.** With 64-bit moduli whose product exceeds $2^{63}$, `m * t` overflows in C++ — use `__int128` or Barrett/Montgomery. Python is safe.
- **Negative intermediate `(ri - r)`.** Keep the sign, divide exactly by $g$, then reduce; don't reduce before dividing.
- **Garner with non-coprime primes.** Garner's formula needs the moduli pairwise coprime. Repeated distinct primes are fine; a repeated prime is not.
- **Reporting range.** The solution is a residue class; "smallest positive" is `r` if `r > 0` else `r + m`.

## The keystone

CRT reconstructs a number from its residues by merging congruences two at a time, each merge being one [linear congruence](/citadel/cp/linear-congruence-equation) solve; the same loop handles non-coprime moduli and detects inconsistency. Computing modulo several primes and CRT-combining is the standard way to keep exact arithmetic — for instance in [NTT-based](/citadel/cp/operations-on-polynomials) multiplication — without a big-integer library.
