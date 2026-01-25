---
title: Limits and Derivatives - The Foundation of Calculus and the Science of Change
description: Delve into the core concepts of differential calculus, from the intuitive idea of a limit to the powerful applications of the derivative. This post explores the definitions, rules, and theorems that allow us to analyze rates of change, optimize functions, and understand the dynamic nature of the world.
date: 2023-01-09
draft: false
slug: /pensieve/maths/limits-derivatives
tags:
  - Mathematics
  - Calculus
---

Calculus, a monumental achievement in mathematics, provides the tools to understand change and motion. Its development revolutionized science, engineering, and economics. At the very foundation of this powerful discipline lie two interconnected concepts: **limits** and **derivatives**. The concept of a limit allows us to analyze the behavior of functions as they approach a certain point, providing a rigorous basis for continuity. The derivative, built upon this foundation, gives us a precise way to measure instantaneous rates of change. This exploration delves into these fundamental ideas, their properties, rules of evaluation, and their far-reaching applications.

---
## The Concept of a Limit – Getting Infinitesimally Close

The idea of a limit is central to all of calculus. It formalizes the notion of "approaching" a value.

* **Informal Definition:** We say the limit of a function $f(x)$ as $x$ approaches a value $c$ is $L$, written $\lim_{x \to c} f(x) = L$, if we can make the value of $f(x)$ arbitrarily close to $L$ by choosing $x$ sufficiently close to $c$ (but not equal to $c$).

* **Neighborhood (NBD) of a Point:** An open interval containing a point is called a neighborhood of that point. For a point $c$, a $\delta$-neighborhood is the set of points $(c-\delta, c+\delta)$ for some $\delta > 0$. A deleted neighborhood excludes the point $c$ itself.

* **Formal ($\epsilon$-$\delta$) Definition of a Limit:**
    $\lim_{x \to c} f(x) = L$ if for every number $\epsilon > 0$, there exists a corresponding number $\delta > 0$ such that if $0 < |x-c| < \delta$, then $|f(x)-L| < \epsilon$.

* **One-Sided Limits:**
    * **Left-Hand Limit:** $\lim_{x \to c^-} f(x) = L_1$, where $x$ approaches $c$ from values less than $c$.
    * **Right-Hand Limit:** $\lim_{x \to c^+} f(x) = L_2$, where $x$ approaches $c$ from values greater than $c$.

* **Existence of a Limit:** A limit exists if and only if the left-hand and right-hand limits both exist and are equal.
    $$ \lim_{x \to c} f(x) = L \iff \lim_{x \to c^-} f(x) = \lim_{x \to c^+} f(x) = L $$

* **Indeterminate Forms:** When direct substitution yields forms like $\dfrac{0}{0}, \dfrac{\infty}{\infty}, 0 \cdot \infty, \infty - \infty, 1^\infty, 0^0, \infty^0$, the limit is said to be in an indeterminate form and requires further analysis.

### The Algebra of Limits
If $\lim_{x \to c} f(x) = L$ and $\lim_{x \to c} g(x) = M$, then:
1.  **Sum/Difference Rule:** $\lim_{x \to c} [f(x) \pm g(x)] = L \pm M$.
2.  **Product Rule:** $\lim_{x \to c} [f(x)g(x)] = LM$.
3.  **Constant Multiple Rule:** $\lim_{x \to c} [kf(x)] = kL$.
4.  **Quotient Rule:** $\lim_{x \to c} \dfrac{f(x)}{g(x)} = \dfrac{L}{M}$, provided $M \neq 0$.
5.  **Power Rule:** $\lim_{x \to c} [f(x)]^[g(x)] = L^M$.
6.  **Absolute Value Rule:** $\lim_{x \to c} |f(x)| = |L|$.
7.  **Composition Rule:** If $\lim_{x \to c} g(x) = L$ and $f$ is continuous at $L$, then $\lim_{x \to c} f(g(x)) = f(L)$.
8.  **Limits at Infinity:** If $f(x)$ approaches $L$ as $x \to \infty$, we write $\lim_{x \to \infty} \dfrac{1}{f(x)} = 0$.
9.  **Inequality Limits:** If $g(x) \le f(x)$ for all $x$ in a neighborhood of $c$, then $\lim_{x \to c} g(x) \le \lim_{x \to c} f(x)$.

