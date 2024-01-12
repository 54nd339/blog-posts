---
title: Discrete Logarithm - Baby-Step Giant-Step
description: Solving a to the x congruent b mod m in O(sqrt m) with a meet-in-the-middle split of the exponent, a hash table of baby steps, and the tweak that makes it work when a and m share a factor.
date: 2024-01-12
draft: false
slug: /cp/discrete-logarithm
tags:
  - Competitive Programming
  - Algebra
  - Modular Arithmetic
---

Solving $a^x \equiv b \pmod m$ for $x$ is the discrete logarithm problem — easy to state, believed hard in general (public-key crypto rests on it), but $O(\sqrt m)$ is reachable with a meet-in-the-middle trick called baby-step giant-step.

## The problem

Given $a$, $b$, $m$, find the smallest non-negative $x$ with $a^x \equiv b \pmod m$, or report that none exists.

Example: $2^x \equiv 3 \pmod 5$. Powers of $2$ mod $5$ cycle $1, 2, 4, 3, 1, \dots$, so $x = 3$.

## The idea

Write $x = n p - q$ with $n = \lceil \sqrt m \rceil$ and $0 \le q < n$, $1 \le p \le n$. Then

$$a^{np - q} \equiv b \pmod m \;\Longleftrightarrow\; a^{np} \equiv b \, a^{q} \pmod m \;\Longleftrightarrow\; (a^n)^{p} \equiv b\, a^{q} \pmod m.$$

- **Baby steps:** compute $b a^{q}$ for $q = 0, 1, \dots, n-1$ and store each in a hash map value → $q$.
- **Giant steps:** compute $(a^n)^{p}$ for $p = 1, 2, \dots, n$; the first one found in the map gives $x = n p - q$.

Two loops of length $\sqrt m$ each, one hash lookup per giant step — $O(\sqrt m)$ time and memory.

## How it works

$2^x \equiv 3 \pmod 5$, $n = \lceil \sqrt 5 \rceil = 3$.

Baby steps $3 \cdot 2^q \bmod 5$: $q=0 \to 3$, $q=1 \to 1$, $q=2 \to 2$. Map: $\{3:0,\ 1:1,\ 2:2\}$.

Giant step base $2^3 = 8 \equiv 3$. $p = 1$: $3^1 = 3$, found in the map with $q = 0$. So $x = 3 \cdot 1 - 0 = 3$.

## Algorithm (coprime case, $\gcd(a, m) = 1$)

1. $n \leftarrow \lceil \sqrt m \rceil$.
2. Build `table = {}`; `cur = b % m`; for $q$ in $0 \dots n-1$: `table.setdefault(cur, q)`; `cur = cur * a % m`.
3. $a_n \leftarrow a^{n} \bmod m$; `giant = 1`.
4. For $p$ in $1 \dots n$: `giant = giant * a_n % m`; if `giant in table`: return $n p - \texttt{table[giant]}$.
5. Return "no solution".

```python
def bsgs(a: int, b: int, m: int):
    a %= m; b %= m
    if b == 1 % m:
        return 0
    n = int(m ** 0.5) + 1
    table = {}
    cur = b
    for q in range(n + 1):
        table.setdefault(cur, q)
        cur = cur * a % m
    an = pow(a, n, m)
    giant = 1
    for p in range(1, n + 1):
        giant = giant * an % m
        if giant in table:
            return n * p - table[giant]
    return None
```

## The non-coprime case

If $g = \gcd(a, m) > 1$, then for $x \ge 1$ the value $a^x \bmod m$ is always a multiple of $g$, so a target $b$ with $g \nmid b$ is unreachable. Peel one factor at a time. From $a^x \equiv b \pmod m$ with $g \mid a$ and $g \mid m$, dividing through by $g$ gives

$$\frac{a}{g} \cdot a^{x-1} \equiv \frac{b}{g} \pmod{\frac{m}{g}}.$$

Repeat until $\gcd(a, m) = 1$, accumulating the leading coefficient in $c$ and the shift in $k$. What remains is $c \cdot a^{y} \equiv b' \pmod{m'}$ with $\gcd(a, m') = 1$ and $x = y + k$ — solved by BSGS with the giant step seeded at $c$ instead of $1$.

```python
def bsgs_general(a: int, b: int, m: int):
    a %= m; b %= m
    k, c = 0, 1
    while (g := gcd(a, m)) > 1:
        if b == c:                       # x == k is the answer
            return k
        if b % g:
            return None                  # b unreachable
        b //= g
        m //= g
        c = c * (a // g) % m
        k += 1
    # now gcd(a, m) == 1; solve c * a^y == b (mod m), answer x = y + k
    n = int(m ** 0.5) + 1
    table = {}
    cur = b
    for q in range(n + 1):
        table.setdefault(cur, q)
        cur = cur * a % m
    an = pow(a, n, m)
    giant = c
    for p in range(1, n + 1):
        giant = giant * an % m
        if giant in table:
            return n * p - table[giant] + k
    return None
```

## Complexity

- **Time:** $O(\sqrt m \log m)$ — $\sqrt m$ modular multiplications for each phase, plus hash-map operations; the $\log m$ is from `pow`/`gcd` in the reduction.
- **Space:** $O(\sqrt m)$ for the baby-step table.

## Common pitfalls

- **`setdefault` vs `[]=`.** You want the *smallest* $q$ per value, so keep the first write. Overwriting can still give a valid $x$ but not the minimum.
- **Checking $x = 0$.** $a^0 = 1$; if $b \equiv 1$ the answer is $0$. The giant loop starting at $p = 1$ misses it — test up front.
- **`int(m ** 0.5)` rounding.** Floating `sqrt` can be a hair low for large $m$; add $1$ (or use `isqrt(m) + 1`) so $n^2 \ge m$.
- **Non-coprime without the reduction.** Plain BSGS on $\gcd(a, m) > 1$ silently misses solutions. Use the peeling version.
- **Order vs discrete log.** If you only need the multiplicative order of $a$, that is the smallest $x > 0$ with $a^x \equiv 1$ — a divisor of $\varphi(m)$, found by factorising $\varphi(m)$, usually faster than BSGS.

## The keystone

Baby-step giant-step splits the unknown exponent as $x = np - q$ and meets in the middle: a $\sqrt m$-size table of $b a^q$ against $\sqrt m$ powers of $a^n$. That turns a search over $m$ exponents into $O(\sqrt m)$ work, and a short gcd-peeling preamble extends it to moduli that share a factor with the base.
