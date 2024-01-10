---
title: Linear Congruence Equation - Solving ax congruent b mod n
description: When ax equivalent to b mod n has solutions, how many there are, and how to list them — the gcd divisibility test, dividing the congruence through, and one modular inverse.
date: 2024-01-10
draft: false
slug: /cp/linear-congruence-equation
tags:
  - Competitive Programming
  - Algebra
  - Modular Arithmetic
---

The one-variable congruence $a x \equiv b \pmod n$ is the modular cousin of a [linear Diophantine equation](/citadel/cp/linear-diophantine-equations). It has either no solutions or exactly $\gcd(a, n)$ of them modulo $n$, and finding them is one divisibility check plus one [modular inverse](/citadel/cp/modular-inverse).

## The problem

Given $a$, $b$, $n$, find all $x \in [0, n)$ with $a x \equiv b \pmod n$.

Example: $6x \equiv 8 \pmod{10}$. Here $g = \gcd(6, 10) = 2$ and $2 \mid 8$, so there are $2$ solutions: $x \equiv 3$ and $x \equiv 8 \pmod{10}$ (check: $18 \equiv 8$, $48 \equiv 8$).

## The idea

Let $g = \gcd(a, n)$.

- **If $g \nmid b$:** no solution. $a x$ is always a multiple of $g$, so $a x \bmod n$ can only land on multiples of $g$.
- **If $g \mid b$:** divide the whole congruence by $g$:

$$\frac{a}{g}\, x \equiv \frac{b}{g} \pmod{\frac{n}{g}}.$$

Now $\gcd(a/g,\ n/g) = 1$, so $a/g$ is invertible modulo $n/g$, and

$$x \equiv \frac{b}{g} \cdot \left(\frac{a}{g}\right)^{-1} \pmod{\frac{n}{g}}.$$

That is one residue modulo $n/g$. Lifting back to modulus $n$, it splits into $g$ residues: $x_0,\ x_0 + \tfrac{n}{g},\ x_0 + \tfrac{2n}{g},\ \dots,\ x_0 + \tfrac{(g-1)n}{g}$.

## How it works

$6x \equiv 8 \pmod{10}$, $g = 2$, $2 \mid 8$. Divide through: $3x \equiv 4 \pmod 5$. The inverse of $3$ mod $5$ is $2$, so $x \equiv 4 \cdot 2 = 8 \equiv 3 \pmod 5$. Lift to modulus $10$: $x \equiv 3$ and $x \equiv 3 + 5 = 8 \pmod{10}$.

## Algorithm

1. $g \leftarrow \gcd(a, n)$.
2. If $b \bmod g \ne 0$: return "no solution".
3. $a' \leftarrow a/g$, $b' \leftarrow b/g$, $n' \leftarrow n/g$.
4. $x_0 \leftarrow (b' \cdot \operatorname{inv}(a', n')) \bmod n'$.
5. Return $\{\, x_0 + k \cdot n' : k = 0, 1, \dots, g - 1 \,\}$.

```python
def solve_congruence(a: int, b: int, n: int) -> list[int]:
    a %= n
    b %= n
    g = gcd(a, n)
    if b % g != 0:
        return []
    a_, b_, n_ = a // g, b // g, n // g
    x0 = b_ * inv(a_, n_) % n_
    return [(x0 + k * n_) % n for k in range(g)]
```

using the [`inv`](/citadel/cp/modular-inverse) and `gcd` from the earlier posts.

## Complexity

- **Time:** $O(\log n)$ for the GCD and the inverse, then $O(g)$ to list the solutions.
- **Space:** $O(g)$ for the output.

## Common pitfalls

- **Skipping the $g \mid b$ test.** Dividing $b$ by $g$ when it does not divide gives a bogus congruence with a bogus "solution".
- **Reporting one solution when there are $g$.** A problem asking for the count, or the smallest non-negative $x$, needs all $g$ residues — the smallest is $x_0 \bmod n'$, but list them if the problem wants every solution below $n$.
- **Not reducing $a, b$ modulo $n$ first.** Large or negative $a, b$ still work mathematically, but reducing keeps the inverse call clean.
- **Confusing modulus $n$ and $n/g$.** The unique solution lives modulo $n/g$; only after lifting do you have residues modulo $n$.

## The keystone

$a x \equiv b \pmod n$ reduces, via $g = \gcd(a, n)$, to a congruence with an invertible coefficient: solvable iff $g \mid b$, and then it has $g$ solutions modulo $n$ built from one [modular inverse](/citadel/cp/modular-inverse). Chaining several such congruences with different moduli is the [Chinese Remainder Theorem](/citadel/cp/chinese-remainder-theorem).
