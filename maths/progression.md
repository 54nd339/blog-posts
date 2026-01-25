---
title: Sequences and Series - Unraveling Patterns and Infinite Sums
description: Explore the mathematical world of order and summation, from the steady steps of arithmetic progressions to the multiplying leaps of geometric series. This post delves into sequences, different types of progressions, the fascinating relationships between means, and the art of summing series, providing tools to understand patterns and calculate cumulative effects.
date: 2022-12-30
draft: false
slug: /pensieve/maths/sequences-series-progressions
tags:
  - Mathematics
  - Combinatorics
---

Mathematics is rich with structures that help us understand and predict patterns. Among the most fundamental of these are sequences and series. A sequence arranges elements in a specific order, while a series is concerned with the sum of these elements. When these sequences follow particular rules, they form progressions, each with unique characteristics and properties. This exploration covers various types of sequences and series, the methods for their summation, the concept of means, and important inequalities that arise from them.

---
## Laying the Groundwork: Sequences, Series, and Progressions – Order and Sum

* **Sequence:** A sequence is an ordered list of numbers, called terms, arranged according to a definite rule.
    * A **Real Sequence** is a sequence whose terms are real numbers.
    * A **Finite Sequence** has a limited number of terms.
    * An **Infinite Sequence** continues indefinitely.
    We often denote the $n^{th}$ term of a sequence as $a_n$, and the sequence itself as $\{a_n\}$ or $(a_n)$.

* **Series:** A series is formed by adding the terms of a sequence.
    * If $a_1, a_2, a_3, \ldots, a_n, \ldots$ is a sequence, then the expression $a_1 + a_2 + a_3 + \ldots + a_n + \ldots$ is a series.
    * The sum of the first $n$ terms is denoted by $S_n = \sum_{k=1}^n a_k$.
    * An **infinite series** is the sum of an infinite number of terms, denoted $\sum_{k=1}^\infty a_k$.

* **Progressions:** These are sequences where the terms are formed according to a specific, constant pattern. We will focus on arithmetic, geometric, and harmonic progressions.

---
## The Steady March: Arithmetic Progressions (A.P.) – Constant Steps

An **arithmetic progression (A.P.)** is a sequence of numbers such that the difference between consecutive terms is constant. This constant difference is called the **common difference ($d$)**.

* **Characteristics:** If $a_1, a_2, a_3, \ldots$ are in A.P., then $a_2-a_1 = a_3-a_2 = \ldots = d$.
* **$n^{th}$ Term ($a_n$):** If the first term is $a_1$ and the common difference is $d$, the $n^{th}$ term is given by:
    $$ a_n = a_1 + (n-1)d $$
* **Sum of first $n$ terms ($S_n$):**
    The sum of the first $n$ terms of an A.P. can be calculated using two common formulas:
    1.  $S_n = \dfrac{n}{2}[2a_1 + (n-1)d]$
    2.  $S_n = \dfrac{n}{2}[a_1 + a_n]$ (using the last term $a_n$)
* **Sum of an infinite A.P.:** An infinite arithmetic progression diverges (its sum tends to $\pm\infty$) unless both $a_1=0$ and $d=0$ (in which case the sum is 0). Therefore, we typically do not discuss the sum of an infinite A.P. in the context of convergence to a finite value.

---
## The Growing (or Shrinking) Leap: Geometric Progressions (G.P.) – Constant Ratios

A **geometric progression (G.P.)** is a sequence of numbers where each term after the first is found by multiplying the previous one by a fixed, non-zero number called the **common ratio ($r$)**.

* **Characteristics:** If $a_1, a_2, a_3, \ldots$ are in G.P., then $a_2/a_1 = a_3/a_2 = \ldots = r$.
* **$n^{th}$ Term ($a_n$):** If the first term is $a_1$ and the common ratio is $r$, the $n^{th}$ term is:
    $$ a_n = a_1 r^{n-1} $$
* **Sum of first $n$ terms ($S_n$):**
    If $r=1$, then $S_n = n a_1$.
    If $r \neq 1$, the sum is:
    $$ S_n = \dfrac{a_1(r^n - 1)}{r-1} \quad \text{or} \quad S_n = \dfrac{a_1(1 - r^n)}{1-r} $$
* **Sum of an infinite G.P. ($S_\infty$):**
    An infinite geometric series converges (has a finite sum) if and only if the absolute value of the common ratio is less than 1 (i.e., $|r|<1$).
    If $|r|<1$, the sum to infinity is:
    $$ S_\infty = \dfrac{a_1}{1-r} $$
    If $|r| \ge 1$ (and $a_1 \neq 0$), the series diverges.

---
## The Reciprocal Rhythm: Harmonic Progressions (H.P.) – Inverted Arithmetic

