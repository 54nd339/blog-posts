---
title: Permutations and Combinations - The Art and Science of Counting
description: Unlock the principles of systematic counting with permutations and combinations. This exploration covers fundamental rules, factorial notation, arrangements in lines and circles, selections, distributions, the Multinomial Theorem, and the Principle of Inclusion-Exclusion – essential tools for probability, discrete mathematics, and beyond.
date: 2022-12-31
draft: false
slug: /pensieve/maths/permutations-combinations
tags:
  - Mathematics
  - Combinatorics
---

The ability to count efficiently and accurately is fundamental not only in mathematics but also in numerous fields such as computer science, probability, statistics, and everyday problem-solving. Permutations and combinations are the core concepts of combinatorics that provide systematic methods for determining the number of ways objects can be arranged or selected. This exploration delves into these powerful tools, starting from the fundamental principles of counting and extending to more advanced theorems and applications.

---
## The Ground Rules of Counting: Fundamental Principles – Multiply or Add

At the heart of combinatorics lie two basic principles that form the foundation for more complex counting techniques.

* **Multiplication Rule (Fundamental Principle of Counting):**
    If a task can be performed in $m$ ways, and following this, a second task can be performed in $n$ ways, then the total number of ways to perform both tasks in succession is $m \times n$. This principle extends to any number of sequential tasks. If there are $k$ tasks, and the first can be done in $n_1$ ways, the second in $n_2$ ways after the first, ..., and the $k^{th}$ in $n_k$ ways after the $(k-1)^{th}$, then the total number of ways to perform all $k$ tasks is $n_1 \times n_2 \times \dots \times n_k$.

* **Addition Rule:**
    If a task can be performed in $m$ ways and a second, *mutually exclusive* task can be performed in $n$ ways, then there are $m+n$ ways to perform either the first task *or* the second task. This rule applies when choices are alternative and cannot occur simultaneously.

---
## Factorial Power: Notation and Prime Insights

The factorial of a non-negative integer $n$, denoted by $n!$, represents the product of all positive integers less than or equal to $n$.
$$ n! = n \times (n-1) \times (n-2) \times \dots \times 3 \times 2 \times 1 $$
By definition, $0! = 1$. Factorials are fundamental in calculating permutations and combinations.

### Exponent of a Prime $p$ in $n!$ (Legendre's Formula)
To find the highest power of a prime number $p$ that divides $n!$ (denoted $E_p(n!)$), we use Legendre's Formula:
$$ E_p(n!) = \sum_{k=1}^{\infty} \left\lfloor \dfrac{n}{p^k} \right\rfloor = \left\lfloor \dfrac{n}{p} \right\rfloor + \left\lfloor \dfrac{n}{p^2} \right\rfloor + \left\lfloor \dfrac{n}{p^3} \right\rfloor + \dots $$
where $\lfloor x \rfloor$ is the floor function, giving the greatest integer less than or equal to $x$. The sum is finite because terms become zero when $p^k > n$.

---
## Order Matters: Permutations – Arranging Items

A **permutation** is an arrangement of a set of objects in a specific order.

* **Permutations of $n$ distinct objects taken $r$ at a time ($P(n,r)$ or $^nP_r$ or $P^n_r$):**
    This is the number of ways to arrange $r$ objects chosen from $n$ distinct objects, where $0 \le r \le n$.
    $$ P(n,r) = \dfrac{n!}{(n-r)!} = n(n-1)(n-2)\dots(n-r+1) $$

* **Permutations of $n$ distinct objects taken all at a time ($P(n,n)$):**
    This is simply $n!$.

* **Permutations when some objects are identical:**
    If there are $n$ objects in total, where $p_1$ objects are alike of one kind, $p_2$ objects are alike of another kind, ..., $p_k$ objects are alike of a $k^{th}$ kind, such that $p_1+p_2+\dots+p_k = n$, then the number of distinct permutations is:
    $$ \dfrac{n!}{p_1! p_2! \dots p_k!} $$

* **Permutations with repetitions allowed:**
    The number of permutations of $n$ distinct objects taken $r$ at a time, when repetition of objects is allowed, is $n^r$. Each of the $r$ positions can be filled in $n$ ways.

---
## Order Doesn't Matter: Combinations – Selecting Items

A **combination** is a selection of objects from a set where the order of selection does not matter.

* **Combinations of $n$ distinct objects taken $r$ at a time ($C(n,r)$ or $^nC_r$ or $\binom{n}{r}$):**
    This is the number of ways to choose $r$ objects from $n$ distinct objects, where $0 \le r \le n$.
    Since each combination of $r$ objects can be permuted in $r!$ ways, $P(n,r) = C(n,r) \times r!$.
    $$ C(n,r) = \binom{n}{r} = \dfrac{P(n,r)}{r!} = \dfrac{n!}{r!(n-r)!} $$

