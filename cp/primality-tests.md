---
title: Primality Tests - From Trial Division to Miller-Rabin
description: Testing whether one large number is prime — trial division for small inputs, the Fermat test and its Carmichael blind spot, and deterministic Miller-Rabin for everything up to 64 bits.
date: 2023-12-19
draft: false
slug: /cp/primality-tests
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

A [sieve](/citadel/cp/sieve-of-eratosthenes) answers "which numbers up to $n$ are prime" but needs $O(n)$ memory. When you have a single 18-digit number and want to know if it is prime, you need a test that works on one input in polylogarithmic time. Miller-Rabin is that test, and with a fixed set of witnesses it is *deterministic* for every 64-bit integer.

## The problem

Given one integer $n$ up to $\approx 10^{18}$, decide whether it is prime. Trial division is $O(\sqrt n)$ — around $10^9$ operations at the top of that range, too slow. We want $O(\log^k n)$.

## Approach 1: Trial division

### Intuition

If $n$ has a factor, it has one no larger than $\sqrt n$.

### Algorithm

1. Handle $n < 2$ (not prime) and $n = 2, 3$.
2. Reject even $n$ and multiples of $3$.
3. Test divisors of the form $6k \pm 1$ from $5$ up to $\sqrt n$.

```python
def is_prime_trial(n: int) -> bool:
    if n < 2:
        return False
    for p in (2, 3):
        if n % p == 0:
            return n == p
    i = 5
    while i * i <= n:
        if n % i == 0 or n % (i + 2) == 0:
            return False
        i += 6
    return True
```

### Complexity

- **Time:** $O(\sqrt n / \log n)$ with the wheel. Fine for $n \le 10^{12}$; unusable near $10^{18}$.
- **Space:** $O(1)$.

## Approach 2: Fermat test

### Intuition

Fermat's little theorem: if $n$ is prime and $\gcd(a, n) = 1$, then $a^{n-1} \equiv 1 \pmod n$. Pick random bases $a$; if any gives $a^{n-1} \not\equiv 1$, $n$ is definitely composite.

### The catch

The converse is false. **Carmichael numbers** ($561 = 3 \cdot 11 \cdot 17$, $1105$, $1729$, …) satisfy $a^{n-1} \equiv 1$ for *every* $a$ coprime to $n$, so the Fermat test calls them prime no matter how many bases you try. This makes the plain Fermat test unsafe for adversarial input.

```python
def fermat_test(n: int, rounds: int = 20) -> bool:
    import random
    if n < 4:
        return n in (2, 3)
    for _ in range(rounds):
        a = random.randrange(2, n - 1)
        if pow(a, n - 1, n) != 1:
            return False
    return True    # probably prime -- but fooled by Carmichael numbers
```

## Approach 3: Miller-Rabin

### Intuition

Strengthen Fermat with a fact that has *no* Carmichael-style exception: modulo a prime $p$, the only square roots of $1$ are $\pm 1$. Write $n - 1 = 2^s \cdot d$ with $d$ odd. If $n$ is prime, then for any base $a$ either

$$a^d \equiv 1 \pmod n, \quad \text{or} \quad a^{2^r d} \equiv -1 \pmod n \text{ for some } 0 \le r < s.$$

If neither holds, $a$ is a **witness** that $n$ is composite. For an odd composite $n$, at least $3/4$ of the bases in $[2, n-2]$ are witnesses, so $k$ random bases give error $\le 4^{-k}$.

### Deterministic witness sets

For $n < 3{,}317{,}044{,}064{,}679{,}887{,}385{,}961{,}981$ (well past $2^{64}$), testing the bases $\{2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37\}$ is *proven* to give the exact answer. A smaller set $\{2, 3, 5, 7, 11, 13, 17\}$ suffices for all $n < 3.4 \cdot 10^{14}$, and $\{2, 3, 5, 7\}$ for $n < 3.2 \cdot 10^{9}$.

### Algorithm

1. Small cases: $n < 2$ false; $n$ in $\{2, 3\}$ true; even $n$ false.
2. Factor out twos: $n - 1 = 2^s d$, $d$ odd.
3. For each base $a$ in the witness set (skip if $a \bmod n = 0$):
   - $x \leftarrow a^d \bmod n$. If $x \in \{1, n-1\}$, this base passes.
   - Repeat $s - 1$ times: $x \leftarrow x^2 \bmod n$; if $x = n - 1$, this base passes.
   - If no pass, return composite.
4. All bases passed ⇒ prime.

```python
def is_prime(n: int) -> bool:
    if n < 2:
        return False
    for p in (2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37):
        if n % p == 0:
            return n == p
    d = n - 1
    s = 0
    while d % 2 == 0:
        d //= 2
        s += 1
    for a in (2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37):
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(s - 1):
            x = x * x % n
            if x == n - 1:
                break
        else:
            return False
    return True
```

### Complexity

- **Time:** $O(k \log^2 n)$ with $k$ fixed at 12 bases and each [modular exponentiation](/citadel/cp/binary-exponentiation) costing $O(\log n)$ multiplications of $O(\log n)$-bit numbers.
- **Space:** $O(1)$.

## Common pitfalls

- **Trusting the Fermat test on unknown input.** Carmichael numbers pass it for all coprime bases; use Miller-Rabin.
- **Wrong witness set for the range.** Using $\{2, 3, 5, 7\}$ for a number above $3.2 \cdot 10^9$ can give a false "prime". Match the set to the bound, or use the full 12-base set and stop worrying.
- **`x * x % n` overflow.** In Python, fine. In C++ with $n$ near $2^{63}$ you need `__int128` or Montgomery multiplication.
- **Not short-circuiting small prime factors.** If $a \bmod n = 0$ (e.g. $n = 3$ with base $3$), the witness loop misbehaves; the leading small-prime check handles it.
- **`for ... else`.** The `else` on the inner `for` runs only if no `break` fired — that is the "no square root of $-1$ found, so composite" path. Easy to get wrong when porting.

## The keystone

Miller-Rabin adds one fact to the Fermat test — a prime modulus has no nontrivial square root of $1$ — and that fact has no Carmichael exception, so a fixed dozen bases decide primality exactly for every 64-bit integer in $O(\log^2 n)$ per base. Once you can test primality this cheaply, [Pollard's rho](/citadel/cp/integer-factorization) turns it into full factorization of the same-sized numbers.
