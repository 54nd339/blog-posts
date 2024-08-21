---
title: Lattice Points Under a Segment - A Euclid-Like Count
description: Counting integer points under the line segment from (0,0) to (n, m) with a recursion that mirrors the Euclidean algorithm, giving the floor-sum sum floor(a*i + b) over i in O(log) time.
date: 2024-08-21
draft: false
slug: /cp/lattice-points-under-segment
tags:
  - Competitive Programming
  - Geometry
  - Number Theory
---

How many lattice points $(x, y)$ with $y \ge 1$ lie strictly below the segment from $(0, 0)$ to $(p, q)$? Equivalently, evaluate the **floor sum** $\sum_{i=0}^{n-1} \left\lfloor \dfrac{a i + b}{m} \right\rfloor$. A direct loop is $O(n)$; a recursion that peels off the integer part exactly like the [Euclidean algorithm](/citadel/cp/euclidean-gcd) does it in $O(\log)$.

## The problem

Compute

$$S(a, b, m, n) = \sum_{i=0}^{n-1} \left\lfloor \frac{a\,i + b}{m} \right\rfloor,$$

with $a, b \ge 0$, $m \ge 1$, $n \ge 0$. Geometrically, with $b = 0$ and the line $y = \frac{a}{m} x$, this counts lattice points $(x, y)$, $0 \le x < n$, $1 \le y \le \lfloor a x / m \rfloor$ — the points under the line.

Example: $a = 3, b = 0, m = 2, n = 5$: $\lfloor 0 \rfloor + \lfloor 1.5 \rfloor + \lfloor 3 \rfloor + \lfloor 4.5 \rfloor + \lfloor 6 \rfloor = 0 + 1 + 3 + 4 + 6 = 14$.

## The idea

Reduce the coefficients modulo $m$, pulling out the integer parts as closed-form sums:

- If $a \ge m$: $\lfloor (ai + b)/m \rfloor = i\lfloor a/m \rfloor + \lfloor ((a \bmod m) i + b)/m \rfloor$, so subtract $\lfloor a/m \rfloor \cdot \frac{n(n-1)}{2}$ and replace $a$ by $a \bmod m$.
- If $b \ge m$: similarly subtract $\lfloor b/m \rfloor \cdot n$ and replace $b$ by $b \bmod m$.

Now $0 \le a, b < m$. Let $y_{\max} = \lfloor (a n + b)/m \rfloor$ = the largest floor value. Count the same lattice points **by rows instead of columns**: the number of $i$ with $\lfloor (ai + b)/m \rfloor \ge y$ is $n - \lceil (ym - b)/a \rceil = \lfloor (ym - b - 1)/a \rfloor + 1$ (for $a > 0$). Summing over $y = 1 \dots y_{\max}$ turns into another floor sum with $a$ and $m$ swapped — the Euclidean step. If $a = 0$, the sum is just $n\lfloor b/m \rfloor$ (already handled).

## Algorithm

```python
def floor_sum(n: int, m: int, a: int, b: int) -> int:
    """sum_{i=0}^{n-1} floor((a*i + b) / m).  Requires m > 0, n >= 0. Works for negative a, b."""
    ans = 0
    if a < 0:
        a2 = a % m
        ans -= n * (n - 1) // 2 * ((a2 - a) // m)
        a = a2
    if b < 0:
        b2 = b % m
        ans -= n * ((b2 - b) // m)
        b = b2
    while True:
        if a >= m:
            ans += n * (n - 1) // 2 * (a // m)
            a %= m
        if b >= m:
            ans += n * (b // m)
            b %= m
        y_max = a * n + b
        if y_max < m:
            break
        n = y_max // m
        b = y_max % m
        m, a = a, m
    return ans
```

This is the AtCoder Library `floor_sum` — battle-tested for exactly this problem.

## What it computes for you

- **Lattice points under a line** $y = \frac{a}{m}x$ for $x \in [0, n)$: `floor_sum(n, m, a, 0)`.
- **Lattice points inside a triangle** with a right angle at the origin and hypotenuse to $(p, q)$: `floor_sum(p + 1, q, ...)`-style, minus boundary via [gcd](/citadel/cp/euclidean-gcd).
- **$\sum \lfloor (a i + b) / m \rfloor$** modular / divisibility sums: e.g. $\sum_{i=1}^{n} \lfloor n/i \rfloor$ (divisor summatory function) has its own $O(\sqrt n)$ trick, but many similar sums fall to `floor_sum` directly.
- **Number of solutions to $ax + by \le c$** in a box, a linear-programming-flavoured count.
- **Rounding-error accumulation**, **Stern-Brocot / continued-fraction** navigation, **counting fractions $\le$ a bound with denominator $\le n$**.

## Complexity

- **Time:** $O(\log(\max(a, m)))$ — the loop runs the Euclidean recursion on $(a, m)$.
- **Space:** $O(1)$.

## Common pitfalls

- **Negative $a$ or $b$.** Reduce them into $[0, m)$ first, adjusting `ans` by the closed-form correction (as the code does), or the row-counting step goes wrong.
- **$n = 0$.** The sum is empty $= 0$; the formulas with $\frac{n(n-1)}{2}$ handle it, but check.
- **Overflow.** $\frac{n(n-1)}{2}\lfloor a/m \rfloor$ can be huge; Python is fine, C++ needs 64-bit or 128-bit.
- **Off-by-one between "under" and "on or under".** Points *on* the line ($y = \frac{a}{m}x$ exactly) are $\gcd(p, q) + 1$ of them on the segment $(0,0)$–$(p,q)$; decide whether your count includes them and adjust.
- **$m = 0$.** Undefined — the caller must pass $m \ge 1$.

## The keystone

$\sum_{i<n} \lfloor (ai + b)/m \rfloor$ counts lattice points under a line, and swapping the roles of $a$ and $m$ (after pulling out integer parts) recurses exactly like the Euclidean algorithm — $O(\log)$ instead of $O(n)$. It is the same identity that navigates the [Stern-Brocot tree](/citadel/cp/stern-brocot-tree).
