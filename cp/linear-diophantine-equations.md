---
title: Linear Diophantine Equations - Integer Solutions to ax + by = c
description: When ax + by = c has integer solutions, how extended Euclid produces one, how to slide along the solution line to the rest, and how to count or minimise solutions in a range.
date: 2023-12-16
draft: false
slug: /cp/linear-diophantine-equations
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

A Diophantine equation asks for *integer* solutions. The linear two-variable case, $ax + by = c$, is completely solved: it has solutions exactly when $\gcd(a, b)$ divides $c$, [extended Euclid](/citadel/cp/extended-euclidean) builds one, and every other solution is a fixed step away along a line.

## The problem

Given integers $a$, $b$, $c$ (with $a, b$ not both zero), find integers $x, y$ with

$$a x + b y = c,$$

or report that none exist. Follow-up questions common in contests: find the solution with the smallest non-negative $x$; count solutions with $x \in [x_1, x_2]$ and $y \in [y_1, y_2]$.

Example: $12x + 18y = 30$. Here $\gcd(12, 18) = 6$ and $6 \mid 30$, so solutions exist; one is $(x, y) = (-5, 5)$, since $12(-5) + 18(5) = -60 + 90 = 30$, and another is $(1, 1)$.

## The idea

**Existence.** $ax + by$ is always a multiple of $g = \gcd(a, b)$, so if $g \nmid c$ there is nothing to find. If $g \mid c$, extended Euclid gives $x', y'$ with $ax' + by' = g$; scale by $c / g$:

$$x_0 = x' \cdot \frac{c}{g}, \qquad y_0 = y' \cdot \frac{c}{g}$$

is one solution.

**All solutions.** If $(x_0, y_0)$ works, so does

$$\left(x_0 + k \cdot \frac{b}{g},\;\; y_0 - k \cdot \frac{a}{g}\right) \quad \text{for every integer } k,$$

because the added terms contribute $a \cdot k\frac{b}{g} - b \cdot k\frac{a}{g} = 0$. And these are the *only* solutions: subtract two solutions and you get $a\,\Delta x = -b\,\Delta y$, which forces $\Delta x$ to be a multiple of $b/g$. So the solution set is an arithmetic progression with step $(b/g,\, -a/g)$.

## How it works

$12x + 18y = 30$, $g = 6$, $c/g = 5$. Extended Euclid on $(12, 18)$: $\gcd = 6$ with $12(-1) + 18(1) = 6$, so $x' = -1$, $y' = 1$. Scale by $5$: $(x_0, y_0) = (-5, 5)$, and indeed $12(-5) + 18(5) = 30$.

The general solution is $(-5 + 3k,\; 5 - 2k)$ since $b/g = 3$, $a/g = 2$. Taking $k = 2$: $(1, 1)$, and $12 + 18 = 30$. Taking $k = 1$: $(-2, 3)$, and $-24 + 54 = 30$.

## Algorithm

1. Compute $g, x', y'$ from `ext_gcd(a, b)`.
2. If $c \bmod g \ne 0$, return "no solution".
3. Set $x_0 = x' \cdot (c / g)$, $y_0 = y' \cdot (c / g)$.
4. The step vector is $(b / g,\; -a / g)$. Shift by whichever multiple $k$ lands $x$ (or $y$) in the required range: to make $x$ the smallest non-negative value, $k = \lceil -x_0 / (b/g) \rceil$ (mind the sign of $b/g$).

```python
def diophantine(a: int, b: int, c: int):
    g, x, y = ext_gcd(abs(a), abs(b))
    if c % g != 0:
        return None
    x *= c // g
    y *= c // g
    if a < 0: x = -x
    if b < 0: y = -y
    return x, y, g            # step is (b // g, -a // g)

def ext_gcd(a: int, b: int):
    if b == 0:
        return a, 1, 0
    g, x1, y1 = ext_gcd(b, a % b)
    return g, y1, x1 - (a // b) * y1
```

To count solutions in a box, intersect the two intervals of valid $k$ (one from the $x$ bounds, one from the $y$ bounds) and take the size of the overlap.

## Complexity

- **Time:** $O(\log \min(|a|, |b|))$, dominated by extended Euclid. Range/count follow-ups are $O(1)$ arithmetic on top.
- **Space:** $O(\log \min(|a|, |b|))$ for the recursion, $O(1)$ iteratively.

## Common pitfalls

- **Divisibility test first.** If $g \nmid c$, stop — do not scale, or you produce a fake non-integer "solution".
- **Sign of $a$, $b$.** `ext_gcd` on absolute values is cleanest; then flip the coefficient signs to match the original $a, b$.
- **Step direction.** The step is $(b/g,\, -a/g)$. When $b$ is negative, "increase $k$" can *decrease* $x$; compute the bound with the actual signed step, or take absolute values and reason about direction explicitly.
- **$c = 0$.** Then $(0, 0)$ always works and the general family is $(k\,b/g,\, -k\,a/g)$ — don't let the scaling by $c/g = 0$ hide the non-trivial solutions.
- **Both $a$ and $b$ zero.** Degenerate: solvable iff $c = 0$, with every $(x, y)$ a solution. Guard against dividing by $g = 0$.

## The keystone

$ax + by = c$ is solvable iff $\gcd(a, b) \mid c$; one solution comes from scaling the [Bezout coefficients](/citadel/cp/extended-euclidean), and all solutions form a line with step $(b/g,\, -a/g)$. Counting or optimising over that line is then a one-dimensional interval problem. The same "solve one congruence, then step by the modulus" shape reappears in [linear congruences](/citadel/cp/linear-congruence-equation).
