---
title: Karatsuba's Algorithm - Multiplying Large Numbers Faster with Divide and Conquer
description: Unlock the magic of Karatsuba's algorithm for fast multiplication of large numbers! We explore how this divide and conquer technique reduces n-digit multiplications to just three n/2-digit multiplications, beating the traditional O(n^2) method.
date: 2023-02-09
draft: false
slug: /pensieve/algorithms/karatsuba-multiplication
tags:
  - DSA
  - Divide and Conquer
---

Hey everyone, and welcome back to the blog! We all learned how to multiply numbers in school, typically using the standard long multiplication method. It works perfectly well for numbers we encounter daily. But what happens when you need to multiply *really* large numbers – numbers with thousands, or even millions, of digits? This is a common scenario in areas like cryptography, scientific computing, and computer algebra systems. The traditional O(n²) algorithm becomes too slow.

Enter **Karatsuba's Algorithm**, a brilliant example of how the "Divide and Conquer" strategy can lead to significantly faster multiplication. Developed by Anatoly Karatsuba in 1960 (published in 1962), it was a groundbreaking discovery that showed we could multiply n-digit numbers more efficiently than the standard quadratic approach. Let's see how it achieves this speed-up!

## The Problem: Multiplying Giants  x
Given two n-digit numbers, say $x$ and $y$, our goal is to compute their product $x \cdot y$.
The standard "schoolbook" multiplication method involves multiplying every digit of $x$ by every digit of $y$, which results in approximately $n^2$ single-digit multiplications, giving it an O(n²) time complexity. Karatsuba's algorithm cleverly reduces the number of recursive multiplications needed.

The core idea is to reduce the multiplication of two n-digit numbers to at most **three** multiplications of n/2-digit numbers, along with some additions and shifts (which are computationally cheaper than multiplications for large numbers).

## The Karatsuba Algorithm: A Divide and Conquer Masterpiece

