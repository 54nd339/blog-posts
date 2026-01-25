---
title: Crunching the Numbers - Inside CPU Arithmetic
description: Exploring how computers perform addition, subtraction, multiplication, and division for both integers and floating-point numbers, including fast adder designs and standard representations.
date: 2023-04-28
draft: false
slug: /pensieve/coa/arithmetic
tags:
  - Computer Architecture
  - CS Basics
---

Hey number crunchers! We've seen the CPU's overall structure and control mechanisms. Now, let's zoom into one of its most crucial parts: the Arithmetic and Logic Unit (ALU), specifically focusing on how it performs arithmetic. From simple addition to complex floating-point operations, understanding these processes reveals the computational power of the processor.

## Integer Addition and Subtraction: The Foundation

Binary addition is the bedrock of CPU arithmetic.

### Basic Adder Logic: The Full Adder (FA)

At the single-bit level, addition requires considering two operand bits ($x_i, y_i$) and a carry-in bit ($c_i$) from the previous position. This is done by a **Full Adder (FA)**, which produces a sum bit ($s_i$) and a carry-out bit ($c_{i+1}$).

The logic equations are:
$s_i = x_i \oplus y_i \oplus c_i$
$c_{i+1} = (x_i \land y_i) \lor (x_i \land c_i) \lor (y_i \land c_i)$

### Ripple-Carry Adder (RCA): Chaining Full Adders

To add n-bit numbers, n Full Adders are connected in cascade. The carry-out ($c_{i+1}$) from one stage becomes the carry-in ($c_i$) for the next stage. This is called a **Ripple-Carry Adder (RCA)**.

