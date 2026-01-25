---
title: Numbers, Equations, and Comparisons - The Language of Mathematics
description: From simple counting to solving complex cosmic puzzles, numbers and equations form the bedrock of our understanding. This post explores number systems, the logic of inequalities, and the art of solving polynomial equations. Delve into the fundamental language that describes patterns, quantities, and relationships in the universe!
date: 2022-12-25
draft: false
slug: /pensieve/maths/numbers-equations-inequalities
tags:
  - Mathematics
  - Number Theory
---

Mathematics is often described as the language of the universe, providing a framework to understand patterns, quantities, and relationships. From the simplest act of counting to the intricate equations describing the cosmos, its principles are fundamental. This exploration delves into foundational mathematical concepts: the diverse systems of numbers, the precise logic of inequalities, and the elegant structure of the theory of equations, particularly focusing on polynomials.

---
## The Building Blocks: Number Systems – A Universe of Digits

Our mathematical journey begins with understanding the different categories of numbers, each expanding our ability to quantify and conceptualize.

* **Natural Numbers ($\mathbb{N}$):** These are the counting numbers, the very first numbers we learn: $1, 2, 3, 4, \ldots$. They are used for counting and ordering.

* **Whole Numbers ($\mathbb{W}$):** This set includes all natural numbers along with zero: $0, 1, 2, 3, \ldots$. The inclusion of zero was a significant conceptual leap in mathematics.

* **Integers ($\mathbb{Z}$):** Integers encompass all whole numbers and their negative counterparts: $\ldots, -3, -2, -1, 0, 1, 2, 3, \ldots$. This set allows us to represent concepts like debt, temperatures below zero, or direction.

* **Rational Numbers ($\mathbb{Q}$):** These are numbers that can be expressed as a fraction $p/q$, where $p$ and $q$ are integers and $q \neq 0$. Rational numbers include all integers (e.g., $5 = 5/1$) and terminating or repeating decimals (e.g., $0.5 = 1/2$, $0.333\ldots = 1/3$).

* **Irrational Numbers ($\mathbb{I}$ or $\mathbb{R}\setminus\mathbb{Q}$):** These numbers cannot be expressed as a simple fraction of two integers. Their decimal representations are non-terminating and non-repeating. Famous examples include $\pi \approx 3.14159\ldots$, $e \approx 2.71828\ldots$, and $\sqrt{2} \approx 1.41421\ldots$.

* **Real Numbers ($\mathbb{R}$):** This set comprises all rational and irrational numbers. Real numbers can be visualized as all points on an infinitely long number line.

* **Imaginary Numbers:** These numbers arise when we consider the square root of negative numbers. The fundamental imaginary unit is $i$, defined as $i = \sqrt{-1}$. An imaginary number is of the form $bi$, where $b$ is a real number and $b \neq 0$.

* **Complex Numbers ($\mathbb{C}$):** Complex numbers extend the real numbers by including imaginary numbers. A complex number is generally expressed in the form $a + bi$, where $a$ and $b$ are real numbers. $a$ is called the real part, and $b$ is called the imaginary part. The set of real numbers is a subset of complex numbers (where $b=0$). Complex numbers are crucial in many areas of science and engineering.

---
## Setting the Stage: Constants, Variables, Functions, and Intervals

To build mathematical arguments and models, we use several key concepts:

* **Constants and Variables:**
    * A **constant** is a value that does not change. Examples include numbers like $2, \pi, e$.
    * A **variable** is a symbol (like $x, y, t$) that can represent any value from a specified set.
        * An **independent variable** is a variable whose value is not dependent on any other variable in the context of a specific problem or function; it is often the input.
        * A **dependent variable** is a variable whose value depends on one or more independent variables; it is often the output of a function.

* **Functions:** A function is a rule that assigns to each input value (from a set called the domain) exactly one output value (in a set called the codomain; the set of actual output values is the range). We often write $y = f(x)$, where $x$ is the independent variable (input), $y$ is the dependent variable (output), and $f$ is the function's name.

* **Intervals:** An interval is a set of real numbers that contains all real numbers lying between any two numbers of the set.
    * **Open Interval:** $(a, b)$ represents all real numbers $x$ such that $a < x < b$. The endpoints $a$ and $b$ are not included.
    * **Closed Interval:** $[a, b]$ represents all real numbers $x$ such that $a \le x \le b$. The endpoints $a$ and $b$ are included.
    * **Half-Open/Half-Closed Intervals:** $(a, b]$ ($a < x \le b$) or $[a, b)$ ($a \le x < b$).
    Intervals can also extend to infinity, e.g., $(a, \infty)$ or $(-\infty, b]$.

