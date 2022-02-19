---
title: Computer Arithmetic - Adders, Multipliers, and IEEE 754
description: How a CPU adds, subtracts, multiplies, and divides - the full adder and ripple-carry chain, carry-lookahead for speed, shift-and-add and Booth multiplication, restoring and non-restoring division, and the IEEE 754 floating-point format with its arithmetic.
date: 2022-02-19
draft: false
slug: /computer-architecture/arithmetic
tags:
  - Computer Architecture
  - CPU
  - Floating Point
---

Addition looks free. It isn't. A 64-bit add has to know whether a carry out of bit 3 reaches bit 40, and the naive circuit computes that by letting the carry *ripple* through 64 gates in series — slow enough that fast adders are a whole subfield. Multiplication and division are worse, and floating point adds a layer of alignment, normalisation, and rounding on top.

This post walks the ALU's arithmetic: how carries are built and then sped up, how repeated addition becomes multiplication (and how Booth's trick handles negative numbers directly), how division mimics long division in hardware, and how [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) packs a huge range of reals into 32 or 64 bits.

## Integer addition and subtraction

### The full adder

At one bit position you have two operand bits $x_i, y_i$ and a carry-in $c_i$ from the position below. A **full adder** produces a sum bit and a carry-out:

$$s_i = x_i \oplus y_i \oplus c_i$$
$$c_{i+1} = (x_i \land y_i) \lor (x_i \land c_i) \lor (y_i \land c_i)$$

The sum is 1 when an odd number of the three inputs are 1; the carry is 1 when at least two are.

### Ripple-carry, and why it's slow

Chain $n$ full adders, feeding each carry-out into the next carry-in — a **ripple-carry adder (RCA)**:

```mermaid
flowchart RL
  FA0["FA 0"] -->|c1| FA1["FA 1"]
  FA1 -->|c2| FA2["FA 2"]
  FA2 -->|c3| FA3["FA 3"]
  c0["c0"] --> FA0
  FA3 -->|c4| OUT["carry out"]
```

Correct, and tiny. But bit $i$'s sum can't settle until $c_i$ arrives, and $c_i$ waited for every carry below it. The final carry of an $n$-bit RCA can take about $2n$ gate delays — linear in width.

### Subtraction and overflow, for free-ish

Two's-complement subtraction is addition: $X - Y = X + \overline{Y} + 1$. Build a combined adder/subtractor from an RCA plus one XOR per $Y$ bit, controlled by an `Add/Sub` line. When it's 0, $Y$ passes through and $c_0 = 0$ (plain add). When it's 1, the XORs invert $Y$ (one's complement) and $c_0 = 1$ supplies the `+1`.

**Overflow** in two's complement happens exactly when the carry *into* the sign bit differs from the carry *out* of it:

$$\text{Overflow} = c_n \oplus c_{n-1}$$

Equivalently: the operands share a sign but the result's sign differs.

## Fast adders: carry-lookahead

Ripple delay is unacceptable in a fast CPU. A **carry-lookahead adder (CLA)** computes the carries *ahead of time* from the input bits alone.

For each position, define **generate** and **propagate**:

$$G_i = x_i \land y_i \qquad P_i = x_i \oplus y_i$$

A carry is generated at $i$ if both inputs are 1; an incoming carry propagates through if exactly one is. Both depend only on $x_i, y_i$, so all $G_i$ and $P_i$ are available after one gate delay. The carry recurrence

$$c_{i+1} = G_i \lor (P_i \land c_i)$$

then unrolls into a flat expression in the inputs and $c_0$:

$$c_1 = G_0 \lor (P_0 \land c_0)$$
$$c_2 = G_1 \lor (P_1 G_0) \lor (P_1 P_0 c_0)$$
$$c_3 = G_2 \lor (P_2 G_1) \lor (P_2 P_1 G_0) \lor (P_2 P_1 P_0 c_0)$$

