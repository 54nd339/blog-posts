---
title: Integer Factorization - Pollard's Rho and Friends
description: Factoring a single large integer past sieve range — trial division and Fermat for the easy cases, then Pollard's rho with Brent's improvement and a Miller-Rabin primality gate for the full 64-bit range.
date: 2023-12-21
draft: false
slug: /cp/integer-factorization
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

A [sieve](/citadel/cp/linear-sieve) factorizes any number below its bound instantly, but it needs $O(n)$ memory. To factor one 18-digit number you need an algorithm that works on a single input. Trial division is $O(\sqrt n)$ and dies around $10^{13}$; Pollard's rho finds a factor in roughly $O(n^{1/4})$ and handles the whole 64-bit range in microseconds.

## The problem

Given one integer $n$ up to $\approx 10^{18}$, produce its prime factorization $n = \prod p_i^{e_i}$.

Example: $n = 1{,}000{,}000{,}007 \cdot 1{,}000{,}000{,}009 = 1{,}000{,}000{,}016 \cdot 10^{9} + 63$ — a product of two primes near $10^9$. Trial division to $\sqrt n \approx 10^9$ is far too slow; rho finds one factor in a few thousand iterations.

## Approach 1: Trial division

### Intuition

Strip small prime factors directly. Every remaining factor is $> $ the last divisor tried, so once the divisor exceeds $\sqrt{n_{\text{remaining}}}$ what is left is prime.

### Algorithm

1. For $d = 2$, then odd $d$ from $3$ upward while $d \cdot d \le n$: while $d \mid n$, record $d$ and divide it out.
2. If $n > 1$ at the end, it is a prime factor.

```python
def factor_trial(n: int) -> dict[int, int]:
    f = {}
    d = 2
    while d * d <= n:
        while n % d == 0:
            f[d] = f.get(d, 0) + 1
            n //= d
        d += 1 if d == 2 else 2
    if n > 1:
        f[n] = f.get(n, 0) + 1
    return f
```

### Complexity

- **Time:** $O(\sqrt n)$. Practical up to $n \approx 10^{12}$–$10^{13}$.
- **Space:** $O(\log n)$ for the factor list.

Use it as the first pass to remove small primes even when rho does the heavy lifting.

## Approach 2: Pollard's rho (with Brent's cycle detection)

### Intuition

Iterate a pseudo-random map $x \mapsto f(x) = x^2 + c \pmod n$. The sequence of values, taken modulo an unknown prime factor $p$, must cycle after $O(\sqrt p)$ steps by the birthday paradox (the "$\rho$" shape: a tail leading into a loop). When two iterates $x_i, x_j$ collide modulo $p$ but not modulo $n$, $\gcd(|x_i - x_j|, n)$ is a nontrivial factor. Since $p \le \sqrt n$, this takes $O(n^{1/4})$ iterations.

Brent's version replaces Floyd's tortoise-and-hare with a faster cycle finder and batches the $\gcd$ over many differences (one $\gcd$ per ~128 steps), cutting the constant sharply.

### Algorithm

1. If $n$ is even, return $2$.
2. Pick random $x_0$ and constant $c$; set $y = x_0$, $g = 1$, and a running product $q = 1$.
3. Repeat in blocks: advance $x$ by $f$, multiply $q \mathrel{\ast}= |x - y| \bmod n$; every ~128 steps set $g = \gcd(q, n)$ and reset the block.
4. If $g = 1$, keep going. If $1 < g < n$, that is a factor. If $g = n$, the block hid two collisions — restart with a new $c$.

```python
from math import gcd
import random

def pollard_rho(n: int) -> int:
    if n % 2 == 0:
        return 2
    while True:
        c = random.randrange(1, n)
        f = lambda x: (x * x + c) % n
        x = y = random.randrange(2, n)
        g = q = 1
        while g == 1:
            for _ in range(128):
                x, y = f(x), f(f(y))
                q = q * abs(x - y) % n
                if q == 0:
                    break
            g = gcd(q, n)
        if g != n:
            return g

def factorize(n: int) -> dict[int, int]:
    f: dict[int, int] = {}
    def rec(m: int):
        if m == 1:
            return
        if is_prime(m):                       # Miller-Rabin, see the primality post
            f[m] = f.get(m, 0) + 1
            return
        d = pollard_rho(m)
        rec(d)
        rec(m // d)
    # peel small primes first for speed
    for p in range(2, 1000):
        while n % p == 0:
            f[p] = f.get(p, 0) + 1
            n //= p
    rec(n)
    return f
```

### Complexity

- **Time:** expected $O(n^{1/4} \log n)$ to split $n$; the recursion adds a $\log n$ factor for the number of prime factors, plus a [Miller-Rabin](/citadel/cp/primality-tests) test ($O(\log^2 n)$) at each node.
- **Space:** $O(\log n)$ recursion depth.

## Approach 3: Fermat's method

### Intuition

Write $n = a^2 - b^2 = (a-b)(a+b)$. Start $a = \lceil \sqrt n \rceil$ and increase it until $a^2 - n$ is a perfect square. Fast **only** when $n$ has two factors close to $\sqrt n$; worst case (a small factor times a large one) it is worse than trial division.

```python
from math import isqrt

def fermat(n: int) -> tuple[int, int]:
    a = isqrt(n)
    if a * a < n:
        a += 1
    while True:
        b2 = a * a - n
        b = isqrt(b2)
        if b * b == b2:
            return a - b, a + b
        a += 1
```

Niche: useful when a problem guarantees two near-equal factors (e.g. an RSA-flavoured toy).

## Common pitfalls

- **No primality gate in the recursion.** After `pollard_rho` splits `m`, each part may still be composite. Test with Miller-Rabin and recurse; otherwise you emit composite "factors".
- **`abs(x - y)` becoming $0$.** If $x \equiv y \pmod n$ the product $q$ hits $0$ and $\gcd$ returns $n$. Break the block and restart with a fresh $c$.
- **The polynomial $x^2$ (i.e. $c = 0$) or $c = -2$.** Both have known degeneracies; pick $c \in [1, n)$, $c \ne 0$.
- **Perfect powers.** $n = p^k$ can make rho slow; a quick check for square/cube roots up front helps in adversarial settings.
- **$n = 1$ or $n$ prime.** Handle before calling rho — `factorize` above checks primality first.
- **`x*x` overflow.** Fine in Python. In C++ near $2^{63}$ you need `__int128` multiplication or Montgomery form.

## The keystone

Pollard's rho turns factorization into cycle detection: iterate $x^2 + c$ modulo $n$, and a collision modulo an unseen prime factor $p$ shows up as $\gcd(|x_i - x_j|, n)$ after only $O(p^{1/2}) = O(n^{1/4})$ steps. Paired with a [Miller-Rabin](/citadel/cp/primality-tests) gate to know when a part is already prime, it factors any 64-bit integer effectively instantly — which is what makes [divisor](/citadel/cp/divisors-count-and-sum) and [totient](/citadel/cp/eulers-totient-function) queries on big single numbers feasible.