---
## The Hidden Patterns: An Introduction to Number Theory

Number theory is a branch of pure mathematics devoted primarily to the study of the integers and their properties. It explores the beautiful and often surprising relationships between these fundamental numbers.

### Divisibility, Primes, and the Foundation of Integers
* **Divisibility:** An integer $a$ **divides** an integer $b$, written $a|b$, if there exists an integer $k$ such that $b = ak$. Here, $a$ is a divisor of $b$, and $b$ is a multiple of $a$.
* **Prime Numbers:** A **prime number** is a natural number greater than 1 that has no positive divisors other than 1 and itself. A natural number greater than 1 that is not prime is called a **composite number**.
* **The Fundamental Theorem of Arithmetic:** This cornerstone of number theory states that every integer greater than 1 is either a prime number itself or can be represented as a product of prime numbers, and this representation is unique, apart from the order of the factors.

### Finding Common Ground: GCD, LCM, and the Euclidean Algorithm
* **Greatest Common Divisor (GCD):** The GCD of two integers $a$ and $b$ (not both zero), denoted $\text{gcd}(a,b)$, is the largest positive integer that divides both $a$ and $b$.
* **Least Common Multiple (LCM):** The LCM of two non-zero integers $a$ and $b$, denoted $\text{lcm}(a,b)$, is the smallest positive integer that is divisible by both $a$ and $b$.
* **Relationship:** For positive integers $a$ and $b$: $a \cdot b = \text{gcd}(a,b) \cdot \text{lcm}(a,b)$.

* **The Euclidean Algorithm:** This is a highly efficient method for computing the greatest common divisor of two integers. It is based on the principle that the greatest common divisor of two numbers does not change if the larger number is replaced by its difference with the smaller number. Repeatedly applying this, the algorithm relies on the fact that $\text{gcd}(a,b) = \text{gcd}(b, a \pmod b)$.
    * **Derivation/Steps:** To find $\text{gcd}(a,b)$ with $a>b$:
        1. Divide $a$ by $b$ and find the remainder $r_1$. So, $a = q_1 b + r_1$.
        2. Replace $a$ with $b$ and $b$ with $r_1$, and repeat the division. $b = q_2 r_1 + r_2$.
        3. Continue this process until the remainder is zero. The last non-zero remainder is the GCD.
    * **Example:** Find $\text{gcd}(1071, 462)$.
        * $1071 = 2 \cdot 462 + 147$
        * $462 = 3 \cdot 147 + 21$
        * $147 = 7 \cdot 21 + 0$
        The last non-zero remainder is 21. Thus, $\text{gcd}(1071, 462) = 21$.

### Clock Arithmetic: Modular Arithmetic and Congruences
**Modular arithmetic** is a system of arithmetic for integers, where numbers "wrap around" after they reach a certain value—the **modulus**.
* **Congruence Relation:** Two integers $a$ and $b$ are said to be **congruent modulo $n$** if their difference $(a-b)$ is an integer multiple of $n$. This is written as:
    $$ a \equiv b \pmod n $$
    This is equivalent to stating that $a$ and $b$ have the same remainder when divided by $n$. For example, $38 \equiv 14 \pmod{12}$ because $38-14=24$, which is a multiple of 12. Also, both 38 and 14 have a remainder of 2 when divided by 12.

### Solving Systems of Congruences: The Chinese Remainder Theorem
The **Chinese Remainder Theorem** provides a way to solve a system of simultaneous congruences with pairwise coprime moduli.
* **Statement:** Let $n_1, n_2, \ldots, n_k$ be integers that are pairwise coprime (i.e., $\text{gcd}(n_i, n_j)=1$ for any $i \neq j$). Then for any given sequence of integers $a_1, a_2, \ldots, a_k$, the system of congruences:
    $$ \begin{cases} x \equiv a_1 \pmod{n_1} \\ x \equiv a_2 \pmod{n_2} \\ \vdots \\ x \equiv a_k \pmod{n_k} \end{cases} $$
    has a solution, and the solution is unique modulo $N = n_1 n_2 \dots n_k$.

