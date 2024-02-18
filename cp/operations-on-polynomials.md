---
title: Operations on Polynomials and Series - Inverse, Log, and Exp
description: Building the power-series toolkit on top of fast multiplication — Newton iteration for the inverse and square root, formal log and exp, and polynomial division — each doubling the number of correct coefficients per step.
date: 2024-02-18
draft: false
slug: /cp/operations-on-polynomials
tags:
  - Competitive Programming
  - Algebra
  - Polynomials
---

Once you can multiply two degree-$n$ polynomials in $O(n \log n)$ (see the [FFT](/citadel/algorithms/FastFourierTransform) post, and its number-theoretic transform variant for exact arithmetic modulo a prime), a whole family of operations follows: inverse, division, square root, logarithm, exponential. Each is a Newton iteration that **doubles** the number of correct coefficients per step, so the total cost is a geometric series dominated by the last multiply: $O(n \log n)$.

## The problem

Work with formal power series modulo $x^n$ over $\mathbb{Z}/p\mathbb{Z}$ (with $p$ an NTT-friendly prime like $998244353 = 119 \cdot 2^{23} + 1$). Given $A(x)$, compute:

- $A(x)^{-1} \bmod x^n$ (needs $A(0) \ne 0$),
- $Q, R$ with $A = QB + R$, $\deg R < \deg B$ (polynomial division),
- $\sqrt{A} \bmod x^n$ (needs $A(0)$ a QR, usually $A(0) = 1$),
- $\ln A \bmod x^n$ and $\exp A \bmod x^n$ (need $A(0) = 1$ and $A(0) = 0$ respectively).

## The idea: Newton iteration on series

To solve $F(G) = 0$ for a series $G$, given a solution $G_k$ correct modulo $x^{k}$, the update

$$G_{k+1} \equiv G_k - \frac{F(G_k)}{F'(G_k)} \pmod{x^{2k}}$$

is correct modulo $x^{2k}$. Start from the constant term and double the precision $1 \to 2 \to 4 \to \dots \to n$.

### Inverse

$F(G) = 1/G - A$, giving the update

$$G_{k+1} \equiv G_k\,(2 - A\,G_k) \pmod{x^{2k}}, \qquad G_1 = A(0)^{-1}.$$

```python
MOD = 998244353

def inverse(A: list[int], n: int) -> list[int]:
    g = [pow(A[0], MOD - 2, MOD)]
    k = 1
    while k < n:
        k <<= 1
        a = A[:k] + [0] * (k - len(A[:k]))
        ag = multiply(a, g)[:k]                 # multiply = NTT-based product mod MOD
        two_minus = [(-c) % MOD for c in ag]
        two_minus[0] = (two_minus[0] + 2) % MOD
        g = multiply(g, two_minus)[:k]
    return g[:n]
```

### Logarithm and exponential

For $A(0) = 1$: $\ln A = \int \dfrac{A'}{A}\,dx \bmod x^n$ — one inverse, one multiply, one termwise integrate.

For $A(0) = 0$: $\exp A$ by Newton on $F(G) = \ln G - A$:

$$G_{k+1} \equiv G_k\,\big(1 + A - \ln G_k\big) \pmod{x^{2k}}, \qquad G_1 = 1.$$

```python
def deriv(A):  return [(i * c) % MOD for i, c in enumerate(A)][1:] or [0]
def integ(A):  return [0] + [c * pow(i + 1, MOD - 2, MOD) % MOD for i, c in enumerate(A)]

def log(A: list[int], n: int) -> list[int]:
    assert A[0] == 1
    res = multiply(deriv(A), inverse(A, n))[:n - 1]
    return integ(res)[:n]

def exp(A: list[int], n: int) -> list[int]:
    assert A[0] == 0
    g = [1]
    k = 1
    while k < n:
        k <<= 1
        lg = log(g + [0] * (k - len(g)), k)
        t = [(A[i] if i < len(A) else 0) - lg[i] for i in range(k)]
        t[0] = (t[0] + 1) % MOD
        g = multiply(g, [c % MOD for c in t])[:k]
    return g[:n]
```

### Polynomial division

To divide $A$ (degree $n$) by $B$ (degree $m$), reverse both, multiply $A^R$ by $(B^R)^{-1} \bmod x^{\,n-m+1}$, reverse back to get $Q$; then $R = A - QB$.

## How it works

Inverse of $A = 1 + 2x + 3x^2 + \dots$ modulo $x^4$. $G_1 = 1$. Step to $k = 2$: $A G_1 = 1 + 2x$, $2 - AG_1 = 1 - 2x$, $G_2 = G_1(1 - 2x) = 1 - 2x$. Check: $(1 + 2x)(1 - 2x) = 1 - 4x^2 \equiv 1 \pmod{x^2}$. Step to $k = 4$: $A G_2 \bmod x^4 = (1 + 2x + 3x^2 + 4x^3)(1 - 2x) \bmod x^4 = 1 + 0x + (-1)x^2 + (-2)x^3$; $2 - AG_2 = 1 + x^2 + 2x^3$; $G_4 = G_2 (1 + x^2 + 2x^3) \bmod x^4 = 1 - 2x + x^2 + 0x^3$. Multiplying $A$ by $G_4$ gives $1 + O(x^4)$.

## Complexity

- **Multiplication:** $O(n \log n)$ via NTT.
- **Inverse, log, exp, sqrt, division:** $O(n \log n)$ — each Newton step at precision $k$ costs $O(k \log k)$, and $\sum_{k = 1, 2, 4, \dots, n} k \log k = \Theta(n \log n)$.

## Common pitfalls

- **Non-invertible constant term.** Inverse needs $A(0) \ne 0$; $\sqrt{A}$ and $\ln A$ need $A(0) = 1$ (or a known square root of $A(0)$); $\exp A$ needs $A(0) = 0$. Check before iterating.
- **Truncation discipline.** Every product must be cut back to the current precision $x^{k}$ *before* the next step, or the series grows and the cost blows up.
- **NTT prime and length.** The transform length must be a power of two dividing $p - 1$. With $p = 998244353$ that allows up to $2^{23}$; for larger sizes use a different prime or a 3-prime CRT.
- **Integration dividing by zero.** $\int$ multiplies coefficient $i$ by $(i+1)^{-1}$; fine modulo a prime, but plain-integer code must special-case it.
- **Reversal length in division.** Reverse to exactly degree $n$ and $m$; an off-by-one there shifts the whole quotient.

## The keystone

Every power-series operation past multiplication is a Newton iteration that doubles correct coefficients per step: $G_{k+1} = G_k(2 - A G_k)$ for the inverse, and analogous updates for $\sqrt{}$, $\ln$, $\exp$. Because the work per step doubles, the total stays $O(n \log n)$ — the same as a single [fast multiplication](/citadel/algorithms/FastFourierTransform).
