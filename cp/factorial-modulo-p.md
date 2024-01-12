---
title: Factorial Modulo p - Wilson's Theorem and Windowed Products
description: Computing n factorial with the multiples of p removed, why that product is periodic modulo p, how Wilson's theorem pins the period's value, and the O(p log n) routine for the full factorial's p-free part.
date: 2024-01-12
draft: false
slug: /cp/factorial-modulo-p
tags:
  - Competitive Programming
  - Algebra
  - Modular Arithmetic
---

$n! \bmod p$ is $0$ for any $n \ge p$, which is useless. What you actually want in [Lucas' theorem](/citadel/cp/binomial-coefficients) and [Legendre's formula](/citadel/cp/binomial-coefficients) is $n!$ with every factor of $p$ stripped out, taken modulo $p$. That "$p$-free factorial" has a neat periodic structure, and Wilson's theorem gives the period a name.

## The problem

Define $n!_p$ = the product of all integers in $[1, n]$ that are **not** divisible by $p$, reduced modulo $p$:

$$n!_p \;=\; \prod_{\substack{1 \le i \le n \\ p \,\nmid\, i}} i \pmod p.$$

With $n!_p$ and the exponent of $p$ in $n!$ (from Legendre), you recover $n! / p^{e}$ modulo $p$, which is what Lucas needs. Example: $p = 5$, $n = 12$. The kept factors are $1,2,3,4,6,7,8,9,11,12$; their product modulo $5$ is $4$.

## The idea

Split $[1, n]$ into blocks of length $p$. Within one full block $[kp+1, kp+p]$, the residues of the non-multiples of $p$ are exactly $1, 2, \dots, p-1$ in some order, so each full block contributes the same value:

$$\prod_{i=1}^{p-1} i = (p-1)! \pmod p.$$

**Wilson's theorem** says $(p-1)! \equiv -1 \pmod p$ for prime $p$. So $\lfloor n/p \rfloor$ full blocks contribute $(-1)^{\lfloor n/p \rfloor}$, and the leftover partial block contributes $\prod_{i=1}^{n \bmod p} i$ (skipping any multiple of $p$, of which there are none in a partial block below $p$).

That handles $[1, n]$ but not the recursion: $n!$ also contains $\lfloor n/p \rfloor!$ worth of $p$-factors nested inside. To get the $p$-free part of the *full* factorial you recurse on $\lfloor n/p \rfloor$.

## How it works

$p = 5$, $n = 12$. Full blocks: $\lfloor 12/5 \rfloor = 2$, contributing $(-1)^2 = 1$. Partial block: $12 \bmod 5 = 2$, product $1 \cdot 2 = 2$. So the one-level product is $1 \cdot 2 = 2 \pmod 5$... but the direct count above gave $4$. The difference is the nested multiples $5, 10$ (i.e. $5 \cdot 1, 5 \cdot 2$): dividing them by $5$ leaves $1 \cdot 2 = 2$, and $2 \cdot 2 = 4$. Recursing on $\lfloor 12/5 \rfloor = 2$ gives that factor of $2$, and $2 \cdot 2 = 4$. Matches.

## Algorithm

`factmod(n, p)` returns the $p$-free part of $n!$ modulo $p$:

1. Result $\leftarrow 1$.
2. While $n > 1$:
   a. Multiply result by $(-1)^{\lfloor n/p \rfloor}$ (i.e. flip sign if $\lfloor n/p \rfloor$ is odd).
   b. Multiply result by $\prod_{i=2}^{n \bmod p} i \pmod p$.
   c. $n \leftarrow \lfloor n / p \rfloor$.
3. Return result modulo $p$ (normalised to $[0, p)$).

```python
def factmod(n: int, p: int) -> int:
    result = 1
    while n > 1:
        if (n // p) % 2 == 1:
            result = p - result            # multiply by -1
        block = 1
        for i in range(2, n % p + 1):
            block = block * i % p
        result = result * block % p
        n //= p
    return result % p
```

For many queries with the same $p$, precompute `fact[0..p-1]` once so step 2b is an $O(1)$ table lookup, making each `factmod` call $O(\log_p n)$.

## Complexity

- **Time:** $O(p + \log_p n)$ per call with the precomputed table; $O(p \log_p n)$ without it. The $\log_p n$ is the number of recursion levels.
- **Space:** $O(p)$ for the table.

## Where it is used

- **Lucas' theorem:** $\binom{n}{k} \bmod p$ for prime $p$ and huge $n, k$, by combining digit-wise binomials in base $p$ — each digit needs a factorial modulo $p$.
- **Legendre's formula:** the exponent of $p$ in $n!$ is $\sum_{i \ge 1} \lfloor n/p^i \rfloor = \dfrac{n - s_p(n)}{p - 1}$, where $s_p(n)$ is the digit sum of $n$ in base $p$. Pair it with `factmod` to get $n! \bmod p^{\text{anything}}$ via the $p$-free part times $p^{\text{exponent}}$.
- **Wilson-based primality:** $(n-1)! \equiv -1 \pmod n$ **iff** $n$ is prime — elegant but $O(n)$, so only a talking point next to [Miller-Rabin](/citadel/cp/primality-tests).

## Common pitfalls

- **Using it for $n! \bmod p$ directly.** If $n \ge p$ the true $n! \bmod p$ is $0$; `factmod` deliberately computes the $p$-**free** part, a different quantity.
- **Sign bookkeeping.** The factor is $(-1)^{\lfloor n/p \rfloor}$ *per level*; implement it as a conditional negate, and normalise the final result into $[0, p)$.
- **Composite modulus.** Wilson's theorem needs $p$ prime. For $p = 4$, $(p-1)! = 6 \equiv 2 \not\equiv -1$. The whole block argument collapses.
- **Off-by-one in the partial block.** It runs $i = 1 \dots (n \bmod p)$; starting the loop at $2$ is only an optimisation because multiplying by $1$ is a no-op.

## The keystone

Modulo a prime $p$, the $p$-free factorial is periodic with period $p$, and Wilson's theorem fixes each period's product at $-1$ — so $n!$ with its $p$-factors removed collapses to a sign, a short partial product, and a recursion on $\lfloor n/p \rfloor$. That is the engine behind [Lucas' theorem](/citadel/cp/binomial-coefficients) for binomial coefficients with astronomically large arguments.