### Powerful Theorems of Modular Arithmetic
* **Fermat's Little Theorem:**
    * **Statement:** If $p$ is a prime number, then for any integer $a$ not divisible by $p$, the number $a^{p-1} - 1$ is an integer multiple of $p$. In modular arithmetic notation:
        $$ a^{p-1} \equiv 1 \pmod p $$
    * **Alternative Form:** For any integer $a$ and prime $p$, $a^p \equiv a \pmod p$.

* **Euler's Totient Theorem:** This theorem generalizes Fermat's Little Theorem to any modulus $n$.
    * **Euler's Totient Function $\phi(n)$:** This function counts the number of positive integers up to a given integer $n$ that are relatively prime to $n$ (i.e., $\text{gcd}(k, n)=1$ for $1 \le k \le n$). For a prime $p$, $\phi(p) = p-1$.
    * **Statement:** If $a$ and $n$ are relatively prime positive integers, then:
        $$ a^{\phi(n)} \equiv 1 \pmod n $$
    Since $\phi(p)=p-1$ for a prime $p$, Fermat's Little Theorem is a special case of Euler's Theorem.

---
## Setting the Stage: Constants, Variables, Functions, and Intervals

To build mathematical arguments and models, we use several key concepts:

* **Constants and Variables:**
    * A **constant** is a value that does not change.
    * A **variable** is a symbol that can represent any value from a specified set. We distinguish between **independent variables** (inputs) and **dependent variables** (outputs).

* **Functions:** A function is a rule that assigns to each input value from its domain exactly one output value. We write $y = f(x)$, where $x$ is the independent variable and $y$ is the dependent variable.

* **Intervals:** An interval is a set of real numbers between two given numbers. They can be **open** (endpoints not included, e.g., $(a,b)$) or **closed** (endpoints included, e.g., $[a,b]$).

---
## The Balancing Act (Or Not!): Inequalities – Comparing Quantities

Inequalities are mathematical statements that compare the relative sizes of two expressions using symbols such as:
* $<$ (less than)
* $>$ (greater than)
* $\le$ (less than or equal to)
* $\ge$ (greater than or equal to)

**Properties of Inequalities:** Understanding these properties is crucial for solving inequalities:
1.  **Addition/Subtraction:** Adding or subtracting the same real number to both sides of an inequality preserves the inequality. If $a < b$, then $a+c < b+c$ and $a-c < b-c$.
2.  **Multiplication/Division by a Positive Constant:** Multiplying or dividing both sides by the same *positive* real number preserves the inequality. If $a < b$ and $c > 0$, then $ac < bc$ and $a/c < b/c$.
3.  **Multiplication/Division by a Negative Constant:** Multiplying or dividing both sides by the same *negative* real number **reverses** the direction of the inequality. If $a < b$ and $c < 0$, then $ac > bc$ and $a/c > b/c$.
4.  **Reciprocals:** If $a$ and $b$ are both positive (or both negative) and $a < b$, then $1/a > 1/b$. If $a$ is positive and $b$ is negative, the inequality $1/a > 1/b$ still holds.
5.  **Squaring:**
    * If $a$ and $b$ are both non-negative and $a < b$, then $a^2 < b^2$.
    * If $a$ and $b$ are both negative and $a < b$ (e.g., $-3 < -2$), then $a^2 > b^2$ (e.g., $9 > 4$).
    * Care must be taken when one number is positive and the other is negative, or when dealing with variables whose sign is unknown.

---
## The Algebraic Stars: Polynomials – Expressions of Power

A **polynomial** in one variable $x$ is an expression of the form:
$$ P(x) = a_n x^n + a_{n-1} x^{n-1} + \dots + a_1 x + a_0 $$
Where:
* $a_n, a_{n-1}, \ldots, a_1, a_0$ are constants called the **coefficients**.
* If the coefficients are real numbers, it is a **real polynomial**. If they can be complex numbers, it is a **complex polynomial**.
* $n$ is a non-negative integer called the **degree** of the polynomial (if $a_n \neq 0$).
* $a_n x^n$ is the **leading term**, and $a_n$ is the **leading coefficient**.
* $a_0$ is the **constant term**.

---
## Solving the Puzzles: Polynomial Equations – Finding the Roots

A **polynomial equation** is formed when a polynomial is set equal to zero:
$$ P(x) = a_n x^n + a_{n-1} x^{n-1} + \dots + a_1 x + a_0 = 0 $$