* **Properties of $C(n,r)$:**
    * $C(n,0) = 1$ (one way to choose no objects)
    * $C(n,n) = 1$ (one way to choose all objects)
    * $C(n,r) = C(n, n-r)$ (choosing $r$ objects is the same as choosing $n-r$ objects to leave behind)
    * $C(n,r) + C(n, r-1) = C(n+1, r)$ (Pascal's Identity)

* **Combinations with repetitions allowed (Multiset Coefficient):**
    The number of ways to choose $r$ items from $n$ distinct types of items, where repetition is allowed (also known as combinations with replacement), is given by the multiset coefficient:
    $$ \binom{n+r-1}{r} = \binom{n+r-1}{n-1} = \dfrac{(n+r-1)!}{r!(n-1)!} $$
    This can be visualized using the "stars and bars" method.

---
## Going in Circles: Circular Permutations – Arrangements Around a Table

A **circular permutation** is an arrangement of objects in a circle. Unlike linear permutations, rotations of the same arrangement are considered identical.

* **Number of circular permutations of $n$ distinct objects:**
    If we fix one object's position, the remaining $n-1$ objects can be arranged in $(n-1)!$ ways relative to it.
    So, the number of distinct circular permutations is $(n-1)!$.

* **Circular permutations when clockwise and anti-clockwise arrangements are indistinguishable:**
    If arrangements that are mirror images of each other (e.g., for necklaces or bracelets where flipping over doesn't change the arrangement) are considered the same, then the number of distinct circular permutations is $\dfrac{(n-1)!}{2}$, provided $n > 2$. For $n=1$, it's 0! = 1 (or (1-1)!/2 not well-defined). For $n=2$, it's (2-1)! = 1. The division by 2 applies generally for $n \ge 3$.

---
## Choosing Flexibly: All Possible Selections – Taking Some or All

This refers to the number of ways one can select items from a given set, allowing for different quantities of items to be selected.

* **Selecting one or more items from $n$ distinct items:**
    For each of the $n$ items, there are two choices: either select it or not select it. This gives $2^n$ total possibilities. Since we want to select *at least one* item, we exclude the case where no item is selected.
    Number of ways = $2^n - 1$.
    This can also be seen as $C(n,1) + C(n,2) + \dots + C(n,n)$.

* **Selecting one or more items when items are not all distinct:**
    Suppose there are $p$ items of one kind, $q$ items of another kind, $r$ items of a third kind, and so on.
    The number of ways to select some or all items from this collection is given by:
    $$ (p+1)(q+1)(r+1)\dots - 1 $$
    The '$-1$' excludes the case where no item is selected from any kind. For each kind, one can select $0, 1, \dots, p$ items (i.e., $p+1$ ways).

---
## Sharing is Caring: Divisions & Distributions – Grouping and Handing Out

This involves partitioning a set of items into groups or distributing them.

* **Division of $n$ distinct items into $k$ distinct groups of specified sizes $n_1, n_2, \ldots, n_k$ (where $\sum_{i=1}^k n_i = n$):**
    The number of ways is:
    $$ \dfrac{n!}{n_1! n_2! \dots n_k!} $$
    This is also the coefficient in the Multinomial Theorem.

* **Distribution of $n$ distinct items among $k$ distinct persons/boxes such that they receive $n_1, n_2, \ldots, n_k$ items respectively:**
    This is the same as the division into distinct groups with specified sizes: $\dfrac{n!}{n_1! n_2! \dots n_k!}$.

* **Division of $n$ distinct items into $k$ groups of equal size $m$ (so $n=km$):**
    * If the $k$ groups are **distinct** (e.g., group A, group B, etc.):
        The number of ways is $\dfrac{n!}{(m!)^k}$.
    * If the $k$ groups are **identical** (indistinguishable):
        The number of ways is $\dfrac{n!}{k!(m!)^k}$. We divide by $k!$ because the order of the identical groups does not matter.

* **Distribution of $n$ identical items into $r$ distinct boxes (Stars and Bars):**
    This is equivalent to finding the number of non-negative integer solutions to $x_1 + x_2 + \dots + x_r = n$.
    * If boxes can be empty (i.e., $x_i \ge 0$):
        The number of ways is $\binom{n+r-1}{r-1}$ or $\binom{n+r-1}{n}$.
    * If each box must contain at least one item (i.e., $x_i \ge 1$):
        First, place one item in each of the $r$ boxes. Then distribute the remaining $n-r$ identical items into $r$ distinct boxes without restriction.
        The number of ways is $\binom{(n-r)+r-1}{r-1} = \binom{n-1}{r-1}$.

---
## Expanding with Flair: Multinomial Theorem – Generalizing Binomials

The **Multinomial Theorem** provides a formula for the expansion of a sum of multiple terms raised to a power, generalizing the Binomial Theorem.
For any positive integer $n$ and any non-negative integers $n_1, n_2, \ldots, n_k$ such that $n_1 + n_2 + \dots + n_k = n$, the expansion of $(x_1 + x_2 + \dots + x_k)^n$ is given by:
$$ (x_1 + x_2 + \dots + x_k)^n = \sum_{n_1+n_2+\dots+n_k=n} \dfrac{n!}{n_1! n_2! \dots n_k!} x_1^{n_1} x_2^{n_2} \dots x_k^{n_k} $$
The sum is taken over all possible combinations of non-negative integer indices $n_1$ through $n_k$ such that their sum is $n$.
The coefficient $\dfrac{n!}{n_1! n_2! \dots n_k!}$ is called the **multinomial coefficient**, sometimes written as $\binom{n}{n_1, n_2, \dots, n_k}$.

* The sum of all multinomial coefficients in the expansion (obtained by setting $x_1=x_2=\dots=x_k=1$) is $k^n$.

---
## The Pigeonhole Principle: Guaranteeing a Crowd

The Pigeonhole Principle is a simple yet surprisingly powerful concept in combinatorics that guarantees the existence of a certain condition without explicitly finding it.

* **Basic Principle:** If you have more pigeons than pigeonholes, then at least one pigeonhole must contain more than one pigeon.
    Formally: If $n+1$ items are put into $n$ containers, then at least one container must contain more than one item.
    * **Proof by Contradiction:** Assume no container has more than one item. Then the maximum number of items you could have is $n \times 1 = n$. This contradicts the fact that we have $n+1$ items. Therefore, the assumption must be false, and at least one container must have more than one item.
    * **Example:** In any group of 3 people, at least two must be of the same gender (pigeons=3, pigeonholes=2). In any group of 13 people, at least two must have their birthday in the same month (pigeons=13, pigeonholes=12).

* **Generalized Pigeonhole Principle:**
    If $N$ objects are placed into $k$ boxes, then there is at least one box containing at least $\lceil N/k \rceil$ objects, where $\lceil x \rceil$ is the **ceiling function**, which rounds $x$ up to the nearest integer.
    * **Example:** In a group of 100 people, what is the minimum number of people who were born in the same month? Here, $N=100$ people (objects) and $k=12$ months (boxes). The minimum number guaranteed in at least one month is $\lceil 100/12 \rceil = \lceil 8.33 \rceil = 9$.

---
## Counting Without Overcounting: Principle of Inclusion-Exclusion – The Venn Diagram Master

The **Principle of Inclusion-Exclusion (PIE)** is a counting technique that computes the number of elements in the union of multiple sets. It corrects for overcounting elements that belong to more than one set.

* **For two sets $A$ and $B$:**
    $$ |A \cup B| = |A| + |B| - |A \cap B| $$
* **For three sets $A$, $B$, and $C$:**
    $$ |A \cup B \cup C| = |A| + |B| + |C| - (|A \cap B| + |A \cap C| + |B \cap C|) + |A \cap B \cap C| $$
* **General form for $n$ sets $A_1, A_2, \ldots, A_n$:**
    $$ \left| \bigcup_{i=1}^n A_i \right| = \sum_{i} |A_i| - \sum_{i<j} |A_i \cap A_j| + \sum_{i<j<k} |A_i \cap A_j \cap A_k| - \dots + (-1)^{n-1} |A_1 \cap A_2 \cap \dots \cap A_n| $$
    In a more compact notation:
    $$ \left| \bigcup_{i=1}^n A_i \right| = \sum_{\emptyset \neq I \subseteq \{1,\dots,n\}} (-1)^{|I|-1} \left| \bigcap_{i \in I} A_i \right| $$
PIE is often used to count the number of elements that satisfy *at least one* of several properties. To count elements satisfying *none* of the properties, one can use the complement: $N(\text{none}) = |U| - |\bigcup A_i|$.

---
## Key Takeaways: The Essence of Order and Selection

Permutations and combinations are the foundational tools of combinatorics, providing systematic ways to count arrangements and selections.

* **Fundamental Principles:** The Multiplication Rule (for sequential tasks) and Addition Rule (for mutually exclusive choices) are the building blocks for all counting methods.
* **Factorials ($n!$):** Essential for calculating the number of arrangements of distinct objects. Legendre's formula helps determine prime factorizations.
* **Permutations ($P(n,r)$):** Used when the order of arrangement matters. Formulas adapt for distinct objects, identical objects, and when repetition is allowed. Circular permutations handle arrangements in a loop.
* **Combinations ($C(n,r)$):** Used when the order of selection does not matter. Formulas also adapt for distinct objects and when repetition is allowed (multisets).
* **Pigeonhole Principle:** A powerful non-constructive tool that guarantees that if there are too many items to fit into too few categories, at least one category must contain multiple items.
* **Divisions and Distributions:** Provide methods for partitioning sets into groups or distributing items into distinct or identical containers, with various constraints.
* **Multinomial Theorem:** Generalizes the binomial expansion for powers of sums with multiple terms, with coefficients determined by combinatorial arguments.
* **Principle of Inclusion-Exclusion:** A powerful technique for counting elements in the union of several sets by systematically adding and subtracting the sizes of their intersections to correct for overcounting.

These principles are not merely abstract mathematical exercises; they are critical for calculating probabilities, analyzing algorithms, designing experiments, and solving a wide array of problems where the number of possible outcomes or configurations needs to be determined.