A sequence of numbers $a_1, a_2, a_3, \ldots$ (where no term is zero) is said to be in **harmonic progression (H.P.)** if their reciprocals $1/a_1, 1/a_2, 1/a_3, \ldots$ are in arithmetic progression.

* **Characteristics:** Based on the A.P. of reciprocals.
* **$n^{th}$ Term ($a_n$):** To find the $n^{th}$ term of an H.P., first find the $n^{th}$ term of the corresponding A.P. Let the first term of the H.P. be $a_1$ and the second term be $a_2$. Then the first two terms of the corresponding A.P. are $1/a_1$ and $1/a_2$. The common difference of this A.P. is $d' = 1/a_2 - 1/a_1$.
    The $n^{th}$ term of the A.P. is $1/a_n = 1/a_1 + (n-1)d'$.
    So, the $n^{th}$ term of the H.P. is:
    $$ a_n = \dfrac{1}{1/a_1 + (n-1)d'} $$
    where $d' = \dfrac{1}{a_2} - \dfrac{1}{a_1}$ (or more generally, $d'$ is the common difference of the A.P. formed by the reciprocals).
* **Sum of $n$ terms:** There is no simple general formula for the sum of $n$ terms of a harmonic progression.

---
## Filling the Gaps: Insertion of Means – Finding the In-Betweens

A "mean" between two numbers can be thought of as an intermediate value that maintains the pattern of a specific progression.

* **Arithmetic Mean (A.M.):**
    If $A$ is the single A.M. between $a$ and $b$, then $a, A, b$ are in A.P. This implies $A-a = b-A$, so $2A = a+b$, and $A = \dfrac{a+b}{2}$.
    To insert $k$ arithmetic means ($A_1, A_2, \ldots, A_k$) between two numbers $a$ and $b$:
    The sequence $a, A_1, A_2, \ldots, A_k, b$ will be an A.P. with $k+2$ terms.
    Let $d$ be the common difference. Then $b = a + ((k+2)-1)d = a + (k+1)d$.
    So, $d = \dfrac{b-a}{k+1}$.
    The $j^{th}$ inserted A.M. is $A_j = a + jd = a + j\dfrac{b-a}{k+1}$.

* **Geometric Mean (G.M.):**
    If $G$ is the single G.M. between two positive numbers $a$ and $b$, then $a, G, b$ are in G.P. This implies $G/a = b/G$, so $G^2 = ab$, and $G = \sqrt{ab}$.
    To insert $k$ geometric means ($G_1, G_2, \ldots, G_k$) between $a$ and $b$ (assuming $a,b > 0$):
    The sequence $a, G_1, G_2, \ldots, G_k, b$ will be a G.P. with $k+2$ terms.
    Let $r$ be the common ratio. Then $b = a r^{(k+2)-1} = a r^{k+1}$.
    So, $r = \left(\dfrac{b}{a}\right)^{1/(k+1)}$.
    The $j^{th}$ inserted G.M. is $G_j = a r^j = a \left(\dfrac{b}{a}\right)^{j/(k+1)}$.

* **Harmonic Mean (H.M.):**
    If $H$ is the single H.M. between $a$ and $b$, then $a, H, b$ are in H.P. This means $1/a, 1/H, 1/b$ are in A.P.
    So, $\dfrac{1}{H} = \dfrac{1/a + 1/b}{2} = \dfrac{(a+b)/(ab)}{2} = \dfrac{a+b}{2ab}$.
    Thus, $H = \dfrac{2ab}{a+b}$.
    To insert $k$ harmonic means ($H_1, H_2, \ldots, H_k$) between $a$ and $b$:
    The reciprocals $1/a, 1/H_1, \ldots, 1/H_k, 1/b$ form an A.P.
    Let $d'$ be the common difference of this A.P. $d' = \dfrac{(1/b) - (1/a)}{k+1} = \dfrac{a-b}{ab(k+1)}$.
    Then $\dfrac{1}{H_j} = \dfrac{1}{a} + j d' = \dfrac{1}{a} + j\dfrac{a-b}{ab(k+1)}$. $H_j$ is the reciprocal of this value.

---
## The Mean Trio: Relation between A.M., G.M., and H.M.

For any two positive numbers $a$ and $b$:
* Arithmetic Mean: $A = \dfrac{a+b}{2}$
* Geometric Mean: $G = \sqrt{ab}$
* Harmonic Mean: $H = \dfrac{2ab}{a+b}$

**Relationship 1: $G^2 = AH$**
Let's verify: $AH = \left(\dfrac{a+b}{2}\right) \left(\dfrac{2ab}{a+b}\right) = \dfrac{2ab(a+b)}{2(a+b)} = ab$.
And $G^2 = (\sqrt{ab})^2 = ab$.
Thus, $G^2 = AH$. This means that for two positive numbers, their G.M. is the geometric mean of their A.M. and H.M. (i.e., A.M., G.M., H.M. are in G.P.).