Each carry is now two levels of AND-OR logic over signals that are all ready at once. Total delay is roughly constant — about four gate delays — *regardless of $n$*.

The catch is **fan-in**: $c_{16}$ in pure form needs 17-input gates, which don't exist in practice. Real designs use **blocked carry-lookahead** — 4-bit CLA blocks, with a second level of lookahead computing carries *between* blocks. A 16-bit blocked CLA is far faster than a 16-bit RCA and only a little slower than the impractical flat version.

## Multiplication

### Shift-and-add

Hardware multiplication mirrors the by-hand method. Registers: multiplicand `M`, multiplier `Q`, accumulator `A` (starts at 0), an $n$-bit adder. Repeat $n$ times:

- if the low bit of `Q` is 1, `A ← A + M`;
- shift the double-width register pair `A:Q` right by one (the bit shifted out of `Q` is the next multiplier bit consumed).

After $n$ iterations the $2n$-bit product sits in `A:Q`. A purely combinational **array multiplier** does the same sums in space instead of time — $n$ rows of AND gates for the partial products, an array of full adders to sum them — trading gates for speed.

```python
def multiply_unsigned(m, q, n):
    """Shift-and-add. m, q are n-bit non-negative ints."""
    a = 0
    for _ in range(n):
        if q & 1:
            a += m
        # arithmetic-free right shift of the 2n-bit pair A:Q
        q = (q >> 1) | ((a & 1) << (n - 1))
        a >>= 1
    return (a << n) | q

assert multiply_unsigned(13, 11, 8) == 13 * 11        # 143
assert multiply_unsigned(255, 255, 8) == 255 * 255    # 65025
```

### Signed operands: Booth's algorithm

Plain shift-and-add assumes unsigned values; a negative multiplier breaks it. The simple fix is to negate both operands to positive, multiply, then negate the result if the original signs differed. **Booth's algorithm** handles two's-complement operands directly, and is faster on runs of 1s.

The idea: a block of $k$ consecutive 1s in the multiplier, worth $2^{j+k} - 2^{j}$, normally costs $k$ additions. Booth replaces it with one subtraction at the block's low end and one addition just past its high end. Scan `Q` from the low bit upward, looking at the pair $(q_i, q_{i-1})$ with an implicit $q_{-1} = 0$:

| $(q_i, q_{i-1})$ | action |
| --- | --- |
| `00` or `11` | nothing (inside a run) |
| `01` | add M (a run of 1s just started) |
| `10` | subtract M (a run of 1s just ended) |

between arithmetic *right* shifts of the partial product.

```python
def booth(m, q, n):
    """Booth's algorithm. m, q are n-bit two's-complement bit patterns."""
    def signed(v):
        return v - (1 << n) if v & (1 << (n - 1)) else v
    m = signed(m)
    a, q_1 = 0, 0                       # a: full-width signed partial product
    for _ in range(n):
        if (q & 1, q_1) == (0, 1):
            a += m
        elif (q & 1, q_1) == (1, 0):
            a -= m
        q_1 = q & 1                     # arithmetic right shift of A:Q:Q_-1
        q = (q >> 1) | ((a & 1) << (n - 1))
        a >>= 1                         # Python >> on a negative int is arithmetic
    return a * (1 << n) + q

for x in range(-8, 8):
    for y in range(-8, 8):
        assert booth(x & 0xF, y & 0xF, 4) == x * y, (x, y)
```

Booth's worst case is an alternating `0101…` multiplier, where every pair triggers an operation.

### Speeding the summation

Two more tricks matter for wide multipliers:

- **Bit-pair (modified Booth) recoding** looks at multiplier bits in overlapping groups of three to pick a summand of $0, \pm M, \pm 2M$ per *pair* of bits, halving the number of partial products.
- **Carry-save addition (CSA)** sums many partial products without rippling. One CSA level takes 3 input numbers and emits 2 (a sum vector and a carry vector) whose total equals the 3 inputs, in a single full-adder delay with no horizontal carry. Feed partial products through a tree of CSAs until two vectors remain, then add those once with a CLA.

