---
title: Complex Numbers - Extending Reality, Simplifying Mathematics
description: Journey beyond the number line into the fascinating realm of complex numbers. Discover their algebra, geometry, and the power of forms like polar and Euler. From solving elusive equations to describing intricate transformations, complex numbers are an indispensable tool in mathematics and science.
date: 2022-12-26
draft: false
slug: /pensieve/maths/complex-numbers
tags:
  - Mathematics
  - Algebra
---

The world of numbers is vast and continuously expanding to meet the needs of mathematical inquiry and scientific description. While real numbers serve us well for many everyday measurements and calculations, they fall short when we encounter problems like finding the square root of a negative number. This limitation led to the development of **complex numbers**, a profound extension that not only solves such algebraic quandaries but also provides a powerful framework for understanding various phenomena in geometry, physics, and engineering. This exploration delves into the structure, algebra, and geometric representation of complex numbers, revealing their elegance and utility.

---
## Defining the "Imaginary": The Birth of Complex Numbers

The journey into complex numbers begins with the introduction of a new entity: the **imaginary unit**, denoted by $i$.

* **The Imaginary Unit ($i$):** This is defined as the principal square root of -1:
    $$ i = \sqrt{-1} \quad \text{or} \quad i^2 = -1 $$
    Powers of $i$ follow a cyclical pattern:
    $i^1 = i$
    $i^2 = -1$
    $i^3 = i^2 \cdot i = -i$
    $i^4 = (i^2)^2 = (-1)^2 = 1$
    $i^5 = i^4 \cdot i = i$, and so on.

* **Complex Numbers as Ordered Pairs of Reals:** Fundamentally, a complex number can be defined as an ordered pair of real numbers $(a, b)$. Addition and multiplication for these pairs are defined as:
    $(a, b) + (c, d) = (a+c, b+d)$
    $(a, b) \cdot (c, d) = (ac-bd, ad+bc)$
    Under these definitions, pairs of the form $(a, 0)$ behave exactly like real numbers $a$. The pair $(0, 1)$ corresponds to the imaginary unit $i$.

* **Standard Form ($a+ib$):** Using the imaginary unit $i$, a complex number $z$ is commonly written in the form:
    $$ z = a + ib $$
    where $a$ and $b$ are real numbers.
    * $a = \text{Re}(z)$ is called the **real part** of $z$.
    * $b = \text{Im}(z)$ is called the **imaginary part** of $z$.
    If $b=0$, $z$ is a real number. If $a=0$ and $b \neq 0$, $z$ is a purely imaginary number.

---
## Visualizing the Unseen: The Complex Plane (Argand Diagram)

Complex numbers cannot be represented on a simple number line like real numbers. Instead, they are visualized on a two-dimensional plane called the **complex plane** or **Argand diagram**, named after Jean-Robert Argand.

* The horizontal axis is called the **real axis** (Re).
* The vertical axis is called the **imaginary axis** (Im).
* A complex number $z = a+ib$ is represented as a point $(a,b)$ in this plane, or equivalently, as a vector from the origin $(0,0)$ to the point $(a,b)$.

This geometric representation is crucial for understanding many properties of complex numbers.

---
## The Toolkit: Algebra of Complex Numbers – Basic Operations

Complex numbers follow specific rules for arithmetic operations:

* **Equality:** Two complex numbers $z_1 = a+ib$ and $z_2 = c+id$ are equal if and only if their real parts are equal and their imaginary parts are equal:
    $z_1 = z_2 \iff a=c \text{ and } b=d$.

* **Addition:** $(a+ib) + (c+id) = (a+c) + i(b+d)$.
    (Add real parts, add imaginary parts).

* **Subtraction:** $(a+ib) - (c+id) = (a-c) + i(b-d)$.
    (Subtract real parts, subtract imaginary parts).

* **Multiplication:** $(a+ib)(c+id) = ac + i(ad) + i(bc) + i^2(bd)$
    Since $i^2 = -1$:
    $$ (a+ib)(c+id) = (ac-bd) + i(ad+bc) $$

