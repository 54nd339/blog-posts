---
title: Exponential and Logarithmic Functions - Mastering Growth, Decay, and Scale
description: Explore the powerful world of exponential and logarithmic functions, from their fundamental laws and equation-solving techniques to understanding scales of magnitude. This post delves into their properties, graphs, and the historical significance of concepts like characteristic and mantissa.
date: 2022-12-28
draft: false
slug: /pensieve/maths/exponents-logarithms
tags:
  - Mathematics
  - Algebra
---

The mathematical concepts of exponential and logarithmic functions are fundamental to describing and understanding a vast array of phenomena, from population growth and radioactive decay to the Richter scale for earthquakes and the pH scale in chemistry. These functions, inverses of each other, provide powerful tools for modeling situations involving rapid change and for handling numbers that span immense ranges. This exploration delves into their definitions, fundamental laws, methods for solving related equations and inequalities, and some classical aspects of logarithmic computation.

---
## The Power Players: Exponential Functions – Growth (and Decay) on Steroids

An **exponential function** is a function in which the independent variable appears in the exponent. The general form is:
$$ f(x) = b^x $$
where $b$ is a constant called the **base**. For a valid exponential function, we require $b > 0$ and $b \neq 1$.
* If $b > 1$, the function represents **exponential growth** (e.g., $2^x, 10^x$).
* If $0 < b < 1$, the function represents **exponential decay** (e.g., $(1/2)^x = 2^{-x}$).

A particularly important base is Euler's number, **$e \approx 2.71828\ldots$**, leading to the **natural exponential function** $f(x) = e^x$.

**Characteristics and Graph:**
* **Domain:** All real numbers ($-\infty, \infty$).
* **Range:** All positive real numbers $(0, \infty)$.
* **Y-intercept:** $(0, 1)$, since $b^0 = 1$.
* **Asymptote:** The x-axis ($y=0$) is a horizontal asymptote.
* **Monotonicity:**
    * Increasing if $b > 1$.
    * Decreasing if $0 < b < 1$.
The graph of $y=b^x$ passes through $(0,1)$ and $(1,b)$.

### Fundamental Laws of Exponentials (Properties of Exponents)
These laws govern how exponential expressions are manipulated:
1.  $b^x \cdot b^y = b^{x+y}$ (Product of powers)
2.  $\dfrac{b^x}{b^y} = b^{x-y}$ (Quotient of powers)
3.  $(b^x)^y = b^{xy}$ (Power of a power)
4.  $(ab)^x = a^x b^x$ (Power of a product)
5.  $\left(\dfrac{a}{b}\right)^x = \dfrac{a^x}{b^x}$ (Power of a quotient)
6.  $b^0 = 1$ (Zero exponent)
7.  $b^{-x} = \dfrac{1}{b^x}$ (Negative exponent)
8.  $b^{x/y} = \sqrt[y]{b^x} = (\sqrt[y]{b})^x$ (Fractional exponent)

---
## Taming the Titans: Logarithmic Functions – The Inverse Power

A **logarithmic function** is the inverse of an exponential function. If $b^y = x$, then the logarithm of $x$ to base $b$ is $y$. This is written as:
$$ \log_b x = y \iff b^y = x $$
Here:
* $b$ is the **base** of the logarithm ($b > 0, b \neq 1$).
* $x$ is the **argument** of the logarithm ($x > 0$). The argument must always be positive.
The logarithm, $y$, is the exponent to which the base $b$ must be raised to obtain $x$.

**Common Types:**
* **Common Logarithm:** Base 10, denoted $\log x$ (or $\log_{10} x$). Used in scales like Richter, pH, decibels.
* **Natural Logarithm:** Base $e$, denoted $\ln x$ (or $\log_e x$). Arises naturally in calculus and many scientific formulas.

**Characteristics and Graph:**
The graph of $y=\log_b x$ is the reflection of $y=b^x$ across the line $y=x$.
* **Domain:** All positive real numbers $(0, \infty)$.
* **Range:** All real numbers $(-\infty, \infty)$.
* **X-intercept:** $(1, 0)$, since $\log_b 1 = 0$ (because $b^0=1$).
* **Asymptote:** The y-axis ($x=0$) is a vertical asymptote.
* **Monotonicity:**
    * Increasing if $b > 1$.
    * Decreasing if $0 < b < 1$.
The graph of $y=\log_b x$ passes through $(1,0)$ and $(b,1)$.

### Fundamental Laws of Logarithms (Properties)
These laws are derived from the laws of exponents:
1.  **Product Rule:** $\log_b (xy) = \log_b x + \log_b y$ (for $x>0, y>0$)
2.  **Quotient Rule:** $\log_b \left(\dfrac{x}{y}\right) = \log_b x - \log_b y$ (for $x>0, y>0$)
3.  **Power Rule:** $\log_b (x^p) = p \log_b x$ (for $x>0$)
4.  **Change of Base Formula:** Allows conversion from one base to another. For any suitable base $c$:
    $$ \log_b x = \dfrac{\log_c x}{\log_c b} $$
    Commonly used forms: $\log_b x = \dfrac{\ln x}{\ln b}$ or $\log_b x = \dfrac{\log x}{\log b}$.