**Relationship 2 (Inequality): $A \ge G \ge H$**
For $a,b > 0$:
* $A \ge G$: $\dfrac{a+b}{2} \ge \sqrt{ab} \iff a+b \ge 2\sqrt{ab} \iff (\sqrt{a}-\sqrt{b})^2 \ge 0$. This is always true. Equality holds if $a=b$.
* $G \ge H$: $\sqrt{ab} \ge \dfrac{2ab}{a+b} \iff 1 \ge \dfrac{2\sqrt{ab}}{a+b} \iff a+b \ge 2\sqrt{ab}$. This is also true from the A.M.-G.M. inequality applied to $a$ and $b$. Equality holds if $a=b$.
So, for two positive numbers, $A.M. \ge G.M. \ge H.M.$, with equality holding if and only if the numbers are equal.

---
## A Special Blend: Arithmetico-Geometric Progression (A.G.P.)

An **arithmetico-geometric progression (A.G.P.)** is a sequence where each term is the product of the corresponding terms of an arithmetic progression and a geometric progression.
The general form is:
$a_1 b_1, a_2 b_2, a_3 b_3, \ldots$
If the A.P. is $a, a+d, a+2d, \ldots$ and the G.P. is $1, r, r^2, \ldots$ (or $b, br, br^2, \ldots$), then a common form of A.G.P. is:
$a, (a+d)r, (a+2d)r^2, \ldots, [a+(n-1)d]r^{n-1}, \ldots$

**Sum of $n$ terms ($S_n$) of an A.G.P.:**
Let $S_n = a + (a+d)r + (a+2d)r^2 + \ldots + [a+(n-1)d]r^{n-1}$.
Multiply by $r$: $rS_n = \quad ar + (a+d)r^2 + \ldots + [a+(n-2)d]r^{n-1} + [a+(n-1)d]r^n$.
Subtracting: $S_n(1-r) = a + [dr + dr^2 + \ldots + dr^{n-1}] - [a+(n-1)d]r^n$.
The terms in the square bracket form a G.P. with $n-1$ terms, first term $dr$, and common ratio $r$. Its sum is $\dfrac{dr(1-r^{n-1})}{1-r}$.
So, $S_n(1-r) = a + \dfrac{dr(1-r^{n-1})}{1-r} - [a+(n-1)d]r^n$.
For $r \neq 1$:
$$ S_n = \dfrac{a}{1-r} + \dfrac{dr(1-r^{n-1})}{(1-r)^2} - \dfrac{[a+(n-1)d]r^n}{1-r} $$

**Sum of an infinite A.G.P. ($S_\infty$):**
If $|r|<1$, then as $n \to \infty$, $r^{n-1} \to 0$ and $r^n \to 0$. Also, the term $[a+(n-1)d]r^n \to 0$.
$$ S_\infty = \dfrac{a}{1-r} + \dfrac{dr}{(1-r)^2} $$

---
## Summing It Up: Sum of Some Special Series

Beyond basic progressions, there are well-known formulas for the sums of powers of the first $n$ natural numbers:

* **Sum of first $n$ natural numbers:**
    $$ \sum_{k=1}^n k = 1+2+3+\dots+n = \dfrac{n(n+1)}{2} $$
* **Sum of squares of first $n$ natural numbers:**
    $$ \sum_{k=1}^n k^2 = 1^2+2^2+3^2+\dots+n^2 = \dfrac{n(n+1)(2n+1)}{6} $$
* **Sum of cubes of first $n$ natural numbers:**
    $$ \sum_{k=1}^n k^3 = 1^3+2^3+3^3+\dots+n^3 = \left[\dfrac{n(n+1)}{2}\right]^2 = \left(\sum_{k=1}^n k\right)^2 $$

These formulas are often proven by mathematical induction or other combinatorial arguments.

---
## Nested Sums: Sum of Series When $i$ and $j$ are Dependent

Evaluating double summations where the limits of the inner sum depend on the outer sum requires careful step-by-step summation.