* **Division:** To divide $\dfrac{a+ib}{c+id}$ (where $c+id \neq 0$), we multiply the numerator and denominator by the conjugate of the denominator ($c-id$):
    $$ \dfrac{a+ib}{c+id} = \dfrac{(a+ib)(c-id)}{(c+id)(c-id)} = \dfrac{(ac+bd) + i(bc-ad)}{c^2 - (id)^2} = \dfrac{(ac+bd) + i(bc-ad)}{c^2 + d^2} $$
    $$ \dfrac{a+ib}{c+id} = \left(\dfrac{ac+bd}{c^2+d^2}\right) + i\left(\dfrac{bc-ad}{c^2+d^2}\right) $$

---
## Size and Direction: Modulus and Argument – The Polar Perspective

In the Argand diagram, a complex number $z=a+ib$ can be described not only by its Cartesian coordinates $(a,b)$ but also by its distance from the origin and the angle its vector makes with the positive real axis.

* **Modulus ($|z|$ or $r$):** The modulus of a complex number $z=a+ib$ is its distance from the origin in the complex plane. It is a non-negative real number.
    $$ |z| = r = \sqrt{a^2+b^2} $$
    **Properties of Modulus:**
    * $|z| \ge 0$, and $|z|=0 \iff z=0$.
    * $|z_1 z_2| = |z_1| |z_2|$.
    * $|z_1 / z_2| = |z_1| / |z_2|$ (for $z_2 \neq 0$).
    * $|z_1 + z_2| \le |z_1| + |z_2|$ (Triangle Inequality).
    * $|z^n| = |z|^n$.

* **Argument ($\arg(z)$ or $\theta$):** The argument (or amplitude) of a non-zero complex number $z=a+ib$ is the angle $\theta$ that the vector representing $z$ makes with the positive real axis, measured counterclockwise.
    It is typically found using $\tan\theta = b/a$, but careful consideration of the quadrant in which $(a,b)$ lies is needed to determine the correct angle.
    The argument is multi-valued; any integer multiple of $2\pi$ can be added to it. The **principal argument**, denoted $\text{Arg}(z)$, is usually taken to be in the interval $(-\pi, \pi]$ or $[0, 2\pi)$.
    **Properties of Argument:**
    * $\arg(z_1 z_2) = \arg(z_1) + \arg(z_2) \pmod{2\pi}$.
    * $\arg(z_1 / z_2) = \arg(z_1) - \arg(z_2) \pmod{2\pi}$.
    * $\arg(z^n) = n \arg(z) \pmod{2\pi}$.

---
## New Outfits: Polar & Euler Forms – Expressing Complexity Differently

Using the modulus $r$ and argument $\theta$, we can express complex numbers in alternative forms that are particularly useful for multiplication, division, powers, and roots.

* **Polar Form:** From trigonometry in the Argand diagram, if $z=a+ib$, then $a = r\cos\theta$ and $b = r\sin\theta$.
    Thus, the polar form of $z$ is:
    $$ z = r(\cos\theta + i\sin\theta) $$

* **Euler's Form (Exponential Form):** Leonhard Euler provided a profound connection between exponential functions and trigonometric functions:
    **Euler's Formula:** $e^{i\theta} = \cos\theta + i\sin\theta$.
    Using this, the polar form can be written more compactly as:
    $$ z = re^{i\theta} $$
    This form is extremely powerful for operations:
    * Multiplication: If $z_1 = r_1 e^{i\theta_1}$ and $z_2 = r_2 e^{i\theta_2}$, then
        $$ z_1 z_2 = (r_1 e^{i\theta_1})(r_2 e^{i\theta_2}) = r_1 r_2 e^{i(\theta_1+\theta_2)} $$
        (Multiply moduli, add arguments).
    * Division:
        $$ \dfrac{z_1}{z_2} = \dfrac{r_1 e^{i\theta_1}}{r_2 e^{i\theta_2}} = \dfrac{r_1}{r_2} e^{i(\theta_1-\theta_2)} $$
        (Divide moduli, subtract arguments).

---
## The Mirror Image: Conjugate of Complex Numbers – Properties and Uses

