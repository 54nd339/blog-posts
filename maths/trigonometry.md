---
title: Trigonometry - The Science of Angles, Waves, and Periodic Phenomena
description: From the fundamental properties of triangles to the intricate dance of trigonometric functions and their inverses, this post explores the core concepts of trigonometry. Uncover the relationships between angles and sides, master trigonometric identities, and learn to solve equations and inequalities that model the periodic nature of our world.
date: 2022-12-29
draft: false
slug: /pensieve/maths/trigonometry
tags:
  - Mathematics
  - Trigonometry
---

Trigonometry, at its heart, is the study of relationships between the angles and sides of triangles. However, its reach extends far beyond simple geometric figures, providing an indispensable language for describing periodic phenomena, waves, oscillations, and a multitude of applications in fields ranging from physics and engineering to astronomy, navigation, and even music. This exploration delves into the foundational elements of trigonometry, from basic triangle properties and angle measurement to the rich world of trigonometric functions, identities, equations, and their inverses.

---
## The Foundation: Triangles – Shapes of Stability and Measurement

While trigonometry has evolved to describe more abstract concepts, its origins lie firmly in the study of triangles.

### Properties of Triangles
A triangle is a fundamental geometric shape with three sides and three angles.
* **Types of Triangles:**
    * Based on sides: **Scalene** (all sides different), **Isosceles** (two sides equal), **Equilateral** (all sides equal).
    * Based on angles: **Acute-angled** (all angles < 90°), **Obtuse-angled** (one angle > 90°), **Right-angled** (one angle = 90°).
* **Congruence of Triangles:** Two triangles are congruent if they have the same size and shape. Common postulates for congruence include SSS (Side-Side-Side), SAS (Side-Angle-Side), ASA (Angle-Side-Angle), AAS (Angle-Angle-Side), and RHS (Right-angle-Hypotenuse-Side for right triangles).
* **Similarity of Triangles:** Two triangles are similar if they have the same shape but not necessarily the same size (corresponding angles are equal, and corresponding sides are in proportion). Criteria include AA (Angle-Angle), SSS (Side-Side-Side proportionality), and SAS (Side-Angle-Side proportionality).

The right-angled triangle, in particular, forms the initial basis for defining trigonometric ratios.

---
## Measuring Turn: Angles in Trigonometry – Degrees, Radians, and Signs

An **angle** is a measure of rotation of a given ray about its initial point. The original ray is called the initial side and the final position of the ray after rotation is called the terminal side of the angle.

