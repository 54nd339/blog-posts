---
title: Arbitrary-Precision Arithmetic - Big Integers From Scratch
description: How a bignum is stored as a vector of base-1e9 limbs, schoolbook add and multiply on that representation, division by a small integer, and why Python contestants only need this for the ideas.
date: 2024-02-11
draft: false
slug: /cp/arbitrary-precision-arithmetic
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

Some answers do not fit in 64 bits: $100!$, the $500$th Fibonacci number, the exact product of two $200$-digit inputs. Python hands you this for free — `int` is already arbitrary precision — so this post is about *how* a bignum library works, which matters when you port to C++ or need to reason about the cost of a big-integer step.

## The problem

Represent an integer of unbounded size and support $+$, $-$, $\times$, and division by a small integer, plus reading and printing in base 10.

## The representation

Store the number as a list of **limbs**: digits in a large base $B$, least significant first, with the sign kept separately. Base $B = 10^9$ is the usual contest choice — it fits in a 32-bit slot, two limbs multiply without overflowing 64 bits, and printing is easy because $B$ is a power of $10$.

```python
BASE = 10 ** 9

def from_int(x: int) -> list[int]:
    a = []
    while x:
        a.append(x % BASE)
        x //= BASE
    return a or [0]                       # least significant limb first

def to_str(a: list[int]) -> str:
    s = str(a[-1])
    for limb in reversed(a[:-1]):
        s += f"{limb:09d}"                # pad every non-leading limb
    return s
```

## Addition and subtraction

Schoolbook, limb by limb, propagating a carry (or borrow):

```python
def add(a: list[int], b: list[int]) -> list[int]:
    res, carry = [], 0
    for i in range(max(len(a), len(b))):
        cur = carry
        if i < len(a): cur += a[i]
        if i < len(b): cur += b[i]
        res.append(cur % BASE)
        carry = cur // BASE
    if carry:
        res.append(carry)
    return res
```

Subtraction (assuming $a \ge b$) is the same with a borrow: `cur = a[i] - borrow - (b[i] if i < len(b) else 0)`, then `borrow = 1` and `cur += BASE` when `cur < 0`; strip trailing zero limbs at the end.

Cost: $O(n)$ for $n$-limb operands.

## Multiplication

Schoolbook is the double loop: every limb of $a$ times every limb of $b$, accumulated at the right offset in a 64-bit-wide buffer, then a single carry-normalisation pass.

```python
def mul(a: list[int], b: list[int]) -> list[int]:
    res = [0] * (len(a) + len(b))
    for i, ai in enumerate(a):
        carry = 0
        for j, bj in enumerate(b):
            cur = res[i + j] + ai * bj + carry
            res[i + j] = cur % BASE
            carry = cur // BASE
        res[i + len(b)] += carry
    while len(res) > 1 and res[-1] == 0:
        res.pop()
    return res
```

Cost: $O(n^2)$. [Karatsuba](/citadel/algorithms/Karatsuba) drops it to $O(n^{1.585})$ by trading one of four half-size multiplies for additions, and [FFT / NTT](/citadel/algorithms/FastFourierTransform) reach $O(n \log n)$ — the crossover to Karatsuba is around a few hundred limbs, to FFT around a few thousand.

## Division by a small integer

Long division with the divisor $d$ fitting in a machine word: sweep limbs from most significant, carrying the remainder.

```python
def divmod_small(a: list[int], d: int) -> tuple[list[int], int]:
    res = [0] * len(a)
    rem = 0
    for i in range(len(a) - 1, -1, -1):
        cur = rem * BASE + a[i]
        res[i] = cur // d
        rem = cur % d
    while len(res) > 1 and res[-1] == 0:
        res.pop()
    return res, rem
```

Full bignum-by-bignum division (Knuth's Algorithm D) is fiddlier — estimate each quotient digit, multiply back, correct by at most two — and is the one routine most people are glad Python provides.

## Complexity summary

| operation | schoolbook | with fast multiply |
| --- | --- | --- |
| add / subtract | $O(n)$ | $O(n)$ |
| multiply | $O(n^2)$ | $O(n \log n)$ (FFT) |
| divide by small $d$ | $O(n)$ | $O(n)$ |
| divide bignum / bignum | $O(n^2)$ | $O(n \log n)$ |
| decimal I/O (base $10^9$) | $O(n)$ | $O(n)$ |

## Common pitfalls

- **Limb padding on output.** Only the most significant limb prints without leading zeros; every other limb is zero-padded to $9$ digits (`f"{x:09d}"`). Forgetting this silently drops zeros.
- **Not normalising.** After subtraction or division, strip trailing zero limbs, but keep at least one so "zero" is `[0]`, not `[]`.
- **Base too large in C++.** $B = 10^{18}$ overflows when two limbs multiply. $10^9$ (or $2^{32}$ with a $2^{64}$ accumulator) is the safe choice.
- **Sign handling.** Keep the sign as a separate flag and dispatch add/subtract by comparing magnitudes; do not try to carry a sign through the limb array.
- **Reaching for this in Python at all.** For plain big-integer answers, use `int`. Write a limb array only for a specific need — e.g. per-digit access, or a custom base.

## The keystone

A big integer is a little-endian vector of base-$10^9$ limbs plus a sign, and $+$, $-$, $\times$ are the grade-school algorithms on that vector: $O(n)$, $O(n)$, $O(n^2)$, with [Karatsuba](/citadel/algorithms/Karatsuba) and [FFT](/citadel/algorithms/FastFourierTransform) speeding up the product. In Python you get all of it as `int`; the value of knowing the internals is porting to C++ and cost-modelling a big-number step.