The **complex conjugate** of a complex number $z=a+ib$, denoted by $\bar{z}$ (or $z^*$), is obtained by changing the sign of its imaginary part:
$$ \bar{z} = a - ib $$

* **Geometric Interpretation:** In the Argand diagram, $\bar{z}$ is the reflection of $z$ across the real axis.
* **Properties of Conjugates:**
    * $z + \bar{z} = (a+ib) + (a-ib) = 2a = 2\text{Re}(z)$.
    * $z - \bar{z} = (a+ib) - (a-ib) = 2ib = 2i\text{Im}(z)$.
    * $z\bar{z} = (a+ib)(a-ib) = a^2 - (ib)^2 = a^2 - i^2b^2 = a^2 + b^2 = |z|^2$. This is very useful!
    * $\overline{z_1 + z_2} = \bar{z_1} + \bar{z_2}$.
    * $\overline{z_1 - z_2} = \bar{z_1} - \bar{z_2}$.
    * $\overline{z_1 z_2} = \bar{z_1} \bar{z_2}$.
    * $\overline{\left(\dfrac{z_1}{z_2}\right)} = \dfrac{\bar{z_1}}{\bar{z_2}}$ (for $z_2 \neq 0$).
    * $\overline{z^n} = (\bar{z})^n$.
    * If $z$ is real, then $z = \bar{z}$.
* **Uses:** Conjugates are essential for dividing complex numbers (as seen earlier), finding the modulus, and simplifying expressions.

---
## Power Up!: De Moivre's Theorem – Powers and Roots Made Easy(er)

Abraham de Moivre provided a powerful theorem for finding powers and roots of complex numbers when expressed in polar (or Euler) form.

**De Moivre's Theorem:** For any integer $n$ and any real number $\theta$:
$$ (\cos\theta + i\sin\theta)^n = \cos(n\theta) + i\sin(n\theta) $$
In Euler's form, this is simply $(e^{i\theta})^n = e^{in\theta}$, which is a standard property of exponentials.

* **Powers of Complex Numbers:** If $z = r(\cos\theta + i\sin\theta) = re^{i\theta}$, then raising $z$ to the power $n$ (where $n$ is an integer):
    $$ z^n = [r(\cos\theta + i\sin\theta)]^n = r^n (\cos\theta + i\sin\theta)^n $$
    By De Moivre's theorem:
    $$ z^n = r^n (\cos(n\theta) + i\sin(n\theta)) = r^n e^{in\theta} $$
    (Raise the modulus to the power $n$, multiply the argument by $n$).

* **Roots of Complex Numbers:** Finding the $n^{th}$ roots of a complex number $z = r(\cos\theta + i\sin\theta)$. We are looking for complex numbers $w$ such that $w^n = z$. Let $w = s(\cos\psi + i\sin\psi)$.
    Then $w^n = s^n(\cos(n\psi) + i\sin(n\psi))$.
    So, $s^n = r \implies s = r^{1/n}$ (the real positive $n^{th}$ root of $r$).
    And $n\psi = \theta + 2k\pi$ for any integer $k$ (because $\cos$ and $\sin$ are $2\pi$-periodic).
    Thus, $\psi_k = \dfrac{\theta + 2k\pi}{n}$.
    The $n$ distinct $n^{th}$ roots of $z$ are given by:
    $$ w_k = r^{1/n} \left[\cos\left(\dfrac{\theta + 2k\pi}{n}\right) + i\sin\left(\dfrac{\theta + 2k\pi}{n}\right)\right] $$
    for $k = 0, 1, 2, \ldots, n-1$. These $n$ roots are equally spaced around a circle of radius $r^{1/n}$ in the complex plane.

---
## Roots of Unity: The Special Circle Dance