5.  **Identity Properties:**
    * $\log_b b = 1$ (since $b^1=b$)
    * $\log_b 1 = 0$ (since $b^0=1$)
6.  **Inverse Properties:**
    * $\log_b (b^x) = x$
    * $b^{\log_b x} = x$ (for $x>0$)

---
## Solving the Puzzles: Exponential and Logarithmic Equations

These equations involve variables in exponents or within logarithms.

### Exponential Equations
1.  **Same Base or Convertible to Same Base:** If an equation can be written as $b^{f(x)} = b^{g(x)}$, then since $b^x$ is a one-to-one function, we can equate the exponents: $f(x) = g(x)$.
    Example: $2^{x+1} = 8 \implies 2^{x+1} = 2^3 \implies x+1=3 \implies x=2$.
2.  **Different Bases (Cannot be made same easily):** Take the logarithm (common or natural) of both sides.
    Example: $3^x = 5^{x-1}$.
    $\ln(3^x) = \ln(5^{x-1}) \implies x\ln 3 = (x-1)\ln 5$.
    $x\ln 3 = x\ln 5 - \ln 5 \implies \ln 5 = x(\ln 5 - \ln 3) \implies x = \dfrac{\ln 5}{\ln 5 - \ln 3}$.
3.  **Quadratic in Form:** Equations like $A(b^x)^2 + B(b^x) + C = 0$. Let $y=b^x$, then $Ay^2 + By + C = 0$. Solve for $y$, then solve $b^x=y$ for $x$. Remember $y=b^x$ must be positive.

### Logarithmic Equations
Key strategy: Use properties of logarithms to combine terms into a single logarithm on one or both sides, then convert to exponential form or equate arguments. **Always check for extraneous solutions** by ensuring that the arguments of all original logarithmic expressions are positive for the solutions found.
1.  **$\log_b f(x) = \log_b g(x)$:** If the bases are the same, then $f(x) = g(x)$. Solve and verify solutions in the original domains ($f(x)>0, g(x)>0$).
2.  **$\log_b f(x) = c$:** Convert to exponential form: $f(x) = b^c$. Solve and verify $f(x)>0$.
    Example: $\log_2 (x-3) + \log_2 (x-1) = 3$.
    Domain: $x-3>0 \implies x>3$ AND $x-1>0 \implies x>1$. So, $x>3$.
    $\log_2 ((x-3)(x-1)) = 3 \implies (x-3)(x-1) = 2^3 = 8$.
    $x^2 - 4x + 3 = 8 \implies x^2 - 4x - 5 = 0$.
    $(x-5)(x+1) = 0 \implies x=5$ or $x=-1$.
    Check domain: $x=5$ is valid ($>3$). $x=-1$ is not valid ($<3$). So, solution is $x=5$.

---
## The Balancing Act: Exponential and Logarithmic Inequalities

Solving inequalities involving these functions requires care, especially regarding the base.

### Exponential Inequalities
1.  **If base $b > 1$:** The exponential function $y=b^x$ is increasing. The inequality sign is preserved when comparing exponents.
    $b^{f(x)} > b^{g(x)} \implies f(x) > g(x)$.
2.  **If $0 < b < 1$:** The exponential function $y=b^x$ is decreasing. The inequality sign is **reversed** when comparing exponents.
    $b^{f(x)} > b^{g(x)} \implies f(x) < g(x)$.
3.  **Using Logarithms:** If bases are different or complex, one can take logarithms. If taking $\log_c$ where $c>1$, the inequality is preserved. If $0<c<1$, it's reversed. Always ensure arguments are positive if they involve variables.

### Logarithmic Inequalities
**Crucial First Step: Determine the Domain!** The argument of every logarithm must be strictly positive.
1.  **If base $b > 1$:** The logarithmic function $y=\log_b x$ is increasing. The inequality sign is preserved when comparing arguments.
    $\log_b f(x) > \log_b g(x) \implies f(x) > g(x)$.
    Also, ensure $f(x)>0$ and $g(x)>0$.
2.  **If $0 < b < 1$:** The logarithmic function $y=\log_b x$ is decreasing. The inequality sign is **reversed** when comparing arguments.
    $\log_b f(x) > \log_b g(x) \implies f(x) < g(x)$.
    Also, ensure $f(x)>0$ and $g(x)>0$.
3.  **$\log_b f(x) > c$ (or $<, \le, \ge$):** Convert to exponential form, carefully considering the base.
    * If $b>1$: $\log_b f(x) > c \implies f(x) > b^c$.
    * If $0<b<1$: $\log_b f(x) > c \implies f(x) < b^c$. (Remember $f(x)>0$ from domain).
    The final solution must satisfy both the inequality derived and the domain restrictions.

