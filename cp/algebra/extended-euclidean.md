---
title: Extended Euclidean Algorithm - Finding Bezout Coefficients
description: Running Euclid's recursion while carrying coefficients so it returns x and y with ax + by = gcd(a, b) — the general route to modular inverses and the base of every Diophantine solver.
date: 2023-12-16
draft: false
slug: /cp/extended-euclidean
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

The plain [Euclidean algorithm](/citadel/cp/euclidean-gcd) tells you $\gcd(a, b)$. The extended version also tells you *how to build it* from $a$ and $b$: it returns integers $x$ and $y$ with $ax + by = \gcd(a, b)$. Those coefficients are what you need for modular inverses when the modulus is not prime, and for solving $ax + by = c$ over the integers.

## The problem

Given integers $a$ and $b$, find $g = \gcd(a, b)$ together with a pair $(x, y)$ satisfying

$$a x + b y = g.$$

Such a pair always exists (Bezout's identity). Example: $a = 240$, $b = 46$. Then $g = 2$, and one valid pair is $x = -9$, $y = 47$, since $240(-9) + 46(47) = -2160 + 2162 = 2$.

## The idea

Euclid recurses on $\gcd(a, b) = \gcd(b, a \bmod b)$. Suppose the recursive call has already solved the smaller instance:

$$b \cdot x_1 + (a \bmod b) \cdot y_1 = g.$$

Substitute $a \bmod b = a - \lfloor a / b \rfloor \cdot b$:

$$b x_1 + \left(a - \left\lfloor \tfrac{a}{b} \right\rfloor b\right) y_1 = g \;\Longrightarrow\; a\, y_1 + b\left(x_1 - \left\lfloor \tfrac{a}{b} \right\rfloor y_1\right) = g.$$

So the coefficients for $(a, b)$ are

$$x = y_1, \qquad y = x_1 - \left\lfloor \tfrac{a}{b} \right\rfloor y_1.$$

The base case is $b = 0$: $\gcd(a, 0) = a = a \cdot 1 + 0 \cdot 0$, so $(x, y) = (1, 0)$.

## How it works

Solve $240x + 46y = 2$. The recursion unwinds from the base case upward, each row using $x = y_1$ and $y = x_1 - \lfloor a/b \rfloor\, y_1$ on the row below it:

| $(a, b)$ | $\lfloor a/b \rfloor$ | returns $(g,\; x,\; y)$ | check $ax + by$ |
| --- | --- | --- | --- |
| $(2, 0)$ | — | $(2,\; 1,\; 0)$ | $2$ |
| $(4, 2)$ | 2 | $(2,\; 0,\; 1 - 2\cdot 0) = (2, 0, 1)$ | $2$ |
| $(6, 4)$ | 1 | $(2,\; 1,\; 0 - 1\cdot 1) = (2, 1, -1)$ | $2$ |
| $(10, 6)$ | 1 | $(2,\; -1,\; 1 - 1\cdot(-1)) = (2, -1, 2)$ | $2$ |
| $(46, 10)$ | 4 | $(2,\; 2,\; -1 - 4\cdot 2) = (2, 2, -9)$ | $2$ |
| $(240, 46)$ | 5 | $(2,\; -9,\; 2 - 5\cdot(-9)) = (2, -9, 47)$ | $240(-9) + 46(47) = 2$ |

The final row is the answer: $g = 2$, $(x, y) = (-9, 47)$.

## Algorithm

1. If $b = 0$, return $(a,\; 1,\; 0)$.
2. Recurse on $(b,\; a \bmod b)$ to get $(g,\; x_1,\; y_1)$.
3. Return $(g,\; y_1,\; x_1 - \lfloor a / b \rfloor \cdot y_1)$.

```python
def ext_gcd(a: int, b: int) -> tuple[int, int, int]:
    if b == 0:
        return a, 1, 0
    g, x1, y1 = ext_gcd(b, a % b)
    return g, y1, x1 - (a // b) * y1
```

Iterative form, if recursion depth is a worry:

```python
def ext_gcd_iter(a: int, b: int) -> tuple[int, int, int]:
    old_r, r = a, b
    old_s, s = 1, 0
    old_t, t = 0, 1
    while r:
        q = old_r // r
        old_r, r = r, old_r - q * r
        old_s, s = s, old_s - q * s
        old_t, t = t, old_t - q * t
    return old_r, old_s, old_t
```

## Complexity

- **Time:** $O(\log \min(a, b))$ — the same recursion as plain Euclid, plus $O(1)$ coefficient arithmetic per level.
- **Space:** $O(\log \min(a, b))$ for the recursion stack, $O(1)$ iteratively.

## Variations

- **Modular inverse of $a$ mod $m$.** Call `ext_gcd(a, m)`. If $g \ne 1$ the inverse does not exist; otherwise `x % m` is $a^{-1}$. This works for *any* modulus, unlike the Fermat trick which needs $m$ prime.
- **[Linear Diophantine](/citadel/cp/linear-diophantine-equations) $ax + by = c$.** Solvable iff $g \mid c$; scale $(x, y)$ by $c / g$, then shift by multiples of $(b/g,\, -a/g)$ to reach any other solution or a non-negative one.
- **[CRT](/citadel/cp/chinese-remainder-theorem) merge step.** Combining $x \equiv r_1 \pmod{m_1}$ and $x \equiv r_2 \pmod{m_2}$ uses the Bezout coefficients of $m_1, m_2$.

## Common pitfalls

- **Sign and range of the output.** $x$ can be negative or larger than $m$. For an inverse, return `x % m` to normalise into $[0, m)$.
- **Non-coprime inverse.** If $\gcd(a, m) \ne 1$, there is no inverse — check $g$ before dividing by it.
- **Recursion depth.** For adversarial Fibonacci-like inputs the depth is a few dozen for 64-bit values, but thousands for big integers; use the iterative form when inputs are unbounded.
- **Overflow in the coefficient update.** `x1 - (a // b) * y1` can be large in fixed-width languages; Python is immune, but keep the multiply-then-subtract order clear.

## The keystone

Extended Euclid is plain Euclid with two extra running numbers that reconstruct the GCD as an integer combination $ax + by = g$. Those Bezout coefficients are the general-purpose modular inverse (no primality needed) and the starting point for every equation of the form $ax + by = c$, which is exactly what the [linear Diophantine](/citadel/cp/linear-diophantine-equations) post picks up.