Let's break down how Karatsuba's algorithm works. For simplicity, assume we are working with numbers in base 10, and $n$ is the number of digits (and let's assume $n$ is a power of 2 for easier splitting, though the algorithm can be adapted).

1.  **Splitting the Numbers:**
    If our numbers $x$ and $y$ have $n$ digits, we can split each of them into two halves, each with roughly $n/2$ digits. Let $m = n/2$.
    * We can write $x = a \cdot 10^m + b$
    * And $y = c \cdot 10^m + d$

    Here, $a$ and $c$ are the "high" parts (most significant digits) of $x$ and $y$ respectively, and $b$ and $d$ are the "low" parts (least significant digits). Each of $a, b, c, d$ is an $m$-digit number.

2.  **The Standard Product (for comparison):**
    The product $x \cdot y$ can then be expressed as:
    $x \cdot y = (a \cdot 10^m + b) \cdot (c \cdot 10^m + d)$
    $x \cdot y = (ac) \cdot 10^{2m} + (ad + bc) \cdot 10^m + (bd)$

    Notice that this standard expansion requires *four* multiplications of $m$-digit numbers: $ac$, $ad$, $bc$, and $bd$. This is what leads to the O(n²) complexity if done recursively without further optimization.

3.  **Karatsuba's Insight (The Clever Trick):**
    Karatsuba realized that the middle term $(ad + bc)$ could be computed with just *one* additional multiplication, by cleverly rearranging terms. We need $ac$ and $bd$ anyway. Consider the product $(a+b)(c+d)$:
    $(a+b)(c+d) = ac + ad + bc + bd$
    From this, we can see that the middle term we need is:
    $ad + bc = (a+b)(c+d) - ac - bd$

    So, we only need to compute three $m$-digit (or slightly larger if $a+b$ or $c+d$ have $m+1$ digits) multiplications:
    * $P_1 = ac$
    * $P_2 = bd$
    * $P_3 = (a+b)(c+d)$

    Then, the middle term $ad+bc$ is $P_3 - P_1 - P_2$.
    The final product $x \cdot y$ becomes:
    $x \cdot y = P_1 \cdot 10^{2m} + (P_3 - P_1 - P_2) \cdot 10^m + P_2$

    This requires only **three** recursive multiplications of numbers roughly half the size, plus some additions, subtractions, and shifts (multiplications by powers of 10), which are much faster operations than full multiplications for large numbers.

### Pseudocode for Karatsuba's Algorithm

Here's a conceptual pseudocode that captures the algorithm:

```pseudocode
// x, y: The two large numbers to multiply (can be represented as strings or lists of digits)
function Karatsuba(x, y):
    // Base Case: If numbers are small enough, use standard multiplication
    // (e.g., if they fit within a single machine word or are less than 10 digits)
    if length(x) < SOME_BASE_CASE_LENGTH or length(y) < SOME_BASE_CASE_LENGTH:
        // For simplicity, user's pseudocode used < 10
        // This check actually depends on the number representation (string vs. integer)
        // Assuming x and y are integers here for this base case:
        if x < 10 or y < 10: // From user's pseudocode
            return x * y

    // Determine n (number of digits, roughly) and m (half the digits)
    n_x = number_of_digits(x)
    n_y = number_of_digits(y)
    n = max(n_x, n_y)
   
    // Make n an even number for simpler splitting, pad if necessary (implicitly handled by division/modulo)
    // Or ensure n is at least 2 for meaningful split. If n=1, it hits base case.
    if n < 2: // Handles single digit numbers not caught by x<10, y<10 if they are > 0
        return x * y

    m = ceiling(n / 2) // Or simply n / 2, integer division if n is even

    // Split numbers: x = a * 10^m + b,  y = c * 10^m + d
    power_of_10_m = 10^m
   
    a = x / power_of_10_m  // High part of x
    b = x % power_of_10_m  // Low part of x
    c = y / power_of_10_m  // High part of y
    d = y % power_of_10_m  // Low part of y

    // Recursive calls for the three products
    P1_ac = Karatsuba(a, c)
    P2_bd = Karatsuba(b, d)
    P3_sum_prods = Karatsuba(a + b, c + d)

    // Calculate the middle term
    P_ad_plus_bc = P3_sum_prods - P1_ac - P2_bd

    // Combine the results using shifts (multiplication by powers of 10) and additions
    // result = P1_ac * (10^(2*m)) + P_ad_plus_bc * (10^m) + P2_bd
    // Note: 10^(2*m) is (10^m) * (10^m)
    power_of_10_2m = power_of_10_m * power_of_10_m // Or 10^(2*m)
   
    result = (P1_ac * power_of_10_2m) + (P_ad_plus_bc * power_of_10_m) + P2_bd
   
    return result
```

## Explanation of the Steps:

1. **Base Case:** The recursion stops when the numbers become small enough (e.g., single digit, or fit within a native multiplication word of the machine). At this point, standard multiplication is used.
2. **Determine Split Point:** Calculate n, the maximum number of digits in $x$ or $y$. Then, determine m, which is roughly n/2. This m defines how many digits belong to the lower part of the numbers.
3. **Split Numbers:** Divide $x$ into $a$ (high part) and $b$ (low part), and $y$ into $c$ (high part) and $d$ (low part). For example, if $x = 12345$ and $m=2$, then $10^m = 100$, $a = 123$, $b = 45$.
4. **Recursive Multiplications:** Recursively compute the three necessary products:
    * $P_1 = ac = \text{Karatsuba}(a, c)$
    * $P_2 = bd = \text{Karatsuba}(b, d)$
    * $P_3 = (a+b)(c+d) = \text{Karatsuba}(a+b, c+d)$
5. **Calculate Middle Term:** Compute $ad+bc = P_3 - P_1 - P_2$.
6. **Combine Results:** Combine these three products using the formula: $x \cdot y = P_1 \cdot 10^{2m} + (P_3 - P_1 - P_2) \cdot 10^m + P_2$. The multiplications by $10^m$ and $10^{2m}$ are just left-shifts of the digits, which are computationally inexpensive operations.

## Correctness and Why it Works
The correctness of Karatsuba's algorithm stems from the algebraic identity:
$$ad + bc = (a+b)(c+d) - ac - bd$$

By calculating $ac$, $bd$, and $(a+b)(c+d)$ recursively, and then performing a few additions and subtractions, we effectively get all the components needed for the final product $ac \cdot 10^{2m} + (ad+bc) \cdot 10^m + bd$, but with only three recursive multiplications of half-sized numbers instead of four. The divide and conquer structure ensures that this reduction in multiplications at each step leads to an overall faster algorithm.

## Complexity Analysis

* **Time Complexity:**
    The recurrence relation for Karatsuba's algorithm is:
    $$T(n) = 3T(n/2) + O(n)$$
    where:
    - $T(n)$ is the time to multiply two $n$-digit numbers.
    - $3T(n/2)$ comes from the three recursive calls on $n/2$-digit numbers.
    - $O(n)$ represents the time for additions, subtractions, and shifts (which take linear time with respect to the number of digits).

    Using the Master Theorem, this recurrence relation solves to O($n^{log₂3}$), which is approximately **O($n^{1.585}$)**. This is a significant improvement over the O(n²) time complexity of the traditional long multiplication algorithm.

* **Space Complexity:** The space complexity is primarily determined by the depth of the recursion stack. Since the problem size is halved at each recursive step, the maximum depth of the stack will be **O(log n)**.

## Applications of Karatsuba's Algorithm

While modern CPUs have highly optimized hardware multiplication for standard integer sizes, Karatsuba's algorithm (and other more advanced fast multiplication algorithms like Toom-Cook and Schönhage–Strassen) becomes beneficial when dealing with arbitrarily large integers (bignums) that exceed hardware limits.

* **Cryptography:** Many cryptographic algorithms (e.g., RSA) involve operations on very large numbers.
* **Computer Algebra Systems:** Software like Mathematica or Maple that performs symbolic mathematics often needs to multiply large polynomials or numbers.
* **Scientific Computing:** Certain simulations or calculations might involve high-precision arithmetic with very large numbers.
* **Algorithm Design:** It's a classic example of the power of divide and conquer and is often taught in algorithm courses.

## Key Takeaways

* **Karatsuba's Algorithm** is a fast multiplication algorithm for large numbers, significantly outperforming the traditional O(n²) method.
* It uses a **divide and conquer** strategy, reducing the multiplication of two n-digit numbers to three multiplications of n/2-digit numbers, plus some cheaper additions and shifts.
* The time complexity is approximately **O($n^{1.585}$)**.
* It's particularly useful for multiplying numbers that are too large to fit into standard hardware registers (bignum arithmetic).

Karatsuba's algorithm is a beautiful illustration of how a clever change in approach can lead to substantial improvements in computational efficiency for a fundamental operation like multiplication!