### Units of Angle Measurement
1.  **Degrees (°):** A full circle is divided into 360 degrees. A degree is further subdivided into 60 minutes ('), and a minute into 60 seconds (''). So, $1^\circ = 60'$, $1' = 60''$.
2.  **Radians (rad):** The radian is the standard unit of angular measure in many areas of mathematics. One radian is the angle subtended at the center of a circle by an arc that is equal in length to the radius of the circle.
    The circumference of a circle is $2\pi r$. Thus, a full circle corresponds to an angle of $2\pi r / r = 2\pi$ radians.

### Relationship between Degrees and Radians
Since a full circle is $360^\circ$ or $2\pi$ radians:
$$ \pi \text{ radians} = 180^\circ $$
From this, we can derive conversion factors:
* $1 \text{ radian} = \dfrac{180^\circ}{\pi} \approx 57.296^\circ$
* $1^\circ = \dfrac{\pi}{180} \text{ radians} \approx 0.01745 \text{ rad}$

### Signs of Angles and Quadrants
In a Cartesian coordinate system:
* An angle is **positive** if the rotation from the initial side (typically the positive x-axis) to the terminal side is counter-clockwise.
* An angle is **negative** if the rotation is clockwise.
* The coordinate plane is divided into four **quadrants**. The quadrant in which the terminal side of an angle lies determines the signs of its trigonometric functions.
* **Coterminal angles** are angles in standard position that have the same terminal side. They differ by integer multiples of $360^\circ$ or $2\pi$ radians.

---
## The Core Six: Trigonometric Functions (Ratios) – Relating Angles and Sides

Trigonometric functions are functions of an angle. They are fundamental in studying triangles and modeling periodic phenomena.

### Definitions using a Right-Angled Triangle
For an acute angle $\theta$ in a right-angled triangle:
* **Sine ($\sin\theta$):** Ratio of the length of the side opposite angle $\theta$ to the length of the hypotenuse.
    $$ \sin\theta = \dfrac{\text{Opposite}}{\text{Hypotenuse}} $$
* **Cosine ($\cos\theta$):** Ratio of the length of the side adjacent to angle $\theta$ to the length of the hypotenuse.
    $$ \cos\theta = \dfrac{\text{Adjacent}}{\text{Hypotenuse}} $$
* **Tangent ($\tan\theta$):** Ratio of the length of the side opposite angle $\theta$ to the length of the side adjacent to angle $\theta$.
    $$ \tan\theta = \dfrac{\text{Opposite}}{\text{Adjacent}} = \dfrac{\sin\theta}{\cos\theta} $$

### Reciprocal Functions
* **Cosecant ($\csc\theta$):** $\csc\theta = \dfrac{1}{\sin\theta} = \dfrac{\text{Hypotenuse}}{\text{Opposite}}$
* **Secant ($\sec\theta$):** $\sec\theta = \dfrac{1}{\cos\theta} = \dfrac{\text{Hypotenuse}}{\text{Adjacent}}$
* **Cotangent ($\cot\theta$):** $\cot\theta = \dfrac{1}{\tan\theta} = \dfrac{\text{Adjacent}}{\text{Opposite}} = \dfrac{\cos\theta}{\sin\theta}$

### Definitions using the Unit Circle
For any angle $\theta$ in standard position, let $P(x,y)$ be the point where the terminal side of the angle intersects the unit circle ($x^2+y^2=1$). Then:
* $\cos\theta = x$
* $\sin\theta = y$
* $\tan\theta = y/x$ (for $x \neq 0$)
* $\sec\theta = 1/x$ (for $x \neq 0$)
* $\csc\theta = 1/y$ (for $y \neq 0$)
* $\cot\theta = x/y$ (for $y \neq 0$)

This definition extends trigonometric functions to all real numbers.

### Signs of Trigonometric Functions in Quadrants
* **Quadrant I ($0 < \theta < \pi/2$):** All functions are positive. (All)
* **Quadrant II ($\pi/2 < \theta < \pi$):** Sine and Cosecant are positive. (Silver)
* **Quadrant III ($\pi < \theta < 3\pi/2$):** Tangent and Cotangent are positive. (Tea)
* **Quadrant IV ($3\pi/2 < \theta < 2\pi$):** Cosine and Secant are positive. (Cups)
(Mnemonic: "All Silver Tea Cups" or "All Students Take Calculus")

### Domain, Range, and Periodicity
* $\sin\theta, \cos\theta$: Domain $\mathbb{R}$, Range $[-1, 1]$, Period $2\pi$.
* $\tan\theta, \sec\theta$: Domain $\mathbb{R} - \{(n+1/2)\pi \mid n \in \mathbb{Z}\}$, Range for $\tan\theta$ is $\mathbb{R}$, Range for $\sec\theta$ is $(-\infty, -1] \cup [1, \infty)$. Period for $\tan\theta$ is $\pi$, for $\sec\theta$ is $2\pi$.
* $\cot\theta, \csc\theta$: Domain $\mathbb{R} - \{n\pi \mid n \in \mathbb{Z}\}$, Range for $\cot\theta$ is $\mathbb{R}$, Range for $\csc\theta$ is $(-\infty, -1] \cup [1, \infty)$. Period for $\cot\theta$ is $\pi$, for $\csc\theta$ is $2\pi$.

---
## Special Angles & Relationships: Mastering Trigonometric Ratios

### Trigonometric Ratios of Standard Angles
Knowledge of ratios for certain common angles is essential:

| Angle ($\theta$) | $0^\circ (0)$ | $30^\circ (\pi/6)$ | $45^\circ (\pi/4)$ | $60^\circ (\pi/3)$ | $90^\circ (\pi/2)$ |
| :--------------- | :---------- | :----------------- | :----------------- | :----------------- | :--------------- |
| $\sin\theta$     | $0$         | $1/2$              | $1/\sqrt{2}$       | $\sqrt{3}/2$       | $1$              |
| $\cos\theta$     | $1$         | $\sqrt{3}/2$       | $1/\sqrt{2}$       | $1/2$              | $0$              |
| $\tan\theta$     | $0$         | $1/\sqrt{3}$       | $1$                | $\sqrt{3}$         | Undefined        |

### Trigonometric Ratios of Complementary Angles ($\pi/2 - \theta$)
Complementary angles add up to $90^\circ$ or $\pi/2$ radians.
* $\sin(\pi/2 - \theta) = \cos\theta$
* $\cos(\pi/2 - \theta) = \sin\theta$
* $\tan(\pi/2 - \theta) = \cot\theta$
* $\csc(\pi/2 - \theta) = \sec\theta$
* $\sec(\pi/2 - \theta) = \csc\theta$
* $\cot(\pi/2 - \theta) = \tan\theta$

### Trigonometric Ratios of Supplementary Angles and Other Related Angles
These identities help find ratios for angles outside the first quadrant using the ASTC rule (All, Sin, Tan, Cos positive in Quadrants I, II, III, IV respectively).
* $\sin(\pi - \theta) = \sin\theta$, $\cos(\pi - \theta) = -\cos\theta$, $\tan(\pi - \theta) = -\tan\theta$
* $\sin(\pi + \theta) = -\sin\theta$, $\cos(\pi + \theta) = -\cos\theta$, $\tan(\pi + \theta) = \tan\theta$
* $\sin(2\pi - \theta) = \sin(-\theta) = -\sin\theta$
* $\cos(2\pi - \theta) = \cos(-\theta) = \cos\theta$
* $\tan(2\pi - \theta) = \tan(-\theta) = -\tan\theta$

### Trigonometric Ratios of Compound Angles (Sum and Difference Identities)
These are crucial for expanding or simplifying trigonometric expressions involving sums or differences of angles.
* $\sin(A \pm B) = \sin A \cos B \pm \cos A \sin B$
* $\cos(A \pm B) = \cos A \cos B \mp \sin A \sin B$
* $\tan(A \pm B) = \dfrac{\tan A \pm \tan B}{1 \mp \tan A \tan B}$

---
## Shape Shifting: Transformation of Trigonometric Functions (Identities)

Trigonometric identities are equations involving trigonometric functions that are true for all values of the variables for which both sides of the equation are defined. They are essential tools for simplifying expressions and solving equations.

### Pythagorean Identities
Derived from the Pythagorean theorem in the unit circle ($x^2+y^2=1$):
* $\sin^2\theta + \cos^2\theta = 1$
* $1 + \tan^2\theta = \sec^2\theta$ (Divide first by $\cos^2\theta$)
* $1 + \cot^2\theta = \csc^2\theta$ (Divide first by $\sin^2\theta$)

### Product-to-Sum Identities
These transform products of sines and cosines into sums or differences.
* $2\sin A \cos B = \sin(A+B) + \sin(A-B)$
* $2\cos A \sin B = \sin(A+B) - \sin(A-B)$
* $2\cos A \cos B = \cos(A+B) + \cos(A-B)$
* $2\sin A \sin B = \cos(A-B) - \cos(A+B)$

### Sum-to-Product Identities
These transform sums or differences of sines and cosines into products.
* $\sin C + \sin D = 2\sin\left(\dfrac{C+D}{2}\right)\cos\left(\dfrac{C-D}{2}\right)$
* $\sin C - \sin D = 2\cos\left(\dfrac{C+D}{2}\right)\sin\left(\dfrac{C-D}{2}\right)$
* $\cos C + \cos D = 2\cos\left(\dfrac{C+D}{2}\right)\cos\left(\dfrac{C-D}{2}\right)$
* $\cos C - \cos D = -2\sin\left(\dfrac{C+D}{2}\right)\sin\left(\dfrac{C-D}{2}\right) = 2\sin\left(\dfrac{C+D}{2}\right)\sin\left(\dfrac{D-C}{2}\right)$

### Double Angle Identities
These are special cases of the compound angle identities where $A=B=\theta$.
* $\sin(2\theta) = 2\sin\theta\cos\theta$
* $\cos(2\theta) = \cos^2\theta - \sin^2\theta = 2\cos^2\theta - 1 = 1 - 2\sin^2\theta$
* $\tan(2\theta) = \dfrac{2\tan\theta}{1-\tan^2\theta}$

### Half Angle Identities
These are derived from the $\cos(2\theta)$ identities (by replacing $2\theta$ with $\theta$, so $\theta$ becomes $\theta/2$).
* $\sin^2(\theta/2) = \dfrac{1-\cos\theta}{2} \implies \sin(\theta/2) = \pm\sqrt{\dfrac{1-\cos\theta}{2}}$
* $\cos^2(\theta/2) = \dfrac{1+\cos\theta}{2} \implies \cos(\theta/2) = \pm\sqrt{\dfrac{1+\cos\theta}{2}}$
* $\tan^2(\theta/2) = \dfrac{1-\cos\theta}{1+\cos\theta} \implies \tan(\theta/2) = \pm\sqrt{\dfrac{1-\cos\theta}{1+\cos\theta}} = \dfrac{\sin\theta}{1+\cos\theta} = \dfrac{1-\cos\theta}{\sin\theta}$
The sign depends on the quadrant in which $\theta/2$ lies.

---
## Solving the Angle Puzzles: Trigonometric Equations

A **trigonometric equation** is an equation that involves trigonometric functions of an unknown angle.
* **Roots (Solutions):** The values of the unknown angle that satisfy the equation.
* **Principal Solutions:** The solutions lying in a specific interval, typically $[0, 2\pi)$ or $(-\pi, \pi]$ for sine/cosine/tangent, or within the principal value range of the corresponding inverse function.
* **General Solutions:** An expression that represents all possible solutions, taking into account the periodicity of the trigonometric functions. It usually involves an integer $n \in \mathbb{Z}$.

### General Solutions of Some Standard Trigonometric Equations
Let $\alpha$ be a principal value satisfying the equation.
* If $\sin\theta = \sin\alpha \implies \theta = n\pi + (-1)^n \alpha$, where $n \in \mathbb{Z}$.
* If $\cos\theta = \cos\alpha \implies \theta = 2n\pi \pm \alpha$, where $n \in \mathbb{Z}$.
* If $\tan\theta = \tan\alpha \implies \theta = n\pi + \alpha$, where $n \in \mathbb{Z}$.

Specific cases:
* $\sin\theta = 0 \implies \theta = n\pi$
* $\cos\theta = 0 \implies \theta = (2n+1)\pi/2 = n\pi + \pi/2$
* $\tan\theta = 0 \implies \theta = n\pi$
* $\sin\theta = 1 \implies \theta = 2n\pi + \pi/2 = (4n+1)\pi/2$
* $\cos\theta = 1 \implies \theta = 2n\pi$
* $\sin\theta = -1 \implies \theta = 2n\pi - \pi/2 = (4n-1)\pi/2$
* $\cos\theta = -1 \implies \theta = (2n+1)\pi$

Solving more complex trigonometric equations often involves using identities to simplify the equation into one of these basic forms or into a polynomial in a trigonometric function.

---
## Comparing Angles: Trigonometric Inequalities

**Trigonometric inequalities** are inequalities involving trigonometric functions of an unknown angle. Solving them typically involves:
1.  Utilizing the graphs of trigonometric functions to visualize where the function values are greater than, less than, or equal to a certain value.
2.  Using the unit circle to identify the ranges of angles that satisfy the inequality within one period.
3.  Converting the trigonometric inequality into an algebraic inequality by substitution (e.g., let $t = \sin x$) and solving for $t$, then converting back to find $x$.
4.  Considering the periodicity of the functions to write down the general solution for the angle.
5.  Paying close attention to the domain over which the solution is required.

For example, to solve $\sin x > 1/2$:
1.  Find angles where $\sin x = 1/2$. In $[0, 2\pi)$, these are $x = \pi/6$ and $x = 5\pi/6$.
2.  From the graph of $\sin x$ or the unit circle, $\sin x > 1/2$ when $\pi/6 < x < 5\pi/6$.
3.  The general solution would be $2n\pi + \pi/6 < x < 2n\pi + 5\pi/6$, for $n \in \mathbb{Z}$.

---
## Undoing the Twist: Inverse Trigonometric Functions – Finding the Angle

**Inverse trigonometric functions** are used to find an angle when a trigonometric ratio is known. They "undo" the trigonometric functions.
* $y = \arcsin x$ (or $\sin^{-1} x$) means $\sin y = x$.
* $y = \arccos x$ (or $\cos^{-1} x$) means $\cos y = x$.
* $y = \arctan x$ (or $\tan^{-1} x$) means $\tan y = x$.
And similarly for $\text{arccsc } x, \text{arcsec } x, \text{arccot } x$.

### Domain, Range, and Graphs
Since trigonometric functions are periodic, they are not one-to-one over their entire domain. To define their inverse functions, their domains must be restricted to an interval where they *are* one-to-one. This leads to principal value ranges for the inverse functions.

* **$y = \arcsin x$**
    * Domain: $[-1, 1]$
    * Range (Principal Value): $[-\pi/2, \pi/2]$
* **$y = \arccos x$**
    * Domain: $[-1, 1]$
    * Range (Principal Value): $[0, \pi]$
* **$y = \arctan x$**
    * Domain: $(-\infty, \infty)$ or $\mathbb{R}$
    * Range (Principal Value): $(-\pi/2, \pi/2)$
* **$y = \text{arccsc } x$**
    * Domain: $(-\infty, -1] \cup [1, \infty)$
    * Range (Principal Value): $[-\pi/2, 0) \cup (0, \pi/2]$
* **$y = \text{arcsec } x$**
    * Domain: $(-\infty, -1] \cup [1, \infty)$
    * Range (Principal Value): $[0, \pi/2) \cup (\pi/2, \pi]$
* **$y = \text{arccot } x$**
    * Domain: $(-\infty, \infty)$ or $\mathbb{R}$
    * Range (Principal Value): $(0, \pi)$

The graphs of inverse trigonometric functions are reflections of the graphs of the restricted trigonometric functions across the line $y=x$. Many identities exist for inverse trigonometric functions, such as $\arcsin(-x) = -\arcsin x$, $\arccos(-x) = \pi - \arccos x$, and compositions like $\sin(\arcsin x) = x$ (for $x$ in domain of arcsin) or $\arcsin(\sin x) = x$ (only if $x$ is in the principal value range of arcsin).

---
## Key Takeaways: The Enduring Power of Trigonometry

Trigonometry provides a powerful mathematical framework for understanding angles, triangles, and periodic phenomena, with applications extending across numerous scientific and engineering disciplines.

* **Foundations in Angles and Triangles:** The core of trigonometry lies in relating angles to side lengths, with degrees and radians as primary units of angular measurement.
* **Trigonometric Functions:** Sine, cosine, tangent, and their reciprocals define fundamental ratios that extend from right triangles to the unit circle, describing periodic behavior.
* **Identities are Tools:** A vast array of trigonometric identities (Pythagorean, sum/difference, double/half angle, product-to-sum, sum-to-product) are essential for simplifying expressions, solving equations, and proving other mathematical statements.
* **Solving for Angles:** Trigonometric equations and inequalities can be solved using these identities, graphical methods, and an understanding of the functions' periodic nature, leading to principal and general solutions.
* **Inverse Functions for Angles:** Inverse trigonometric functions allow us to determine angles from known trigonometric ratios, with carefully defined principal value ranges ensuring they are true functions.

From navigating ships by the stars to analyzing electrical circuits and modeling sound waves, the principles of trigonometry are woven into the fabric of our quantitative understanding of the world. Its elegant relationships continue to provide insight and utility in ever-expanding domains.