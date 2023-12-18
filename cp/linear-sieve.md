---
title: Linear Sieve - Primes and Least Prime Factors in O(n)
description: A sieve that crosses every composite exactly once by pairing it with its smallest prime factor, giving true linear time, a ready-made factorization table, and a slot to compute any multiplicative function.
date: 2023-12-18
draft: false
slug: /cp/linear-sieve
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

The [Sieve of Eratosthenes](/citadel/cp/sieve-of-eratosthenes) crosses some composites several times — $12$ gets struck by both $2$ and $3$. The linear sieve fixes the enumeration so every composite is crossed *once*, by its smallest prime factor. That drops the cost to $O(n)$ and, as a bonus, fills a smallest-prime-factor table you can factorize with instantly.

## The problem

For all integers up to $n$: list the primes, and for each composite record its **smallest prime factor** (SPF). With SPF in hand, factorizing any $x \le n$ is an $O(\log x)$ loop, and multiplicative functions (Euler's [totient](/citadel/cp/eulers-totient-function), Möbius $\mu$, number of divisors) can be filled in during the same pass.

## The idea

Maintain a growing list `primes` and an array `spf[2..n]` (0 means "still prime"). Iterate $i$ from $2$ to $n$:

1. If `spf[i] == 0`, then $i$ is prime: append it, set `spf[i] = i`.
2. For each prime $p$ in `primes`, in increasing order, while $p \le \texttt{spf}[i]$ and $i \cdot p \le n$: set `spf[i*p] = p`. **Break as soon as $p$ divides $i$** (equivalently $p = \texttt{spf}[i]$).

The break is the whole trick. Each composite $c$ is written exactly when $i = c / p^{\star}$ and $p = p^{\star}$, where $p^{\star}$ is $c$'s smallest prime factor — because once $p^{\star} \mid i$, continuing to a larger $p$ would mark $i \cdot p$ whose smallest prime factor is $p^{\star}$, not $p$. Stopping prevents that double mark. One write per composite ⇒ $O(n)$.

## How it works

$n = 20$, tracing the marks:

- $i = 2$: prime. $p = 2$: mark $4$ (spf 2). $2 \mid 2$, break.
- $i = 3$: prime. $p = 2$: mark $6$. $p = 3$: mark $9$. $3 \mid 3$, break.
- $i = 4$: `spf[4]=2`. $p = 2$: mark $8$. $2 \mid 4$, break.
- $i = 5$: prime. $p=2$: mark $10$. $p=3$: mark $15$. $p=5$: mark $25 > 20$, stop.
- $i = 6$: `spf[6]=2`. $p = 2$: mark $12$. $2 \mid 6$, break.
- $i = 7$: prime. mark $14$; $p=3$ mark $21>20$ stop.
- $i = 8$: `spf=2`. $p=2$: mark $16$. break.
- $i = 9$: `spf=3`. $p=2$: mark $18$. $p=3$: $3\mid9$, mark $27>20$ stop... actually $9\cdot3=27>20$ so nothing, break.
- $i = 10$: `spf=2`. $p=2$: mark $20$. break.

Every composite $\le 20$ was written exactly once.

## Algorithm

1. `spf = [0] * (n + 1)`, `primes = []`.
2. For `i` from `2` to `n`:
   - If `spf[i] == 0`: `spf[i] = i`; `primes.append(i)`.
   - For `p` in `primes`:
     - If `p > spf[i]` or `i * p > n`: break.
     - `spf[i * p] = p`.

```python
def linear_sieve(n: int):
    spf = [0] * (n + 1)
    primes = []
    for i in range(2, n + 1):
        if spf[i] == 0:
            spf[i] = i
            primes.append(i)
        for p in primes:
            if p > spf[i] or i * p > n:
                break
            spf[i * p] = p
    return primes, spf

def factorize(x: int, spf: list[int]) -> dict[int, int]:
    f = {}
    while x > 1:
        p = spf[x]
        while x % p == 0:
            f[p] = f.get(p, 0) + 1
            x //= p
    return f
```

## Complexity

- **Time:** $O(n)$ — one array write per composite, one append per prime.
- **Space:** $O(n)$ for `spf`, plus $O(\pi(n))$ for the prime list.

In raw speed the linear sieve is often *slower* than a well-tuned bitset Eratosthenes for just "list the primes", because of the modulo-free but cache-unfriendly writes. Reach for it when you actually need the SPF table or a multiplicative function.

## Computing a multiplicative function in the same pass

For a multiplicative $f$ you need three rules: $f(p)$ for a prime, $f(p^k)$ for a prime power, and $f(a \cdot b) = f(a) f(b)$ for coprime $a, b$. The sieve gives you the case split for free — when marking `i * p`:

- if $p \nmid i$: `f[i*p] = f[i] * f[p]` (coprime),
- if $p \mid i$: `i * p` extends the power of $p$ already in `i`; use the prime-power rule, e.g. for the totient `phi[i*p] = phi[i] * p`.

## Common pitfalls

- **Forgetting the break.** Without `if p > spf[i]: break`, composites get multiple writes and the time degrades toward $O(n \log \log n)$ — and any multiplicative-function logic that assumes a single write becomes wrong.
- **Using `i % p == 0` as the break condition.** Equivalent to `p == spf[i]` here, and also fine — but then you must place the `spf[i*p] = p` write *before* the break so $p = \texttt{spf}[i]$ still marks $i \cdot p$.
- **Iterating primes in the wrong order.** `primes` must be ascending; it naturally is, since you append in increasing `i`.
- **`i * p` overflow.** Not in Python. In C++ compare as `i <= n / p` or use 64-bit.

## The keystone

Pairing each composite with its smallest prime factor and stopping the inner loop the moment that factor divides `i` guarantees one write per number, hence linear time — and the by-product `spf` array is a factorization oracle plus a scaffold for any multiplicative function. When a problem needs $\varphi$, $\mu$, or divisor counts for *every* integer up to $n$, this is the pass that produces them.
