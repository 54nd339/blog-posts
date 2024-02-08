---
title: Balanced Ternary - Base Three With Digits Minus One, Zero, One
description: A positional system whose digits are -1, 0, +1, why it represents every integer uniquely with no sign bit, the conversion from ordinary base 3, and the weighing-puzzle it solves in one line.
date: 2024-02-08
draft: false
slug: /cp/balanced-ternary
tags:
  - Competitive Programming
  - Algebra
  - Number Systems
---

Ordinary base 3 uses digits $0, 1, 2$. Balanced ternary uses $-1, 0, +1$ (often written $\text{-}, 0, +$ or $\text{T}, 0, 1$). The payoff: negative numbers need no sign — flipping every digit's sign negates the value — and rounding is just truncation. It shows up in weighing puzzles and the occasional constructive problem.

## The problem

Represent an integer $n$ as $\sum_i d_i \, 3^i$ with each $d_i \in \{-1, 0, +1\}$, and convert back. Every integer has exactly one such representation (ignoring leading zeros).

Example: $n = 5 = 9 - 3 - 1 = 1 \cdot 3^2 + (\text{-}1)\cdot 3^1 + (\text{-}1)\cdot 3^0$, so $5 = (1, \text{-}1, \text{-}1)_{\text{bal3}}$, written "$1\text{TT}$". And $-5$ is "$\text{T}11$" — the same digits negated.

## The idea

Convert like any base, but after taking $r = n \bmod 3$, if $r = 2$ borrow: emit digit $-1$ and carry $+1$ into the next position (because $2 = 3 - 1$). So:

- $r = 0$ → digit $0$, $n \leftarrow n/3$.
- $r = 1$ → digit $+1$, $n \leftarrow (n-1)/3$.
- $r = 2$ → digit $-1$, $n \leftarrow (n+1)/3$.

Negative $n$ works with the same rule if you use a remainder in $\{0, 1, 2\}$ (Python's `%` already does this).

## How it works

$n = 5$: $5 \bmod 3 = 2$ → digit $\text{-}1$, $n \leftarrow 6/3 = 2$. $2 \bmod 3 = 2$ → digit $\text{-}1$, $n \leftarrow 3/3 = 1$. $1 \bmod 3 = 1$ → digit $+1$, $n \leftarrow 0$. Digits low-to-high: $(\text{-}1, \text{-}1, +1)$, i.e. "$1\text{TT}$". Check: $9 - 3 - 1 = 5$.

## Algorithm

```python
def to_balanced_ternary(n: int) -> list[int]:
    digits = []
    while n != 0:
        r = n % 3
        if r == 2:
            digits.append(-1)
            n = (n + 1) // 3
        else:
            digits.append(r)          # 0 or 1
            n //= 3
    return digits or [0]              # least significant first

def from_balanced_ternary(digits: list[int]) -> int:
    value = 0
    for d in reversed(digits):
        value = value * 3 + d
    return value
```

## Where it is useful

- **The 4-weight puzzle.** With weights $1, 3, 9, 27$ grams and a two-pan balance, you can weigh any integer mass from $1$ to $40$: the balanced-ternary digits of the target say which pan each weight goes in ($+1$ = opposite the object, $-1$ = same pan as the object, $0$ = unused).
- **Symmetric rounding.** Truncating a balanced-ternary fraction rounds to the nearest integer (ties away from zero handled naturally), because dropped digits contribute at most $\sum_{i\ge1} 3^{-i} = 1/2$.
- **Sign-free arithmetic.** Negation is digit-wise sign flip; comparison is lexicographic from the top digit. Handy in a few constructive problems where you must emit $\pm$ contributions summing to a target.

## Common pitfalls

- **Using a signed remainder.** If your language's `%` can return $-1$ or $-2$, normalise to $\{0, 1, 2\}$ first (`r = n % 3` then `r += 3` if negative), or the carry logic breaks for negative $n$.
- **Digit order.** The loop produces least-significant first; reverse before printing as a string.
- **Empty representation of $0$.** The `while n != 0` loop emits nothing for $n = 0$; return `[0]` explicitly.
- **Reading a string.** Map characters `'T'/'0'/'1'` (or `'-'/'0'/'+'`) to $-1, 0, 1$ before evaluating.

## The keystone

Balanced ternary is base 3 with the digit set shifted to $\{-1, 0, 1\}$, obtained by turning every remainder of $2$ into a $-1$ digit and a carry. Uniqueness plus sign-free negation is what makes it the natural language for two-pan weighing and a handful of "emit $\pm$ terms summing to $n$" constructions.