---
## Decoding Logarithms: Characteristic and Mantissa (Primarily for Common Logarithms)

Before calculators became ubiquitous, **common logarithms** (base 10, $\log_{10} N$ or simply $\log N$) were essential for complex calculations (multiplication, division, powers, roots) using log tables. The structure of a common logarithm value was split into two parts:

Any positive number $N$ can be expressed in scientific notation as $N = m \times 10^p$, where $1 \le m < 10$ and $p$ is an integer.
Then, $\log_{10} N = \log_{10} (m \times 10^p) = \log_{10} m + \log_{10} (10^p) = \log_{10} m + p$.

* **Characteristic:** The integer part of the logarithm, which is $p$.
    * It determines the position of the decimal point in the number $N$.
    * If $N > 1$: The characteristic is one less than the number of digits to the left of the decimal point. (e.g., $\log_{10} 345.6$. Here $345.6 = 3.456 \times 10^2$. Characteristic is $2$).
    * If $0 < N < 1$: The characteristic is negative. Its magnitude is one more than the number of zeros immediately following the decimal point before the first non-zero digit. (e.g., $\log_{10} 0.03456$. Here $0.03456 = 3.456 \times 10^{-2}$. Characteristic is $-2$).

* **Mantissa:** The non-negative decimal (fractional) part of the logarithm, which is $\log_{10} m$.
    * Since $1 \le m < 10$, we have $\log_{10} 1 \le \log_{10} m < \log_{10} 10$, so $0 \le \text{mantissa} < 1$.
    * The mantissa depends only on the sequence of digits in $N$, not on the position of the decimal point. It was found using logarithm tables.

Example: $\log_{10} 345.6 = \log_{10} (3.456 \times 10^2) = \log_{10} 3.456 + \log_{10} 10^2 = 0.5386 + 2 = 2.5386$.
Characteristic = 2, Mantissa = 0.5386.

The term "characteristic and mantissa" is not standard terminology for describing general exponential functions like $y=b^x$ in the same way it applies to the values of common logarithms.

---
## When Things Get Negative: Negative Characteristics (for Logarithms) and Antilogarithms

* **Negative Characteristics:**
    When the logarithm of a number between 0 and 1 is calculated, the value is negative. For example, $\log_{10} 0.02 \approx -1.69897$.
    To maintain a positive mantissa for use with log tables, this was often written by separating the negative integer part (characteristic) and the positive fractional part (mantissa).
    $-1.69897 = -2 + (2 - 1.69897) = -2 + 0.30103$.
    This is written as $\bar{2}.30103$. The "bar 2" indicates a characteristic of -2, and the mantissa is $0.30103$.
    So, Characteristic = $\bar{2}$ (or -2), Mantissa = $0.30103$.

* **Antilogarithm (Antilog):**
    The antilogarithm is the inverse operation of finding a logarithm. If $\log_b x = y$, then $x$ is the antilogarithm of $y$ to the base $b$.
    $$ x = \text{antilog}_b (y) = b^y $$
    For common logarithms (base 10): If $\log_{10} N = y$, then $N = \text{antilog}_{10}(y) = 10^y$.
    This was used with logarithm tables to find the number $N$ whose logarithm value was known.
    If $\log N = C.M$ (Characteristic $C$, Mantissa $M$), then $N = (\text{antilog}_{10} M) \times 10^C$. The antilog of the mantissa (a number between 1 and 10) was found from antilog tables, and the characteristic determined the decimal point's position.

While calculators have largely superseded the manual use of characteristics and mantissas with log tables, understanding the concept provides insight into the structure of logarithms and their historical application in computation.

---
## Key Takeaways: Harnessing Exponents and Logarithms

Exponential and logarithmic functions are fundamental tools in mathematics, describing phenomena ranging from rapid growth and decay to measurements across vast scales.

* **Inverse Relationship:** Exponential functions ($y=b^x$) and logarithmic functions ($y=\log_b x$) are inverses of each other. What one does, the other undoes.
* **Fundamental Laws:** Each type of function has a set of algebraic laws (properties of exponents for exponentials, properties of logarithms for logs) that govern their manipulation and simplification. These laws are interconnected.
* **Solving Equations & Inequalities:** Specific strategies exist for solving equations and inequalities involving these functions, often relying on their one-to-one nature (for a given base) and the careful application of their properties. Domain considerations are crucial for logarithms, and the base value ($>1$ or $0<b<1$) is critical for inequalities.
* **Scaling and Measurement:** Logarithms, particularly common (base 10) logarithms, are invaluable for handling numbers spanning many orders of magnitude and form the basis of logarithmic scales (e.g., pH, Richter, decibels).
* **Historical Computation:** The concepts of characteristic and mantissa were vital for performing complex calculations using logarithm tables before the advent of modern calculators and computers.

Mastering exponential and logarithmic functions unlocks a deeper understanding of mathematical relationships and provides powerful methods for modeling and analyzing the world around us.