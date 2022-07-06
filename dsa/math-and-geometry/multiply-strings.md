---
title: Multiply Strings - Grade-School Multiplication in an Array
description: Multiplying two non-negative integers given as strings without big-integer types — accumulate every digit-pair product into position i plus j plus 1 of a result array, then normalise carries.
date: 2022-07-06
draft: false
slug: /dsa/multiply-strings
tags:
  - Competitive Programming
  - Math & Geometry
  - Strings
---

The product of `num1[i]` and `num2[j]` lands in result positions `i + j` (carry) and `i + j + 1` (units). Accumulate all `m * n` partial products into a single integer array of length `m + n`, then sweep once to push carries and drop the leading zero.

## Description

Given two non-negative integers `num1` and `num2` as strings, return their product as a string. Do not use built-in big-integer libraries or convert the inputs directly to integers.

**Example**

```
Input:  num1 = "2", num2 = "3"
Output: "6"

Input:  num1 = "123", num2 = "456"
Output: "56088"
```

**Constraints**

- $1 \le \text{num1.length}, \text{num2.length} \le 200$
- Both contain only digits, with no leading zero except `"0"` itself.

## Prerequisites

- The position rule: `num1[i] * num2[j]` contributes to `res[i + j]` and `res[i + j + 1]`.
- Carry normalisation and leading-zero trimming.

## Approach 1: Positional accumulation array

### Intuition

Let `res` be `[0] * (m + n)`. For each `i` (from the right of `num1`) and `j` (from the right of `num2`), add `d1 * d2` to `res[i + j + 1]`, carrying overflow into `res[i + j]`. A final pass is not even needed if you carry as you go, but a clean carry sweep afterward is simplest to reason about.

### Algorithm

1. If either input is `"0"`, return `"0"`.
2. `m, n = len(num1), len(num2)`; `res = [0] * (m + n)`.
3. For `i` from `m - 1` down to `0`, `j` from `n - 1` down to `0`:
   - `mul = (ord(num1[i]) - 48) * (ord(num2[j]) - 48)`.
   - `total = mul + res[i + j + 1]`.
   - `res[i + j + 1] = total % 10`; `res[i + j] += total // 10`.
4. Skip leading zeros in `res`; join the rest. If empty, return `"0"`.

```python
def multiply(num1: str, num2: str) -> str:
    if num1 == "0" or num2 == "0":
        return "0"
    m, n = len(num1), len(num2)
    res = [0] * (m + n)

    for i in range(m - 1, -1, -1):
        d1 = ord(num1[i]) - 48
        for j in range(n - 1, -1, -1):
            d2 = ord(num2[j]) - 48
            total = d1 * d2 + res[i + j + 1]
            res[i + j + 1] = total % 10
            res[i + j] += total // 10

    k = 0
    while k < len(res) - 1 and res[k] == 0:
        k += 1
    return "".join(str(d) for d in res[k:])
```

### Complexity

- **Time:** $O(m \cdot n)$.
- **Space:** $O(m + n)$.

## Common Pitfalls

- **Wrong result index.** `num1[i] * num2[j]` goes to `i + j + 1` (units) with carry to `i + j`. Off-by-one here shifts the whole product.
- **Adding to `res[i+j+1]` without folding in its existing value first.** Include `res[i + j + 1]` in `total` before taking `% 10`, or you lose earlier contributions.
- **Not short-circuiting on `"0"`.** Otherwise you return `"0000"` or an empty string depending on the trim.
- **Trimming away everything.** Keep at least one digit: stop the leading-zero skip at `len(res) - 1`.

## The keystone

Long multiplication is a double loop of digit products dropped into fixed positions, followed by carry normalisation — the array *is* the number. It generalises [Plus One](/citadel/dsa/plus-one)'s single carry to a full grid of partial products. This closes [Math & Geometry](/citadel/dsa/math-and-geometry); [Detect Squares](/citadel/dsa/detect-squares) shifts from arithmetic to coordinate counting.