![Ripple-Carry Adder (RCA)](https://www.gatevidyalay.com/wp-content/uploads/2018/06/4-bit-Ripple-Carry-Adder.png?ezimgfmt=ng:webp/ngcb1)

* **Speed Limitation:** The drawback is speed. The sum and carry bits for higher-order positions depend on the carry propagating ("rippling") through all the lower-order stages. The final carry-out ($c_n$) might take $2n$ gate delays to become stable, making RCAs slow for large 'n'.

### Adder/Subtractor Combo

Subtraction (X - Y) in 2's complement is achieved by adding the 2's complement of Y to X. Recall that 2's complement(Y) = 1's complement(Y) + 1.
We can build a combined adder/subtractor using an RCA:

1. Use an `Add/Sub` control line.
2. If `Add/Sub = 0` (Add): The Y inputs go directly to the FAs, and the initial carry-in $c_0$ is 0.
3. If `Add/Sub = 1` (Subtract): The Y inputs pass through XOR gates (controlled by `Add/Sub`) which inverts them (creating the 1's complement), and the initial carry-in $c_0$ is set to 1 (adding the required +1).

### Overflow Detection

In 2's complement, overflow occurs if:

* Adding two positive numbers results in a negative number.
* Adding two negative numbers results in a positive number.
  This happens precisely when the carry-in to the most significant bit (MSB) stage ($c_{n-1}$) differs from the carry-out of the MSB stage ($c_n$).
  Logic: `Overflow = c_n \oplus c_{n-1}`.
  Alternatively, overflow occurs if the signs ($x_{n-1}$, $y_{n-1}$) of the operands are the same, but the sign of the sum ($s_{n-1}$) is different.

## Design of Fast Adders: Beyond Ripple-Carry

The delay caused by carry propagation in RCAs is unacceptable for high-performance processors. **Carry-Lookahead Adders (CLA)** tackle this.

### Carry-Lookahead Logic

The core idea is to calculate the carries *in advance* based on the input bits, without waiting for them to ripple.

1. **Generate ($G_i$) and Propagate ($P_i$) signals:** For each bit position `i`:
   * $G_i = x_i \land y_i$ (A carry is *generated* here if both inputs are 1)
   * $P_i = x_i \oplus y_i$ (A carry-in *propagates* through if exactly one input is 1)
     *(Note: Sometimes $P_i = x_i \lor y_i$ is used, logic slightly differs)*
     Crucially, $G_i$ and $P_i$ depend *only* on $x_i$ and $y_i$ and can be computed in parallel for all `i` in one gate delay.
2. **Carry Equations:** The carry-out for stage `i` can be expressed using G and P:
   $c_{i+1} = G_i \lor (P_i \land c_i)$
3. **Lookahead Expansion:** We can recursively expand this:
   * $c_1 = G_0 \lor (P_0 \land c_0)$
   * $c_2 = G_1 \lor (P_1 \land c_1) = G_1 \lor (P_1 \land G_0) \lor (P_1 \land P_0 \land c_0)$
   * $c_3 = G_2 \lor (P_2 \land c_2) = G_2 \lor (P_2 \land G_1) \lor (P_2 \land P_1 \land G_0) \lor (P_2 \land P_1 \land P_0 \land c_0)$
     ...and so on.
4. **Speed:** Notice that each carry $c_i$ now depends only on the input bits ($x, y$) and the initial carry $c_0$. These equations can be implemented with two levels of AND-OR logic. Therefore, after $G_i$ and $P_i$ are computed (1 delay), all carries ($c_1$ to $c_n$) can be computed in 2 more gate delays. The sum bits ($s_i = P_i \oplus c_i$) take one more delay. Total delay is roughly constant (e.g., 4 gate delays), regardless of 'n'!

### Practical Limits and Blocked CLA

* **Fan-in Problem:** For large 'n', the AND and OR gates in the pure CLA logic require a very large number of inputs (fan-in), which is impractical.
* **Blocked Carry-Lookahead:** A common solution is to build the adder using blocks (e.g., 4-bit CLAs). Carry-lookahead logic is used *within* each block, and additional lookahead logic is used to quickly generate the carries *between* the blocks. This creates a hierarchical structure that balances speed and complexity. A 16-bit adder built from 4-bit CLA blocks is significantly faster than a 16-bit RCA, though slightly slower than a theoretical (impractical) 16-bit pure CLA.

## Multiplication of Positive Numbers

Multiplication is more complex than addition.

### Combinational Array Multiplier

This approach mimics manual multiplication. For an n x n multiplication:

1. Use n rows of n AND gates to compute the n partial products (each is either 0 or the multiplicand, shifted).
2. Use an array of Full Adders to sum the partial products.
   This is conceptually simple but uses many gates and can be slow due to carry propagation through the adders for large 'n'.

### Sequential Circuit Multiplier

This is a more common and resource-efficient hardware implementation:

* **Components:** An n-bit adder, registers for the Multiplicand (M), Multiplier (Q), and an Accumulator (A, initially 0), plus a control sequencer.
* **Algorithm (Shift-and-Add):**
  1. Initialize A to 0. Load M and Q.
  2. Repeat n times:
     * Check the least significant bit (LSB) of Q ($q_0$).
     * If $q_0 = 1$, add M to A (`A ← A + M`).
     * Shift the combined register pair A and Q one bit to the right (logical shift for A, bringing carry-out from adder or MSB of A into A's MSB; $q_0$ is shifted out).
  3. The final 2n-bit product is in the A and Q registers.

## Signed Operand Multiplication

Multiplying signed (2's complement) numbers requires modifications.

### The Problem with Negative Numbers

The basic shift-and-add algorithm assumes unsigned numbers. If the multiplicand is negative, simply shifting and adding doesn't work correctly unless you perform careful sign extension of the partial products, which complicates the hardware. If the multiplier is negative, the basic algorithm fails completely.

### Simple Solution

If either operand is negative, complement it (take 2's complement) to make it positive. Perform unsigned multiplication. If the original signs were different, complement the final result.

### Booth Algorithm

This elegant algorithm handles both positive and negative 2's complement operands directly and can sometimes be faster than the basic method.

* **Motivation:** Strings of 1s in the multiplier normally require multiple additions. For example, multiplying by $01110_2 (=14)$ involves adding M shifted left once, twice, and three times. But $14 = 16 - 2$, or $10000_2 - 00010_2$. So, we could instead *subtract* M shifted once and *add* M shifted four times. Booth algorithm generalizes this.
* **Recoding:** Scan the multiplier Q from right ($q_0$) to left, looking at pairs of bits ($q_i, q_{i-1}$). Assume an implicit $q_{-1} = 0$.
  * If ($q_i, q_{i-1}$) is `00` or `11`: Do nothing (add 0 to partial product).
  * If ($q_i, q_{i-1}$) is `01`: Add the multiplicand M (+1 * M). (Start of a string of 1s).
  * If ($q_i, q_{i-1}$) is `10`: Subtract the multiplicand M (-1 * M, i.e., add 2's complement of M). (End of a string of 1s).
* **Implementation:** Usually implemented sequentially similar to the basic shift-and-add, but uses the Booth recoding to decide whether to add M, subtract M, or do nothing before the arithmetic right shift of the partial product (A) and multiplier (Q).
* **Performance:** It excels on strings of 1s (including negative numbers which often end in many 1s). Its worst-case (alternating 0s and 1s) can require more operations than the basic method.

## Fast Multiplication Techniques

### Bit-Pair Recoding (Modified Booth)

Extends Booth by examining multiplier bits in groups of three ($q_{i+1}, q_i, q_{i-1}$) to determine the operation for bit pair ($q_i, q_{i-1}$). This allows selecting summands of 0, +/-M, or +/-2M (which is just M shifted left by 1). This effectively reduces the number of partial products to be summed by half.

### Carry-Save Addition (CSA)

When multiplying large numbers, many partial products must be summed. Instead of using slow RCAs or complex CLAs for intermediate sums, Carry-Save Addition is used.

* **Concept:** A CSA level takes 3 input numbers and produces 2 output numbers (a Sum vector and a Carry vector) such that Sum + Carry equals the original 3 inputs. This operation takes only one Full Adder delay, *without* horizontal carry propagation.
* **Process:** Partial products are grouped in threes, fed into CSA units. The resulting Sum and Carry vectors are again grouped in threes and fed into the next CSA level. This continues until only two vectors remain. These final two vectors are then added using a fast adder (like CLA) to get the final product. This significantly speeds up the summation of many partial products.

## Integer Division

Division is typically the slowest basic integer operation.

### Hardware Setup

Often uses a setup similar to the sequential multiplier: registers for Divisor (M), Dividend (initially in Q, sometimes combined with A), Remainder/Accumulator (A), and an (n+1)-bit adder/subtractor.

### Restoring Division Algorithm

Mimics manual division closely:

1. Initialize A to 0 (or MSBs of dividend), Q holds dividend.
2. Repeat n times:
   * Shift A and Q left one bit (bringing in next dividend bit into $q_0$).
   * Subtract Divisor M from A (`A ← A - M`).
   * If A is now positive (sign bit = 0): The subtraction was successful. Set the new $q_0$ to 1.
   * If A is now negative (sign bit = 1): The subtraction failed. Set the new $q_0$ to 0, and **restore** A by adding M back (`A ← A + M`).
3. The quotient is in Q, the remainder is in A.

### Non-Restoring Division Algorithm

Avoids the potentially time-consuming restoration step (adding M back):

1. Initialize A and Q.
2. Repeat n times:
   * If the sign of A is 0: Shift A and Q left, then subtract M from A (`A ← A - M`).
   * If the sign of A is 1: Shift A and Q left, then *add* M to A (`A ← A + M`).
   * Now, set the new $q_0$ to 1 if the *new* sign of A is 0, otherwise set $q_0$ to 0.
3. After n iterations, the quotient is in Q. If the final remainder in A is negative, a final correction step (adding M to A) is needed to get the true remainder.
   This method is generally faster as it avoids the conditional add-back step within the loop.

## Floating-Point Numbers and Operations

Representing numbers with fractional parts and very wide ranges requires a different approach than fixed-point integers.

### Floating-Point Representation

Based on scientific notation:
Value = $sign \times mantissa \times base^{exponent}$

* **Components:**
  * **Sign:** Positive or negative (usually 1 bit).
  * **Mantissa (or Significand/Fraction):** The significant digits of the number. Usually stored in a normalized form.
  * **Exponent:** Determines the scale or magnitude (how far the base's point is shifted).
  * **Base:** The number system base (usually 2 for computers, sometimes 16). Often implied, not stored explicitly.
* **Normalization:** To maximize precision, the mantissa is typically normalized. For binary base=2, this means shifting the mantissa left until its most significant bit is 1, adjusting the exponent accordingly. Example: $0.001101_2 \times 2^5$ becomes $1.101_2 \times 2^2$. Normalization ensures a unique representation for each number (except zero).

### IEEE 754 Standard

This is the universal standard for floating-point representation and arithmetic. Key features:

* **Formats:** Single Precision (32 bits) and Double Precision (64 bits).
* **Fields:**
  * Sign (S): 1 bit (0=positive, 1=negative).
  * Exponent (E): 8 bits (single) or 11 bits (double). Stored using **Excess (or Biased) Notation**. For single precision, it's Excess-127 ($E' = E_{true} + 127$). For double, Excess-1023. This makes comparing exponents easier (treat them as unsigned).
  * Mantissa (M or Fraction F): 23 bits (single) or 52 bits (double). Stores the fractional part of the normalized significand.
* **Hidden Bit:** Because binary normalized mantissas always start with '1.' (e.g., $1.fraction$), this leading '1' is *not* stored explicitly (it's hidden or implicit). This gives an extra bit of precision.
* **Value:** The value of an IEEE 754 number is: $(-1)^S \times (1 . M) \times 2^{(E' - Bias)}$
* **Special Values:** Specific exponent patterns (all 0s or all 1s) are reserved to represent +/- Zero, +/- Infinity, and NaN (Not a Number). This means the actual range of representable exponents is slightly smaller than the full range of the exponent field.

### Floating-Point Operations

Performing arithmetic requires handling the sign, exponent, and mantissa separately.

* **Addition/Subtraction:**
  1. Compare exponents.
  2. Shift the mantissa of the number with the *smaller* exponent right until exponents match. Keep track of shifted-out bits (guard bits).
  3. Add/subtract the mantissas (and the hidden bit).
  4. Determine the sign of the result.
  5. Normalize the resulting mantissa (shift left/right and adjust exponent) if necessary.
  6. Round the mantissa back to the standard number of bits using the guard bits. Check for overflow/underflow based on the final exponent.
* **Multiplication:**
  1. Add the biased exponents and subtract the bias once ($E'_{res} = E'_1 + E'_2 - Bias$).
  2. Multiply the mantissas (including the hidden bits).
  3. Determine the sign ($S_{res} = S_1 \oplus S_2$).
  4. Normalize the resulting mantissa (likely a single right shift, adjusting exponent).
  5. Round the mantissa. Check for overflow/underflow.
* **Division:**
  1. Subtract the biased exponents and add the bias once ($E'_{res} = E'_1 - E'_2 + Bias$).
  2. Divide the mantissas (including hidden bits).
  3. Determine the sign ($S_{res} = S_1 \oplus S_2$).
  4. Normalize the resulting mantissa.
  5. Round the mantissa. Check for overflow/underflow.

### Guard Bits and Rounding

* **Guard Bits:** When aligning mantissas for addition/subtraction, bits can be shifted off the right end. To maintain accuracy, hardware uses extra bits (guard bits, round bit, sticky bit) to hold these intermediate values.
* **Rounding:** After the operation, the result (with guard bits) must be rounded back to the standard mantissa size. Common methods include:
  * **Chop (Truncate):** Simply discard extra bits (introduces bias).
  * **Round to Nearest, Ties to Even (IEEE Default):** Rounds to the nearest representable value. If exactly halfway, rounds to the value with an LSB of 0 (avoids statistical bias).

**(Think:** Why does floating-point addition/subtraction require mantissa alignment, but multiplication/division doesn't?)
*Answer: Addition/subtraction requires operands to have the same scale (exponent) before combining significant digits. Multiplication/division works directly on the significant digits, and the scales (exponents) are handled separately by adding/subtracting them.*

## Conclusion: The Art of Calculation

Computer arithmetic forms the foundation of all numerical computation. From the simple logic of a Full Adder, we build complex circuits for high-speed addition (CLA), multiplication (Booth, CSA), and division (Non-Restoring). Floating-point representation (standardized by IEEE 754) allows computers to handle a vast range of real numbers, albeit with careful management of normalization, rounding, and potential exceptions like overflow and underflow. These intricate designs enable the powerful calculations we rely on every day.