The **$n^{th}$ roots of unity** are the complex numbers $z$ that satisfy the equation $z^n = 1$.
Since $1$ can be written in polar form as $1(\cos(0) + i\sin(0))$, its argument is $\theta=0$ (or $2m\pi$).
Using the formula for roots, the $n^{th}$ roots of unity are:
$$ \omega_k = 1^{1/n} \left[\cos\left(\dfrac{0 + 2k\pi}{n}\right) + i\sin\left(\dfrac{0 + 2k\pi}{n}\right)\right] $$
$$ \omega_k = \cos\left(\dfrac{2k\pi}{n}\right) + i\sin\left(\dfrac{2k\pi}{n}\right) = e^{i(2k\pi/n)} $$
for $k = 0, 1, 2, \ldots, n-1$.
* Geometrically, these $n$ roots are the vertices of a regular $n$-sided polygon inscribed in the unit circle $|z|=1$ in the complex plane, with one vertex at $(1,0)$.
* The sum of the $n^{th}$ roots of unity is 0 (for $n>1$).
* The product of the $n^{th}$ roots of unity is $(-1)^{n-1}$.

**Cube Roots of Unity ($n=3$):** Solutions to $z^3=1$.
The roots are for $k=0,1,2$:
* $k=0: \omega_0 = \cos(0) + i\sin(0) = 1$.
* $k=1: \omega_1 = \cos(2\pi/3) + i\sin(2\pi/3) = -\dfrac{1}{2} + i\dfrac{\sqrt{3}}{2}$. This is often denoted by $\omega$.
* $k=2: \omega_2 = \cos(4\pi/3) + i\sin(4\pi/3) = -\dfrac{1}{2} - i\dfrac{\sqrt{3}}{2}$. This is $\omega^2$ (and also $\bar{\omega}$).
**Properties of Cube Roots of Unity:**
1.  $1, \omega, \omega^2$ are the three cube roots.
2.  $\omega^3 = 1$.
3.  $1 + \omega + \omega^2 = 0$.
4.  $\omega^2 = \bar{\omega}$.

---
## Geometry in the Complex Plane: Shapes and Transformations

The Argand diagram allows us to explore geometry using complex numbers.

* **Standard Loci (Paths of points $z$ satisfying certain conditions):**
    * $|z - z_0| = R$: Represents a **circle** with center $z_0$ and radius $R$.
    * $|z - z_1| = |z - z_2|$: Represents the **perpendicular bisector** of the line segment joining the points $z_1$ and $z_2$.
    * $|z - z_1| + |z - z_2| = 2a$ (where $2a > |z_1 - z_2|$): Represents an **ellipse** with foci at $z_1$ and $z_2$, and major axis length $2a$.
    * $||z - z_1| - |z - z_2|| = 2a$ (where $2a < |z_1 - z_2|$): Represents a **hyperbola** with foci at $z_1$ and $z_2$, and transverse axis length $2a$.
    * $\arg\left(\dfrac{z-z_1}{z-z_2}\right) = \alpha$: Represents an **arc of a circle** passing through $z_1$ and $z_2$, where the angle subtended by the segment $z_1z_2$ at any point $z$ on the arc is $\alpha$. If $\alpha = \pm \pi/2$, it's a circle with $z_1z_2$ as diameter.

* **Concept of Rotation:**
    Multiplication of a complex number $z$ by $e^{i\phi}$ (which is $\cos\phi + i\sin\phi$) rotates the vector representing $z$ by an angle $\phi$ counterclockwise about the origin without changing its modulus.
    If $z_1, z_2, z_3$ are three complex numbers, the complex number $\dfrac{z_3-z_1}{z_2-z_1}$ represents a vector. Its argument, $\arg\left(\dfrac{z_3-z_1}{z_2-z_1}\right)$, gives the angle $\angle z_2 z_1 z_3$ (the angle between vector $z_1z_2$ and vector $z_1z_3$).

* **Section Formula:** The complex number $z$ representing a point that divides the line segment joining points $z_1$ and $z_2$ in the ratio $m:n$ is:
    * Internal division: $z = \dfrac{mz_2 + nz_1}{m+n}$
    * External division: $z = \dfrac{mz_2 - nz_1}{m-n}$
    The midpoint is $(z_1+z_2)/2$.