* **Sandwich (Squeeze) Theorem:** If $g(x) \le f(x) \le h(x)$ for all $x$ in an open interval containing $c$ (except possibly at $c$ itself), and if $\lim_{x \to c} g(x) = \lim_{x \to c} h(x) = L$, then $\lim_{x \to c} f(x) = L$.

### Powerful Tools for Evaluating Limits
* **Expansions (Taylor Series):** For limits as $x \to 0$, using the initial terms of the Taylor/Maclaurin series of functions can simplify evaluation.
    * $e^x = 1 + x + \dfrac{x^2}{2!} + \dots$
    * $\ln(1+x) = x - \dfrac{x^2}{2} + \dfrac{x^3}{3} - \dots$
    * $a^x = 1 + x \ln a + \dfrac{x^2 (\ln a)^2}{2!} + \dots$ for $a > 0$.
    * $\sin x = x - \dfrac{x^3}{3!} + \dfrac{x^5}{5!} - \dots$
    * $\cos x = 1 - \dfrac{x^2}{2!} + \dfrac{x^4}{4!} - \dots$
    * $\tan x = x + \dfrac{x^3}{3} + \dfrac{2x^5}{15} + \dots$
* **L'Hôpital's Rule:** If $\lim_{x \to c} f(x)/g(x)$ produces an indeterminate form $0/0$ or $\infty/\infty$, and $f,g$ are differentiable near $c$ (with $g'(x) \neq 0$), then:
    $$ \lim_{x \to c} \dfrac{f(x)}{g(x)} = \lim_{x \to c} \dfrac{f'(x)}{g'(x)} $$
    provided the limit on the right exists or is $\pm\infty$.

### Evaluation of Limits: A Practical Guide
#### Algebraic Limits
* **Direct Substitution:** If the function is continuous at the point, simply substitute the value.
* **Factoring and Canceling:** For rational functions resulting in $0/0$, factor the numerator and denominator and cancel common factors.
* **Rationalizing:** For limits involving square roots, multiply the numerator and denominator by the conjugate to simplify.
* **Limits at Infinity:** For rational functions, divide the numerator and denominator by the highest power of $x$ in the denominator.
* **Standard Algebraic Limit:**
    $$ \lim_{x \to a} \dfrac{x^n - a^n}{x-a} = na^{n-1} $$
    **Derivation:** Let $x=a+h$. As $x \to a$, $h \to 0$. The limit becomes $\lim_{h \to 0} \dfrac{(a+h)^n - a^n}{h}$. Using the binomial expansion for $(a+h)^n = a^n + na^{n-1}h + \binom{n}{2}a^{n-2}h^2 + \dots + h^n$, the expression is $\lim_{h \to 0} \dfrac{na^{n-1}h + (\text{terms with } h^2 \text{ or higher})}{h} = \lim_{h \to 0} (na^{n-1} + (\text{terms with } h)) = na^{n-1}$.

#### Trigonometric, Exponential, and Logarithmic Limits
Several standard limits are fundamental:
* $$ \lim_{x \to 0} \dfrac{\sin (x-a)}{x-a} = 1 $$
    **Derivation (Geometric):** Consider a unit circle and an angle $x > 0$ (in radians). The area of $\triangle OAP \le$ Area of sector $OAP \le$ Area of $\triangle OAT$. This gives $\dfrac{1}{2}\sin x \le \dfrac{1}{2}x \le \dfrac{1}{2}\tan x$. For $x>0$, this simplifies to $\cos x \le \dfrac{\sin x}{x} \le 1$. As $x \to 0^+$, $\cos x \to 1$. By the Sandwich Theorem, $\lim_{x \to 0^+} \dfrac{\sin x}{x} = 1$. The result holds for $x \to 0^-$ as well.
* $$ \lim_{x \to 0} \dfrac{\sin^{-1} (x-a)}{x-a} = 1 $$
* $$ \lim_{x \to 0} \dfrac{1-\cos x}{x^2} = \dfrac{1}{2} $$
* $$ \lim_{x \to 0} \dfrac{a^x - 1}{x} = \ln{a} $$
* $$ \lim_{x \to 0} \dfrac{\ln(1+x)}{x} = 1 $$
* $$ \lim_{x \to 0} (1+x)^{1/x} = e $$

---
## Staying Connected: Continuity – No Jumps, No Gaps

* **Definition:** A function $f$ is **continuous** at a point $c$ if it meets three conditions:
    1.  $f(c)$ is defined.
    2.  $\lim_{x \to c} f(x)$ exists.
    3.  $\lim_{x \to c} f(x) = f(c)$.
    Intuitively, a function is continuous if its graph can be drawn without lifting the pen from the paper.
* **Directional Continuity:**
    * **Right-continuous** at $c$ if $\lim_{x \to c^+} f(x) = f(c)$.
    * **Left-continuous** at $c$ if $\lim_{x \to c^-} f(x) = f(c)$.
* **Continuity in an Interval:** A function is continuous on an open interval $(a,b)$ if it is continuous at every point in the interval. For a closed interval $[a,b]$, it must be continuous on $(a,b)$, right-continuous at $a$, and left-continuous at $b$.

### Types of Discontinuities
1.  **Removable Discontinuity:** The limit exists at the point, but is not equal to the function's value (either because $f(c)$ is different, or not defined). This corresponds to a "hole" in the graph which can be "plugged."
2.  **Non-Removable Discontinuity:**
    * **Jump Discontinuity:** The left-hand and right-hand limits exist but are not equal. The graph "jumps" from one value to another.
    * **Infinite Discontinuity:** One or both of the one-sided limits tend to $\pm\infty$. The graph has a vertical asymptote.

### Continuity of Special Functions
* **Polynomials, Sine, Cosine, Exponential functions:** Continuous everywhere.
* **Rational functions:** Continuous on their domains (discontinuous where the denominator is zero).
* **Greatest Integer Function, $f(x)=[x]$:** Discontinuous (jump) at every integer value.
* **Signum Function, $f(x)=\text{sgn}(x)$:** Discontinuous (jump) at $x=0$.
* **Absolute Value Function, $f(x)=|x|$:** Continuous everywhere.
* **Composite Functions:** If $g$ is continuous at $c$ and $f$ is continuous at $g(c)$, then the composite function $f \circ g$ is continuous at $c$.

### Intermediate Value Theorem (IVT)
If a function $f$ is continuous on a closed interval $[a,b]$, and $N$ is any number between $f(a)$ and $f(b)$ (where $f(a) \neq f(b)$), then there exists at least one number $c$ in the open interval $(a,b)$ such that $f(c)=N$. In essence, a continuous function must take on all values between its endpoints.

---
## Measuring Change: Differentiability – Smoothness and Slopes

Differentiability is a stronger condition than continuity. It implies that a function is not only connected but also "smooth" with no sharp corners or cusps.

* **Definition of the Derivative:** The derivative of a function $f(x)$ with respect to $x$, denoted $f'(x)$ or $\dfrac{dy}{dx}$, is the function:
    $$ f'(x) = \lim_{h \to 0} \dfrac{f(x+h) - f(x)}{h} $$
    provided the limit exists. This is known as differentiation from the **first principle**.
* **Differentiability:** A function is differentiable at a point if its derivative exists at that point.
    * **Existence:** For the derivative to exist, the left-hand derivative and right-hand derivative must exist and be equal.
* **Differentiability implies Continuity:** If a function is differentiable at a point, it must be continuous at that point. The converse is not true (e.g., $f(x)=|x|$ is continuous at $x=0$ but not differentiable there).

### Geometric Interpretation of the Derivative
1.  **Slope of the Tangent Line:** The derivative $f'(a)$ gives the slope of the tangent line to the curve $y=f(x)$ at the point $(a, f(a))$.
2.  **Instantaneous Rate of Change:** The derivative represents the instantaneous rate of change of the function with respect to its variable. For position function $s(t)$, $s'(t)$ is the instantaneous velocity.

### Standard Derivatives & Theorems of Differentiation (Differentiation Rules)
* **Theoroms:**
    * Addition of differentiable and non-differentiable functions is always non-differentiable.
    * Product of differentiable and non-differentiable functions maybe differentiable or non-differentiable.
    * If $g(x)$ is a differentiable function and $f(x) = |g(x)|$ is a non-differentiable function at $x = a$, then $g(a) = 0$.
    * If both $f(X)$ and $g(X)$ are non-differentiable at $x = a$, then $f(x) + g(x)$ may be differentiable or non-differentiable at $x = a$.
* **Rules:**
    * **Sum/Difference Rule:** $(f \pm g)' = f' \pm g'$
    * **Product Rule:** $(fg)' = f'g + fg'$
        *Derivation:* Let $P(x)=f(x)g(x)$.
        $P'(x) = \lim_{h \to 0} \dfrac{f(x+h)g(x+h) - f(x)g(x)}{h}$
        $= \lim_{h \to 0} \dfrac{f(x+h)g(x+h) - f(x+h)g(x) + f(x+h)g(x) - f(x)g(x)}{h}$
        $= \lim_{h \to 0} \left[ f(x+h)\dfrac{g(x+h)-g(x)}{h} + g(x)\dfrac{f(x+h)-f(x)}{h} \right]$
        $= (\lim_{h \to 0}f(x+h))(\lim_{h \to 0}\dfrac{g(x+h)-g(x)}{h}) + g(x)(\lim_{h \to 0}\dfrac{f(x+h)-f(x)}{h})$
        $= f(x)g'(x) + g(x)f'(x)$.
    * **Quotient Rule:** $\left(\dfrac{f}{g}\right)' = \dfrac{f'g - fg'}{g^2}$
    * **Chain Rule:** For a composite function $F(x) = f(g(x))$, its derivative is:
        $$ F'(x) = f'(g(x)) \cdot g'(x) \quad \text{or} \quad \dfrac{dy}{dx} = \dfrac{dy}{du} \cdot \dfrac{du}{dx} $$
* **Standard Derivatives:**
    * $\dfrac{d}{dx}(c) = 0$
    * $\dfrac{d}{dx}(x^n) = nx^{n-1}, x, n \in \mathbb{R}, x \gt 0$ (Power Rule)
    * $\dfrac{d}{dx}(e^x) = e^x$
    * $\dfrac{d}{dx}(e^{ax}) = ae^{ax}$
    * $\dfrac{d}{dx}(a^x) = a^x \ln{a}$
    * $\dfrac{d}{dx}(\ln|x|) = 1/x$
    * $\dfrac{d}{dx}(\log_a x) = \dfrac{1}{x \ln a}$
    * $\dfrac{d}{dx}(\sin x) = \cos x$, $\dfrac{d}{dx}(\cos x) = -\sin x$
    * $\dfrac{d}{dx}(\tan x) = \sec^2 x$, $\dfrac{d}{dx}(\cot x) = -\csc^2 x$
    * $\dfrac{d}{dx}(\sec x) = \sec x \tan x$, $\dfrac{d}{dx}(\csc x) = -\csc x \cot x$
    * $\dfrac{d}{dx}(\sin^{-1} x) = \dfrac{1}{\sqrt{1-x^2}}$, $\dfrac{d}{dx}(\cos^{-1} x) = -\dfrac{1}{\sqrt{1-x^2}}$
    * $\dfrac{d}{dx}(\tan^{-1} x) = \dfrac{1}{1+x^2}$, $\dfrac{d}{dx}(\cot^{-1} x) = -\dfrac{1}{1+x^2}$
    * $\dfrac{d}{dx}(\sec^{-1} x) = \dfrac{1}{|x|\sqrt{x^2-1}}$, $\dfrac{d}{dx}(\csc^{-1} x) = -\dfrac{1}{|x|\sqrt{x^2-1}}$
* **Some Standard Substitutions:**
    | **Expression**    | **Substitution**    |
    |-------------------|---------------------|
    | $x^2 + a^2$       | $x = a \tan \theta$ |
    | $x^2 - a^2$       | $x = a \sec \theta$ |
    | $a^2 - x^2$       | $x = a \sin \theta$ |
    | $\dfrac{a+x}{a-x}$ | $x = a \cos \theta$ |

---
## Advanced Differentiation Techniques

* **Implicit Differentiation:** Used to find $dy/dx$ for relations where $y$ is not explicitly defined as a function of $x$ (e.g., $x^2+y^2=r^2$). Differentiate both sides of the equation with respect to $x$, treating $y$ as a function of $x$, and then solve for $dy/dx$.
* **Logarithmic Differentiation:** Used for functions with variables in both the base and exponent (e.g., $y=x^x$) or for complex products/quotients. Take the natural logarithm of both sides, use log properties to simplify, then differentiate implicitly.
* **Differentiation of Parametric Functions:** If a curve is defined by $x=f(t), y=g(t)$:
    $$ \dfrac{dy}{dx} = \dfrac{dy/dt}{dx/dt} = \dfrac{g'(t)}{f'(t)} $$
* **Differentiation of one function with respect to another:** To find the derivative of $u(x)$ with respect to $v(x)$:
    $$ \dfrac{du}{dv} = \dfrac{du/dx}{dv/dx} $$
* **Differentiation of Determinants:** For a matrix $A(t)$, the derivative of its determinant with respect to $t$ can be computed using the formula:
    $$ \dfrac{d}{dt} \det(A(t)) = \det(A(t)) \cdot \text{tr}(A^{-1}(t) A'(t)) $$
    where $\text{tr}$ denotes the trace of a matrix.
* **Higher Order Derivatives:** The second derivative is the derivative of the first derivative ($f''(x) = \dfrac{d}{dx}(f'(x))$), and so on for $f'''(x), f^{(n)}(x)$.
* **Partial Derivatives:** For a function of multiple variables, like $f(x,y)$, the partial derivative with respect to one variable (e.g., $\dfrac{\partial f}{\partial x}$) is found by differentiating with respect to that variable while treating all other variables as constants.

---
## Geometric Applications of Derivatives

* **Equation of Tangents & Normals to a Curve $y=f(x)$ at $(x_1, y_1)$:**
    * **Tangent:** $y - y_1 = f'(x_1)(x-x_1)$.
    * **Normal:** $y - y_1 = -\dfrac{1}{f'(x_1)}(x-x_1)$ (if $f'(x_1) \neq 0$).
* **Length of Tangent, Normal, Subtangent, and Subnormal:** These are lengths of specific line segments related to the tangent and normal at a point $(x_1, y_1)$ on a curve. They can be calculated using $y_1$ and the slope $m=y'_1$.
    * Length of Tangent: $|y_1/m|\sqrt{1+m^2}$
    * Length of Normal: $|y_1|\sqrt{1+m^2}$
    * Length of Subtangent: $|y_1/m|$
    * Length of Subnormal: $|y_1 m|$
* **Angle between Curves:** The angle between two intersecting curves is the angle between their tangents at the point of intersection.
    $$ \tan \theta = \left| \dfrac{m_1 - m_2}{1 + m_1 m_2} \right| $$
    where $m_1$ and $m_2$ are the slopes of the tangents to the curves at the point of intersection.
* **Curvature ($\kappa$) and Radius of Curvature ($R$):**
    Curvature measures how sharply a curve is bending. For a function $y=f(x)$:
    $$ \kappa = \dfrac{|y''|}{(1+(y')^2)^{3/2}} $$
    The radius of curvature is the radius of the "kissing circle" that best approximates the curve at that point: $R = 1/\kappa$.

---
## The Big Theorems of Calculus: Mean Value Theorems

These theorems provide fundamental links between the derivative of a function and its values over an interval.

* **Rolle's Theorem:** If a function $f$ is:
    1.  Continuous on the closed interval $[a,b]$,
    2.  Differentiable on the open interval $(a,b)$, and
    3.  $f(a) = f(b)$,
    then there exists at least one number $c$ in $(a,b)$ such that $f'(c)=0$.
    (Geometrically, there's at least one point where the tangent is horizontal).
* **Lagrange's Mean Value Theorem (MVT):** If a function $f$ is:
    1.  Continuous on $[a,b]$, and
    2.  Differentiable on $(a,b)$,
    then there exists at least one number $c$ in $(a,b)$ such that:
    $$ f'(c) = \dfrac{f(b) - f(a)}{b-a} $$
    (Geometrically, there's at least one point where the instantaneous rate of change (slope of tangent) equals the average rate of change (slope of secant line connecting endpoints)).
    **Proof Sketch:** Apply Rolle's Theorem to the auxiliary function $g(x) = f(x) - \left[\dfrac{f(b)-f(a)}{b-a}\right](x-a)$.
* **Cauchy's Mean Value Theorem:** A generalization of MVT for two functions. If $f$ and $g$ are continuous on $[a,b]$ and differentiable on $(a,b)$, and $g'(x) \neq 0$ for all $x \in (a,b)$, then there exists at least one $c \in (a,b)$ such that:
    $$ \dfrac{f(b) - f(a)}{g(b) - g(a)} = \dfrac{f'(c)}{g'(c)} $$

---
## Analyzing Function Behavior: Monotonicity and Extrema

Derivatives are the ultimate tool for analyzing the behavior of functions.

* **Monotonicity:**
    * **Increasing/Decreasing:** A function is increasing on an interval if its values rise as $x$ increases, and decreasing if its values fall.
    * **Test:** Let $f$ be differentiable on an interval $I$:
        * If $f'(x) > 0$ for all $x \in I$, then $f$ is strictly increasing on $I$.
        * If $f'(x) < 0$ for all $x \in I$, then $f$ is strictly decreasing on $I$.
* **Point of Inflection:** A point on a curve at which the **concavity** changes (from concave up to concave down, or vice versa).
    * **Condition:** If $(c, f(c))$ is a point of inflection, then either $f''(c)=0$ or $f''(c)$ does not exist. The second derivative $f''(x)$ must change sign around $x=c$.
* **Extremum (Local/Relative Maxima and Minima):**
    * **Definition:** A local maximum occurs at a peak, and a local minimum at a valley.
    * **Critical Points:** A point $c$ in the domain of $f$ is a critical point if either $f'(c)=0$ or $f'(c)$ is undefined. Local extrema can only occur at critical points.
    * **First Derivative Test:** Let $c$ be a critical point of $f$:
        * If $f'$ changes from positive to negative at $c$, then $f$ has a local maximum at $c$.
        * If $f'$ changes from negative to positive at $c$, then $f$ has a local minimum at $c$.
    * **Second Derivative Test:** Let $c$ be a critical point where $f'(c)=0$:
        * If $f''(c) < 0$, then $f$ has a local maximum at $c$.
        * If $f''(c) > 0$, then $f$ has a local minimum at $c$.
        * If $f''(c) = 0$, the test is inconclusive.
    * **Global (Absolute) Extrema:** To find the absolute maximum/minimum of a continuous function on a closed interval $[a,b]$, one must evaluate the function at all critical points within the interval and also at the endpoints $a$ and $b$. The largest of these values is the absolute maximum, and the smallest is the absolute minimum.

### Applications of Extrema (Optimization)
Finding maxima and minima is crucial for **optimization problems** across numerous fields:
* **Geometry:** Finding the maximum area of a shape for a given perimeter.
* **Physics:** Finding the trajectory that requires minimum energy (principle of least action).
* **Engineering:** Designing structures for maximum strength with minimum material.
* **Economics:** Maximizing profit or minimizing cost.

---
## Key Takeaways: The Essence of Calculus

The development of limits and derivatives marked a turning point in mathematical history, providing a dynamic framework to describe a world in constant flux.

* **Limits as the Foundation:** The concept of a limit underpins all of calculus, providing a rigorous way to handle the infinitesimally small and to define continuity.
* **Derivatives as Instantaneous Change:** The derivative of a function gives its instantaneous rate of change at any point, which is geometrically interpreted as the slope of the tangent line to its graph.
* **A Toolkit of Rules:** A systematic set of differentiation rules (Power, Product, Quotient, Chain) allows for the efficient calculation of derivatives for a vast array of functions.
* **Analyzing Functions:** Derivatives are the ultimate tool for analyzing function behavior, including determining where a function is increasing or decreasing (monotonicity), its concavity and points of inflection, and locating its maximum and minimum values (extrema).
* **Mean Value Theorems:** These theorems (Rolle's, Lagrange's MVT, Cauchy's) provide profound connections between a function's derivative and its values over an interval, forming the theoretical bedrock for many results in calculus.
* **Vast Applications:** From optimizing manufacturing processes to understanding planetary motion and modeling economic trends, the principles of differential calculus are indispensable in nearly every quantitative field.

Together, limits and derivatives provide a language of precision for describing and predicting the behavior of dynamic systems, making them one of the most powerful and consequential intellectual achievements in history.