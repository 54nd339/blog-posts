---
title: Karatsuba Multiplication - Three Half-Size Multiplies Instead of Four
description: Schoolbook multiplication of two n-digit numbers costs n^2 digit multiplies. Karatsuba splits each number in half and, with one algebraic trick, needs only three half-size multiplies instead of four, giving about O(n^1.585).
date: 2021-02-26
draft: false
slug: /algorithms/Karatsuba
tags:
  - Algorithms
  - Divide and Conquer
---

The multiplication you learned in school multiplies every digit of one number by every digit of the other: about $n^2$ single-digit products for two $n$-digit numbers. That's fine for small numbers, but cryptography (RSA keys), computer algebra, and high-precision arithmetic multiply numbers with thousands or millions of digits, and $n^2$ is too slow.

In 1960 Anatoly Karatsuba found that you don't need four half-size multiplications to combine two halves — three is enough. That one saving, applied recursively, drops the exponent from 2 to about 1.585.

## The naive split

Write two $n$-digit numbers by cutting each in half at $m = n/2$ digits:

$$x = a \cdot 10^m + b, \qquad y = c \cdot 10^m + d$$

Then

$$x \cdot y = ac \cdot 10^{2m} + (ad + bc) \cdot 10^m + bd.$$

Multiplication by a power of 10 is just a shift, and additions are cheap. The expensive part is the four products $ac$, $ad$, $bc$, $bd$ — each a multiplication of $m$-digit numbers. Recursing on those four gives $T(n) = 4T(n/2) + O(n)$, which solves to $O(n^2)$: no gain.

## Karatsuba's trick

You need $ac$ and $bd$ anyway. The middle term $ad + bc$ falls out of one more product:

$$(a + b)(c + d) = ac + ad + bc + bd \quad\Longrightarrow\quad ad + bc = (a+b)(c+d) - ac - bd.$$

So compute three products:

$$P_1 = ac, \qquad P_2 = bd, \qquad P_3 = (a+b)(c+d),$$

and assemble

$$x \cdot y = P_1 \cdot 10^{2m} + (P_3 - P_1 - P_2) \cdot 10^m + P_2.$$

Three half-size multiplications, plus additions, subtractions, and shifts.

## The code

```python
def karatsuba(x, y):
    if x < 10 or y < 10:            # base case: one-digit multiply
        return x * y

    m = max(x.bit_length(), y.bit_length()) // 2   # split roughly in half
    base = 1 << m                   # working in base 2^m is fine too

    a, b = divmod(x, base)          # x = a*base + b
    c, d = divmod(y, base)          # y = c*base + d

    p1 = karatsuba(a, c)
    p2 = karatsuba(b, d)
    p3 = karatsuba(a + b, c + d)

    return (p1 << (2 * m)) + ((p3 - p1 - p2) << m) + p2


assert karatsuba(31415926, 27182818) == 31415926 * 27182818
assert karatsuba(2**200 + 1, 2**200 - 1) == (2**200 + 1) * (2**200 - 1)
```

Splitting at a power of two rather than a power of ten just means "base $2^m$"; the algebra is identical and the shifts are bit-shifts.

## Cost

The recurrence is

$$T(n) = 3T(n/2) + O(n),$$

three recursive calls on half-size inputs plus linear-time adds and shifts. By the master theorem this is $O(n^{\log_2 3}) \approx O(n^{1.585})$. The recursion halves the input each level, so the call stack is $O(\log n)$ deep.

## When it's worth it

CPUs multiply register-sized integers in one instruction, so Karatsuba only helps for **bignums** past hardware width — and even then, the linear-time overhead per level means implementations switch back to schoolbook below a threshold of a few dozen digits. Beyond Karatsuba, Toom–Cook generalises the split (more pieces, more savings), and [FFT-based multiplication](/citadel/algorithms/FastFourierTransform) (Schönhage–Strassen) reaches $O(n \log n \log \log n)$ for very large $n$. The same "spend an addition to save a multiplication" idea is exactly what [Strassen's algorithm](/citadel/algorithms/Strassen) does for matrices.