* **Equation of Lines:**
    * Parametric form (line passing through $z_1$ and $z_2$): $z = z_1 + t(z_2-z_1)$, where $t \in \mathbb{R}$. Or $z = (1-t)z_1 + tz_2$.
    * Non-parametric (General form): A line can be represented by the equation $a\bar{z} + \bar{a}z + b = 0$, where $b$ is a real constant and $a$ is a non-zero complex constant.
    * Condition for collinearity of $z_1, z_2, z_3$: The complex number $\dfrac{z_3-z_1}{z_2-z_1}$ must be purely real (its imaginary part is zero).
        Equivalently, $z_1(\bar{z_2}-\bar{z_3}) + z_2(\bar{z_3}-\bar{z_1}) + z_3(\bar{z_1}-\bar{z_2}) = 0$. Or using determinants.

* **Equation of Circles:**
    * $|z-z_0|=R$ (Center $z_0$, radius $R$).
    * Squaring: $(z-z_0)(\overline{z-z_0}) = R^2 \implies (z-z_0)(\bar{z}-\bar{z_0}) = R^2$.
        $z\bar{z} - z\bar{z_0} - \bar{z}z_0 + z_0\bar{z_0} = R^2$.
        This is of the form $z\bar{z} + \alpha\bar{z} + \bar{\alpha}z + k = 0$, where $\alpha = -z_0$ (a complex constant) and $k = |z_0|^2 - R^2$ (a real constant).
        The general form is $z\bar{z} + a\bar{z} + \bar{a}z + b = 0$, where $b$ is real. The center is $-a$ and radius is $\sqrt{|a|^2-b}$ (provided $|a|^2-b > 0$).

* **Equation of Conics (Locus Definitions):**
    * **Parabola:** $|z-S| = |z-L|$, where $S$ is the focus (a complex number) and $L$ represents the directrix (a line). The equation of a line can be written as $\text{Re}(z \cdot e^{-i\alpha}) = p$. More simply, if directrix is $x=-a$ and focus $S=(a,0)$, then $|z-a| = |\text{Re}(z)+a|$.
    * **Ellipse:** $|z-S_1| + |z-S_2| = 2a$ (where $2a > |S_1-S_2|$), $S_1, S_2$ are foci.
    * **Hyperbola:** $||z-S_1| - |z-S_2|| = 2a$ (where $2a < |S_1-S_2|$), $S_1, S_2$ are foci.
    These locus definitions directly translate into equations involving moduli of complex numbers.

---
## Key Takeaways: The Power and Beauty of Complex Numbers

The introduction of complex numbers marked a significant expansion of our mathematical landscape, providing tools and insights that extend far beyond simply solving $\sqrt{-1}$.

* **Fundamental Structure:** Complex numbers $z=a+ib$ can be viewed as ordered pairs of reals, represented geometrically on the Argand diagram, and expressed in Cartesian, polar ($r(\cos\theta+i\sin\theta)$), or Euler ($re^{i\theta}$) forms.
* **Rich Algebra:** They possess a complete algebra (addition, subtraction, multiplication, division) with familiar properties, augmented by the concept of the complex conjugate ($\bar{z}=a-ib$) which is crucial for operations like division and finding the modulus ($|z|^2 = z\bar{z}$).
* **Powers and Roots Simplified:** De Moivre's Theorem ($(\cos\theta+i\sin\theta)^n = \cos(n\theta)+i\sin(n\theta)$) provides an elegant way to calculate powers and find the $n$ distinct $n^{th}$ roots of any complex number, including the geometrically significant roots of unity.
* **Geometric Insight:** The complex plane allows for powerful geometric interpretations. Modulus represents distance, argument represents angle. Operations like multiplication by $e^{i\phi}$ correspond to rotation. Equations of lines, circles, and other conic sections can be concisely expressed using complex numbers.
* **Essential Tool:** Complex numbers are indispensable in many fields of mathematics (like number theory, fractals), physics (quantum mechanics, electromagnetism, fluid dynamics), and engineering (signal processing, electrical engineering).

The development of complex numbers showcases mathematics' ability to create new structures that not only solve existing problems but also open up vast new avenues of exploration and application.