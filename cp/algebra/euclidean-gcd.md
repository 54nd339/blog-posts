---
title: Euclidean Algorithm - Computing the Greatest Common Divisor
description: Why gcd(a, b) equals gcd(b, a mod b), the two-line recursion it produces, its logarithmic running time from a Fibonacci worst case, and the LCM identity that rides along.
date: 2023-12-07
draft: false
slug: /cp/euclidean-gcd
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

The greatest common divisor of two numbers is the largest integer dividing both. Factoring each number and intersecting the prime powers works but needs factorization, which is slow. Euclid's algorithm gets the GCD in a logarithmic number of steps using nothing but the remainder operation, and it is more than two thousand years old.

## The problem

Given non-negative integers $a$ and $b$, not both zero, find $\gcd(a, b)$: the largest $d$ such that $d \mid a$ and $d \mid b$. By convention $\gcd(a, 0) = a$.

Example: $\gcd(48, 18)$. The common divisors are $1, 2, 3, 6$, so the answer is $6$.

## The idea

The whole algorithm rests on one identity:

$$\gcd(a, b) = \gcd(b,\; a \bmod b) \quad \text{for } b > 0.$$

Why it holds: write $a = qb + r$ with $r = a \bmod b$. Any number that divides both $a$ and $b$ also divides $a - qb = r$, so it is a common divisor of $b$ and $r$. Conversely, anything dividing $b$ and $r$ divides $qb + r = a$. The two pairs $(a, b)$ and $(b, r)$ have *exactly the same* set of common divisors, so their greatest one is the same.

Each step replaces $(a, b)$ with $(b, a \bmod b)$. The second entry strictly decreases and stays non-negative, so it hits $0$ after finitely many steps; when it does, the first entry is the GCD.

## How it works

$\gcd(48, 18)$:

$$\gcd(48, 18) = \gcd(18, 48 \bmod 18) = \gcd(18, 12) = \gcd(12, 6) = \gcd(6, 0) = 6.$$

Four remainder operations. Notice the arguments shrink quickly — that is the logarithmic behaviour showing up.

## Algorithm

1. If $b = 0$, return $a$.
2. Otherwise return $\gcd(b,\; a \bmod b)$.

```python
def gcd(a: int, b: int) -> int:
    while b:
        a, b = b, a % b
    return a

def lcm(a: int, b: int) -> int:
    return a // gcd(a, b) * b
```

The iterative form is preferred — it never risks a recursion-depth limit. Python also ships `math.gcd`.

## Complexity

- **Time:** $O(\log \min(a, b))$ remainder operations. The worst case is two consecutive [Fibonacci numbers](/citadel/cp/fibonacci-numbers): $\gcd(F_{k+1}, F_k)$ takes $k$ steps, and $F_k$ grows like $\varphi^k$, so the step count is $O(\log_\varphi \min(a,b))$. Each remainder on big integers costs $O((\log a)^2)$, giving $O((\log a)^2 \log a)$ in the worst case for arbitrary precision.
- **Space:** $O(1)$ iteratively.

## Variations

- **LCM.** From $\gcd(a,b) \cdot \operatorname{lcm}(a,b) = a \cdot b$, compute `a // gcd(a, b) * b`. Divide *before* multiplying so the intermediate value never exceeds the answer.
- **GCD of an array.** Fold: `reduce(gcd, arr)`. It short-circuits to $1$ quickly in practice once the running GCD is small.
- **Binary GCD (Stein's algorithm).** Replaces division with subtraction and bit shifts — faster when division is expensive, same asymptotics. Rarely needed in Python.
- **Extended version.** [Extended Euclidean](/citadel/cp/extended-euclidean) tracks coefficients $x, y$ with $ax + by = \gcd(a, b)$ alongside the same recursion.

## Common pitfalls

- **`gcd(0, 0)`.** Undefined mathematically; the code returns $0$, which is the usual convention but worth knowing before you rely on it.
- **LCM overflow.** `a * b // gcd(a, b)` can overflow in fixed-width languages even when the LCM fits. `a // gcd(a, b) * b` avoids it. In Python there is no overflow, but the habit is still right.
- **Negative inputs.** $\gcd$ is defined for non-negatives; take absolute values first if signs can appear.
- **Assuming it needs $a \ge b$.** It does not. If $a < b$, the first step just swaps them, since $a \bmod b = a$.

## The keystone

$\gcd(a, b) = \gcd(b, a \bmod b)$ because subtracting a multiple of $b$ from $a$ leaves the common divisors untouched. That single fact gives an $O(\log n)$ algorithm with no factorization, and it is the engine underneath [linear Diophantine equations](/citadel/cp/linear-diophantine-equations), modular inverses, and the [Chinese Remainder Theorem](/citadel/cp/chinese-remainder-theorem).