**Example:** Evaluate $S = \sum_{i=1}^n \sum_{j=1}^i j$.
First, evaluate the inner sum for a fixed $i$:
$$ \sum_{j=1}^i j = \dfrac{i(i+1)}{2} $$
Now, substitute this back into the outer sum:
$$ S = \sum_{i=1}^n \dfrac{i(i+1)}{2} = \dfrac{1}{2} \sum_{i=1}^n (i^2+i) $$
$$ S = \dfrac{1}{2} \left( \sum_{i=1}^n i^2 + \sum_{i=1}^n i \right) $$
Using the formulas for sum of squares and sum of first $n$ natural numbers:
$$ S = \dfrac{1}{2} \left( \dfrac{n(n+1)(2n+1)}{6} + \dfrac{n(n+1)}{2} \right) $$
$$ S = \dfrac{1}{2} \left( \dfrac{n(n+1)}{6} [(2n+1) + 3] \right) = \dfrac{1}{2} \left( \dfrac{n(n+1)}{6} [2n+4] \right) $$
$$ S = \dfrac{1}{2} \left( \dfrac{n(n+1)2(n+2)}{6} \right) = \dfrac{n(n+1)(n+2)}{6} $$
This method involves treating the inner summation as a function of the outer summation index, summing it, and then summing the resulting expression. Changing the order of summation is another technique but can be more complex if the dependency is intricate.

---
## Mean Comparisons: Inequalities Involving Means

The relationships between Arithmetic Mean (A.M.), Geometric Mean (G.M.), and Harmonic Mean (H.M.) are not just equalities but also extend to important inequalities.

* **A.M. $\ge$ G.M. $\ge$ H.M. for $n$ positive numbers:**
    For $n$ positive numbers $x_1, x_2, \ldots, x_n$:
    * Arithmetic Mean: $A_n = \dfrac{x_1 + x_2 + \dots + x_n}{n}$
    * Geometric Mean: $G_n = (x_1 x_2 \dots x_n)^{1/n}$
    * Harmonic Mean: $H_n = \dfrac{n}{\dfrac{1}{x_1} + \dfrac{1}{x_2} + \dots + \dfrac{1}{x_n}}$
    The fundamental inequality states:
    $$ A_n \ge G_n \ge H_n $$
    Equality holds if and only if $x_1 = x_2 = \dots = x_n$.

* **Arithmetic Mean of $m^{th}$ Power (Power Means):**
    For a set of positive numbers $x_1, \ldots, x_n$, the power mean (or generalized mean) with exponent $m$ (where $m \neq 0$) is defined as:
    $$ M_m = \left(\dfrac{x_1^m + x_2^m + \dots + x_n^m}{n}\right)^{1/m} $$
    There is a known inequality for power means: If $m > p$, then $M_m \ge M_p$.
    * $M_1$ is the Arithmetic Mean.
    * As $m \to 0$, $M_m$ approaches the Geometric Mean. ($M_0 = G_n$).
    * $M_{-1}$ is the Harmonic Mean.
    Thus, for $m=1, p \to 0, q=-1$, we have $M_1 \ge M_0 \ge M_{-1}$, which is $A_n \ge G_n \ge H_n$.

* **Weighted Means:**
    If we have weights $w_1, w_2, \ldots, w_n$ (all positive) associated with positive numbers $x_1, x_2, \ldots, x_n$:
    * Weighted A.M.: $A_w = \dfrac{w_1x_1 + w_2x_2 + \dots + w_nx_n}{w_1 + w_2 + \dots + w_n}$
    * Weighted G.M.: $G_w = (x_1^{w_1} x_2^{w_2} \dots x_n^{w_n})^{1/(w_1 + w_2 + \dots + w_n)}$
    * Weighted H.M.: $H_w = \dfrac{w_1 + w_2 + \dots + w_n}{\dfrac{w_1}{x_1} + \dfrac{w_2}{x_2} + \dots + \dfrac{w_n}{x_n}}$
    The inequality $A_w \ge G_w \ge H_w$ also holds for positive $x_i$ and positive weights $w_i$. Equality holds if all $x_i$ are equal.

---
## Key Takeaways: The Power of Order and Summation

Sequences and series are fundamental mathematical tools that allow us to model patterns, predict future values, and understand cumulative effects.

* **Ordered Structure:** Sequences provide a framework for listing elements in a defined order, while progressions (Arithmetic, Geometric, Harmonic) introduce specific rules for term generation.
* **Summation Techniques:** Series represent the sum of sequence terms. Formulas for finite and (where convergent) infinite sums of A.P., G.P., and A.G.P. are essential. Special series sums (like powers of natural numbers) are also important.
* **Means as Representatives:** Arithmetic, Geometric, and Harmonic means offer different ways to find a "central" value for a set of numbers, with important relationships ($G^2=AH$) and inequalities ($A \ge G \ge H$) governing them.
* **Pattern Recognition:** The study of sequences and series sharpens our ability to recognize underlying mathematical patterns and relationships.
* **Broad Applications:** These concepts are foundational not only in pure mathematics but also in physics (e.g., motion, wave superposition), finance (compound interest, annuities), computer science (algorithm analysis), and many other disciplines.

By understanding the principles of sequences, series, and progressions, we gain powerful analytical tools for interpreting and interacting with a patterned world.