* **Domain:** The domain of a polynomial function (and thus often considered for the equation) is typically all real numbers ($\mathbb{R}$) or all complex numbers ($\mathbb{C}$), unless specified otherwise.
* **Degree & Number of Roots:**
    * The **degree** of a polynomial equation is the highest power of the variable with a non-zero coefficient.
    * The **Fundamental Theorem of Algebra** states that every non-constant single-variable polynomial with complex coefficients has at least one complex root. An important corollary is that a polynomial of degree $n$ has exactly $n$ complex roots, counting multiplicities. If the polynomial has real coefficients, then any non-real complex roots occur in conjugate pairs ($a+bi$ and $a-bi$).
* **Roots (or Zeros) of a Polynomial:** The values of $x$ that satisfy the polynomial equation $P(x)=0$ are called the roots or zeros of the polynomial $P(x)$.

**Useful Theorems for Polynomial Equations:**

* **Remainder Theorem:** If a polynomial $P(x)$ is divided by a linear factor $(x-c)$, the remainder is $P(c)$.
    * *Proof Idea:* By the division algorithm, $P(x) = (x-c)Q(x) + R$, where $Q(x)$ is the quotient and $R$ is the remainder (a constant since we are dividing by a linear factor). Substituting $x=c$ gives $P(c) = (c-c)Q(c) + R = 0 \cdot Q(c) + R = R$.
* **Factor Theorem:** A linear expression $(x-c)$ is a factor of the polynomial $P(x)$ if and only if $P(c)=0$ (i.e., $c$ is a root of $P(x)$).
    * *Proof Idea:* This follows directly from the Remainder Theorem. If $(x-c)$ is a factor, then $P(x) = (x-c)Q(x)$, so $P(c)=0$. Conversely, if $P(c)=0$, then by the Remainder Theorem, the remainder $R=0$, so $P(x) = (x-c)Q(x)$, meaning $(x-c)$ is a factor.

---
## The Classic Curve: Quadratic Equations – Parabolas and Their Secrets

A **quadratic equation** is a polynomial equation of degree 2, with the standard form:
$$ ax^2 + bx + c = 0 \quad (a \neq 0) $$

* **Roots (Solutions):** The roots of a quadratic equation can be found using the **quadratic formula**:
    $$ x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$
    This formula is derived by completing the square.
* **Nature of Roots (Discriminant):** The term under the square root, $\Delta = b^2 - 4ac$, is called the **discriminant**. It determines the nature of the roots (assuming $a, b, c$ are real):
    * If $\Delta > 0$: Two distinct real roots.
    * If $\Delta = 0$: One real root (a repeated root, or two equal real roots).
    * If $\Delta < 0$: Two complex conjugate roots.
* **Sum and Product of Roots:** If $\alpha$ and $\beta$ are the roots of $ax^2+bx+c=0$:
    * Sum of roots: $\alpha + \beta = -\dfrac{b}{a}$
    * Product of roots: $\alpha \beta = \dfrac{c}{a}$
* **Formation of a Quadratic Equation from Roots:** If $\alpha$ and $\beta$ are given roots, the quadratic equation can be written as $k(x-\alpha)(x-\beta)=0$, or more commonly (for monic form, $k=1$):
    $$ x^2 - (\alpha + \beta)x + \alpha\beta = 0 $$
* **Graph of a Quadratic Polynomial ($y = ax^2+bx+c$):** The graph is a **parabola**.
    * If $a > 0$, the parabola opens upwards.
    * If $a < 0$, the parabola opens downwards.
    * The vertex of the parabola is at $x = -b/(2a)$. The roots of the equation $ax^2+bx+c=0$ are the x-intercepts of the graph.

---
## Beyond Single Variables: Quadratic Expressions in Two Variables

When we have quadratic expressions involving two variables, say $x$ and $y$, the general form is:
$$ ax^2 + 2hxy + by^2 + 2gx + 2fy + c = 0 $$
This equation is very powerful and can represent various **conic sections**:
* Circles
* Ellipses
* Parabolas
* Hyperbolas
* It can also represent degenerate cases like a pair of straight lines, a point, or no graph at all.
The specific type of conic section is determined by the values of the coefficients, particularly through conditions involving discriminants like $\Delta = abc + 2fgh - af^2 - bg^2 - ch^2$ and the term $h^2-ab$. A detailed classification is beyond this overview, but it's important to recognize that quadratic relationships in two variables describe these fundamental geometric shapes.