## Integer division

The slowest basic operation. Setup resembles the sequential multiplier: divisor `M`, dividend in `Q`, remainder/accumulator `A`, an $(n{+}1)$-bit adder/subtractor.

**Restoring division** copies long division exactly. Repeat $n$ times: shift `A:Q` left one bit; `A ← A − M`; if `A` is now negative the subtraction "didn't fit", so set the new low bit of `Q` to 0 and **restore** `A` by adding `M` back; otherwise set it to 1. Quotient ends in `Q`, remainder in `A`.

**Non-restoring division** skips the add-back. Each step: if `A ≥ 0`, shift left then `A ← A − M`; if `A < 0`, shift left then `A ← A + M`. Set the new quotient bit from the resulting sign of `A`. One correction at the end (add `M` if the final remainder is negative). Faster, because there's no conditional restore inside the loop.

## Floating point

A floating-point value is scientific notation in binary:

$$\text{value} = (-1)^{S} \times \text{mantissa} \times 2^{\text{exponent}}$$

The mantissa is **normalised** so its single leading bit is 1 — $0.001101_2 \times 2^5$ becomes $1.101_2 \times 2^2$ — which makes the representation unique (zero excepted) and lets that leading 1 go unstored.

### IEEE 754

| | Sign | Exponent | Fraction |
| --- | --- | --- | --- |
| Single (32-bit) | 1 | 8, excess-127 | 23 |
| Double (64-bit) | 1 | 11, excess-1023 | 52 |

- The exponent is stored **biased**: $E' = E_{\text{true}} + \text{bias}$, so exponents compare as unsigned integers.
- The **hidden bit**: the normalised mantissa's leading `1.` is implicit, buying one extra bit of precision.
- Value: $(-1)^S \times (1.F) \times 2^{E' - \text{bias}}$.
- Reserved exponent patterns — all-zero and all-one — encode $\pm 0$, $\pm\infty$, and NaN, so the usable exponent range is slightly narrower than the field suggests.

### The operations

**Add / subtract** — the one that needs alignment:

1. compare exponents;
2. shift the *smaller*-exponent mantissa right until they match, keeping the shifted-out bits;
3. add or subtract the mantissas (hidden bits included);
4. fix the sign;
5. normalise the result, adjusting the exponent;
6. round back to field width, checking overflow/underflow.

**Multiply** — $E'_{\text{res}} = E'_1 + E'_2 - \text{bias}$ (the bias was added twice, so subtract one), multiply the mantissas, $S_{\text{res}} = S_1 \oplus S_2$, normalise (usually one right shift), round.

**Divide** — $E'_{\text{res}} = E'_1 - E'_2 + \text{bias}$, divide the mantissas, XOR the signs, normalise, round.

Add and subtract need mantissa alignment because you can only combine digits at the same scale; multiply and divide don't, because the significands combine directly and the exponents are handled separately by adding or subtracting them.

### Guard bits and rounding

Bits shifted off the mantissa's right end during alignment aren't discarded immediately — hardware keeps a **guard**, **round**, and **sticky** bit to preserve accuracy through the operation. Afterwards the result is rounded to field width. Plain **truncation** is simple but biases results low; IEEE's default is **round to nearest, ties to even** — nearest representable value, and on an exact tie, the one whose last bit is 0, which removes the statistical bias.

## The one idea to keep

Every arithmetic unit is a race against carry propagation. Addition speeds it up by precomputing carries (lookahead); multiplication avoids re-rippling by keeping sums in redundant carry-save form until the very end; division trades a restore step for a sign test. Floating point then wraps integer mantissa arithmetic in exponent bookkeeping — and the single asymmetry worth remembering is that addition must align exponents first while multiplication just adds them.