---
## Stepping Up: Cubic Equations – The Next Level of Curves

A **cubic equation** is a polynomial equation of degree 3:
$$ ax^3 + bx^2 + cx + d = 0 \quad (a \neq 0) $$

* **Nature of Roots:**
    * If the coefficients $a, b, c, d$ are real, a cubic equation must have at least one real root. The other two roots can be either both real, or a complex conjugate pair.
    * So, it can have: (i) three distinct real roots, (ii) one real root and two equal real roots (a repeated root), (iii) three equal real roots (a triple root), or (iv) one real root and two complex conjugate roots.
* **Relation between Roots and Coefficients:** If $\alpha, \beta, \gamma$ are the roots of $ax^3+bx^2+cx+d=0$:
    * Sum of roots: $\alpha + \beta + \gamma = -\dfrac{b}{a}$
    * Sum of roots taken two at a time: $\alpha\beta + \beta\gamma + \gamma\alpha = \dfrac{c}{a}$
    * Product of roots: $\alpha\beta\gamma = -\dfrac{d}{a}$
* **Solution Methods:** While a general formula (like Cardano's method) exists for solving cubic equations algebraically, it is often quite complex and unwieldy for practical use compared to the quadratic formula. Numerical methods or factorization (if simple roots can be found using the Factor Theorem) are often employed.

---
## To Infinity (Well, Almost!): Higher Degree Equations

For polynomial equations of degree $n \ge 4$:

* **Fundamental Theorem of Algebra:** A polynomial equation of degree $n$ with complex coefficients has exactly $n$ complex roots (counting multiplicities).
* **Tools for Finding Roots (especially for integer/rational roots if coefficients are integers):**
    * **Rational Root Theorem:** If a polynomial equation $a_n x^n + \dots + a_0 = 0$ with integer coefficients has a rational root $p/q$ (in simplest form), then $p$ must be a divisor of the constant term $a_0$, and $q$ must be a divisor of the leading coefficient $a_n$. This helps list possible rational roots to test.
    * **Descartes' Rule of Signs:** Provides an upper bound on the number of positive real roots and negative real roots of a polynomial with real coefficients, based on the number of sign changes in the sequence of its coefficients.
* **General Algebraic Solutions:**
    * Formulas exist for solving quartic (degree 4) equations, but they are even more complicated than for cubics.
    * The **Abel-Ruffini theorem** (and further work by Galois) famously proved that there is **no general algebraic formula** (using only arithmetic operations and radicals - roots) for finding the roots of polynomial equations of degree five or higher.
    * This doesn't mean such equations don't have roots; it just means there isn't a "one-size-fits-all" formula like the quadratic formula. For specific higher-degree equations, roots can sometimes be found by factoring or other special methods. Otherwise, **numerical methods** (like Newton-Raphson) are used to find approximate values for the roots.

---
## Key Takeaways: The Essence of Numbers, Comparisons, and Equations

This exploration has taken us through the diverse world of numbers, the logic of comparisons, and the methods for understanding and solving equations that describe so many phenomena.

* **Number Systems:** From natural numbers for counting to complex numbers for solving all polynomial equations, each system expands our mathematical toolkit.
* **Number Theory:** This field reveals profound properties of integers, including divisibility, prime numbers, and modular arithmetic, with powerful results like the Euclidean Algorithm, Chinese Remainder Theorem, and Euler's Totient Theorem.
* **Constants, Variables, Functions, Intervals:** These are fundamental tools for expressing relationships and defining scopes in mathematics.
* **Inequalities:** Provide the rules for comparing quantities, with specific properties governing their manipulation.
* **Polynomials & Their Equations:** These algebraic expressions and the equations formed from them are central to many mathematical and scientific problems. Understanding their degree, roots, and properties is key.
* **Key Theorems:** The Remainder Theorem, Factor Theorem, and Fundamental Theorem of Algebra are powerful tools for analyzing polynomials.
* **Quadratics & Cubics:** Quadratic equations have a well-known formula for their roots and describe parabolic shapes. Cubic equations have more complex root structures and specific relationships between roots and coefficients.
* **Higher Degree Equations:** While general algebraic solutions are elusive for degree 5 and above, the Fundamental Theorem of Algebra guarantees the existence of roots in the complex plane, and various theorems and numerical methods aid in their analysis.

The study of these foundational topics provides the language and techniques necessary to model and solve a vast array of problems encountered in science, engineering, and beyond.