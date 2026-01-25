---
title: Differential Equations - The Mathematical Language of Change
description: Explore the world of differential equations, the powerful mathematical tools used to model dynamic systems in science and engineering. This post provides a comprehensive overview of ordinary differential equations (ODEs), methods for solving first and second-order equations, the analysis of systems, and their wide-ranging applications.
date: 2023-01-11
draft: false
slug: /pensieve/maths/differential-equations
tags:
  - Mathematics
  - Calculus
---

The universe is in a perpetual state of flux. From the motion of planets and the flow of heat to the growth of populations and the oscillations in an electric circuit, change is a fundamental constant. Differential equations provide the mathematical language to describe these dynamic processes. They are equations that relate a function with its derivatives, capturing the intricate relationship between a quantity and its instantaneous rate of change. This exploration delves into the foundational concepts of ordinary differential equations (ODEs), outlining the methods used to solve them and their profound importance in modeling the world around us.

---
## Introduction to Differential Equations

### What is a Differential Equation?
An **ordinary differential equation (ODE)** is an equation containing a function of one independent variable and its derivatives.
* **Order:** The order of a differential equation is the order of the highest derivative present in the equation. For example, $y'' + 2y' + y = 0$ is a second-order ODE.
* **Degree:** The degree of a differential equation is the power of the highest-order derivative, after the equation has been cleared of radicals and fractions in the derivatives. For example, $(y'')^2 + y' = 0$ is of degree two.

### Formation and Solution
* **Formation:** A differential equation can be formed by eliminating arbitrary constants from a family of curves. For a family with $n$ arbitrary constants, we typically need to differentiate $n$ times to form an $n^{th}$-order ODE.
* **Solution:** A solution to a differential equation is a function that satisfies the equation.
    * **General Solution:** Contains arbitrary constants and represents the entire family of functions that satisfy the ODE.
    * **Particular Solution:** A solution obtained by assigning specific values to the arbitrary constants, often determined by initial or boundary conditions.

### Classifying Differential Equations
* **Linear vs. Nonlinear ODEs:** An ODE is **linear** if the dependent variable and all its derivatives appear to the first degree, and there are no products of the dependent variable and/or its derivatives. Otherwise, it is **nonlinear**.
* **Homogeneous vs. Non-homogeneous:**
    * A linear ODE is **homogeneous** if the term independent of the dependent variable and its derivatives is zero. E.g., $ay'' + by' + cy = 0$.
    * It is **non-homogeneous** if this term is non-zero. E.g., $ay'' + by' + cy = g(x)$.

### Posing the Problem: IVPs and BVPs
* **Initial Value Problems (IVPs):** An ODE where all conditions (e.g., the value of the function and its derivatives) are specified at a single point (the "initial" point).
* **Boundary Value Problems (BVPs):** An ODE where conditions are specified at more than one point, typically at the boundaries of an interval.

### Existence and Uniqueness Theorems
These theorems provide conditions under which a solution to a differential equation is guaranteed to exist and be unique. For an IVP of the form $y' = f(x,y)$, $y(x_0)=y_0$, the **Picard-Lindelöf theorem** states that if $f$ and its partial derivative with respect to $y$ are continuous in a region containing the point $(x_0, y_0)$, then a unique solution exists in some interval around $x_0$.

---
## Solving First-Order ODEs

### Geometric Meaning and Numerical Methods
* **Geometric Meaning of $y' = f(x,y)$:** The equation assigns a slope, $f(x,y)$, to every point $(x,y)$ in a region of the xy-plane.
* **Direction Fields (Slope Fields):** By drawing short line segments with the corresponding slope at many points, we can create a direction field that visually represents the general behavior of the solution curves.
* **Euler's Method:** A simple numerical method to approximate solutions to an IVP. Starting from $(x_n, y_n)$, we approximate the next point $(x_{n+1}, y_{n+1})$ using the tangent line:
    $$ y_{n+1} = y_n + h \cdot f(x_n, y_n) $$
    where $h = x_{n+1} - x_n$ is the step size.

### Analytical Solution Techniques for First-Order ODEs
* **Separable Equations:**
    An equation is separable if it can be written in the form $g(y)dy = f(x)dx$.
    **Solution:** Integrate both sides: $\int g(y)dy = \int f(x)dx + C$.
    **Common Equations:**
    * $xdy + ydx = d(xy)$
    * $\dfrac{xdy-ydx}{x^2} = d(\dfrac{y}{x})$
    * $\dfrac{xdy+ydx}{xy} = d(\ln xy)$
    * $\dfrac{xdy-ydx}{xy} = d(\ln \dfrac{x}{y})$
    * $\dfrac{xdy-ydx}{x^2 + y^2} = d(\tan^{-1} \dfrac{y}{x})$
    * $\dfrac{xdx+ydy}{\sqrt{x^2 + y^2}} = d(\sqrt{x^2 + y^2})$

* **Exact Differential Equations:**
    An equation of the form $M(x,y)dx + N(x,y)dy = 0$ is **exact** if $\dfrac{\partial M}{\partial y} = \dfrac{\partial N}{\partial x}$.
    **Solution:** If the equation is exact, there exists a function $u(x,y)$ such that $du = Mdx + Ndy$. The solution is given implicitly by $u(x,y)=C$. To find $u$, integrate $M$ with respect to $x$ (treating $y$ as constant) and add a function of $y$, then differentiate with respect to $y$ and equate to $N$ to find the function of $y$.

* **Integrating Factors:** If a non-exact equation can be made exact by multiplying it by a function $\mu(x,y)$, then $\mu$ is an integrating factor. For a **linear first-order ODE** of the form:
    $$ y' + p(x)y = r(x) $$
    The integrating factor is $I(x) = e^{\int p(x)dx}$. And the solution is given by:
    $$ y(x) = \dfrac{1}{I(x)}\left(\int I(x)r(x)dx + C\right) $$
    where $C$ is a constant determined by initial conditions.
    **Derivation:** Multiply the equation by an unknown $I(x)$: $I(x)y' + I(x)p(x)y = I(x)r(x)$. We want the left side to be the result of the product rule, i.e., $(I(x)y)'$. For this to be true, we need $(I(x)y)' = I'(x)y + I(x)y'$. Comparing with the equation, we need $I'(x) = I(x)p(x)$. This is a separable DE for $I(x)$: $\dfrac{dI}{I} = p(x)dx$. Integrating gives $\ln|I| = \int p(x)dx$, so $I(x) = e^{\int p(x)dx}$.

* **Bernoulli's Equation:**
    An equation of the form:
    $$ y' + p(x)y = r(x)y^n $$
    where $n \neq 0, 1$.
    **Method:** Divide by $y^n$: $y^{-n}y' + p(x)y^{1-n} = r(x)$.
    Make the substitution $u = y^{1-n}$. Then $u' = (1-n)y^{-n}y'$. The equation transforms into a linear ODE in $u$:
    $\dfrac{1}{1-n}u' + p(x)u = r(x)$, which can be solved using an integrating factor.

### Existence and Uniqueness of Solutions for Initial Value Problems
* **Existence Theorem:** If $f(x,y)$ is continuous in a rectangle $R$ containing the point $(x_0, y_0)$, then there exists at least one solution for all $x$ in the interval around $x_0$.
* **Uniqueness Theorem:** If $f(x,y)$ and $\dfrac{\partial f}{\partial y}$ are continuous in $R$, then the solution is unique in the interval around $x_0$.

---
## Solving Second and Higher-Order Linear ODEs

### Second-Order Homogeneous Linear ODEs with Constant Coefficients
These are of the form: $ay'' + by' + cy = 0$, where $a,b,c$ are constants.
* **Method:** Assume an exponential solution $y = e^{\lambda x}$. Substituting this into the ODE yields the **characteristic equation**:
    $$ a\lambda^2 + b\lambda + c = 0 $$
* **Solutions based on roots of the characteristic equation:**
    1.  **Two Distinct Real Roots ($\lambda_1, \lambda_2$):** The general solution is $y(x) = c_1 e^{\lambda_1 x} + c_2 e^{\lambda_2 x}$.
    2.  **One Real Repeated Root ($\lambda$):** The general solution is $y(x) = (c_1 + c_2 x)e^{\lambda x}$.
    3.  **Complex Conjugate Roots ($\lambda = \alpha \pm i\beta$):** The general solution is $y(x) = e^{\alpha x}(c_1 \cos(\beta x) + c_2 \sin(\beta x))$.

### Euler-Cauchy Equations
These are equations of the form: $x^2y'' + axy' + by = 0$.
* **Method:** Assume a power solution $y=x^m$. Substituting yields an auxiliary quadratic equation in $m$:
    $$m(m-1) + am + b = 0$$.
* **Solutions:** The roots of this equation determine the form of the general solution, similar to the constant coefficient case:
    1.  **Two Distinct Real Roots:** $y(x) = c_1 x^{m_1} + c_2 x^{m_2}$.
    2.  **One Real Repeated Root:** $y(x) = (c_1 + c_2 \ln x)x^m$.
    3.  **Complex Conjugate Roots:** $y(x) = x^{\alpha}(c_1 \cos(\beta \ln x) + c_2 \sin(\beta \ln x))$.

### Existence and Uniqueness of Solutions - Wronskian
For a second-order linear homogeneous ODE, any linear combination of two solutions is also a solution. The general solution is a linear combination of two **linearly independent** solutions $y_1$ and $y_2$.
* **Wronskian:** To test for linear independence, we use the Wronskian determinant:
    $$ W(y_1, y_2) = \det\begin{pmatrix} y_1 & y_2 \\ y_1' & y_2' \end{pmatrix} = y_1 y_2' - y_2 y_1' $$
* **Theorem:** Two solutions $y_1, y_2$ form a basis of solutions (are linearly independent) on an interval $I$ if and only if their Wronskian $W$ is non-zero for some $x \in I$.

### Solution by Variation of Parameters
This method solves non-homogeneous ODEs: $y'' + p(x)y' + q(x)y = r(x)$.
* **Method:** If the homogeneous solution is $y_h = c_1y_1 + c_2y_2$, we seek a particular solution of the form $y_p = u(x)y_1 + v(x)y_2$. Then, the general solution is:
$$ y(x) = y_h + y_p = c_1y_1 + c_2y_2 + u(x)y_1 + v(x)y_2 $$
where $u(x)$ and $v(x)$ are functions to be determined.
* **Conditions for $u$ and $v$:** To find $u$ and $v$, we impose the condition that the derivatives of $y_p$ with respect to $x$ satisfy:
    1.  $u'y_1 + v'y_2 = 0$
    2.  $u'y_1' + v'y_2' = r(x)$
* **Solution Steps:**
1.  Solve the first equation for $v'$ in terms of $u'$: $v' = -\dfrac{u'y_1}{y_2}$.
2.  Substitute $v'$ into the second equation to get a single equation in terms of $u'$.
3.  Solve for $u'$ and then integrate to find $u$.
4.  Substitute $u$ back to find $v$.
5.  Finally, substitute $u$ and $v$ back into the expression for $y_p$ to get the particular solution.

### Method of Undetermined Coefficients
This method is used for solving non-homogeneous linear ODEs with constant coefficients, where the non-homogeneous term $r(x)$ is a polynomial, exponential, sine, or cosine function.
* **Method:** Assume a particular solution of the same form as the non-homogeneous term, but with undetermined coefficients. Substitute this assumed solution into the ODE and solve for the coefficients by matching terms.
| Term in $r(x)$ | Form of Particular Solution |
|----------------|-----------------------------|
| Polynomial of degree $n$ | $y_p = A_n x^n + A_{n-1} x^{n-1} + \ldots + A_0$ |
| Exponential $Ae^{kx}$ | $y_p = Be^{kx}$ |
| Sine/Cosine $A\sin(kx) + B\cos(kx)$ | $y_p = C\sin(kx) + D\cos(kx)$ |
| $ke^{\alpha x}\sin(\beta x)$ or $ke^{\alpha x}\cos(\beta x)$ | $y_p = e^{\alpha x}(C\sin(\beta x) + D\cos(\beta x))$ |
* **Note:** If the assumed form of the particular solution is also a solution of the homogeneous equation, multiply by $x$ (or a higher power of $x$) to ensure linear independence.

### Higher-Order Linear ODEs
The methods for second-order linear ODEs with constant coefficients extend naturally. For an $n^{th}$-order equation, the characteristic equation becomes a polynomial of degree $n$, and the general solution is a linear combination of $n$ linearly independent solutions based on the roots of this polynomial.

---
## Systems of ODEs and Qualitative Methods

### Systems of ODEs
Many applications involve multiple interacting quantities, leading to systems of differential equations. A first-order system can be written in matrix form:
$$ \vec{y}' = A(t)\vec{y} + \vec{g}(t) $$
where $\vec{y}(t)$ is a vector of dependent variables and $A(t)$ is a matrix of coefficients.

* **Constant-Coefficient Homogeneous Systems ($\vec{y}' = A\vec{y}$):**
    * **Method:** We assume a solution of the form $\vec{y} = \vec{x}e^{\lambda t}$. Substituting this into the system leads to the algebraic **eigenvalue problem**:
        $$ A\vec{x} = \lambda\vec{x} $$
    * The solutions $\lambda$ are the eigenvalues of matrix $A$, and $\vec{x}$ are the corresponding eigenvectors. The general solution is a linear combination of terms of the form $\vec{x}e^{\lambda t}$. The nature of the solutions depends on whether the eigenvalues are real, distinct, repeated, or complex.

* **Phase Plane Analysis:** For 2D autonomous systems ($\vec{y}' = \vec{f}(\vec{y})$), the **phase plane** is a plot of $y_2$ vs $y_1$. Solution trajectories are plotted as curves, and the pattern reveals the qualitative behavior of the system.
    * **Critical Points:** Equilibrium points where $\vec{y}'=\vec{0}$. Eigenvalue Criteria for Critical Points:
        | Name | $p = \Lambda_1 + \lambda_2$ | $q = \lambda_1\lambda_2$ | $\Delta = p^2 - 4q$ | Stability |
        |------|-----------------------------|--------------------------|---------------------|----------|
        | Node (Stable) | $p < 0$ | $q > 0$ | $\Delta > 0$ | Stable |
        | Node (Unstable) | $p > 0$ | $q > 0$ | $\Delta > 0$ | Unstable |
        | Saddle Point | $p = 0$ | $q < 0$ | $\Delta < 0$ | Unstable |
        | Spiral Point (Stable) | $p < 0$ | $q < 0$ | $\Delta < 0$ | Stable |
        | Spiral Point (Unstable) | $p > 0$ | $q < 0$ | $\Delta < 0$ | Unstable |
    * **Stability of Critical Points (for linear systems):** The behavior near a critical point is determined by the eigenvalues of the matrix $A$. Critical points can be classified as nodes (stable/unstable), saddle points, centers, or spiral points (stable/unstable).
* **Qualitative Methods for Nonlinear Systems:** The behavior of a nonlinear system near a critical point can often be approximated by linearizing the system around that point using the Jacobian matrix:
    $$ J = \dfrac{\partial(x,y)}{\partial(u,v)} = \begin{pmatrix} \dfrac{\partial x}{\partial u} & \dfrac{\partial x}{\partial v} \\ \dfrac{\partial y}{\partial u} & \dfrac{\partial y}{\partial v} \end{pmatrix} = \dfrac{\partial x}{\partial u} \dfrac{\partial y}{\partial v} - \dfrac{\partial x}{\partial v} \dfrac{\partial y}{\partial u}$$
    where $u$ and $v$ are the independent variables. The Jacobian matrix evaluated at a critical point gives the linear approximation of the system near that point.
* **Eigenvalues of the Jacobian:** The eigenvalues of the Jacobian matrix at a critical point determine the stability and type of the critical point. If the real parts of all eigenvalues are negative, the critical point is stable; if any eigenvalue has a positive real part, it is unstable.
* **Nonhomogeneous Linear Systems:** Can be solved using methods like variation of parameters or by diagonalizing the matrix $A$.

---
## Applications

Differential equations are ubiquitous in modeling real-world phenomena.
* **Population Dynamics:** The logistic equation $P' = kP(1-P/K)$ models population growth with a carrying capacity.
* **Orthogonal Trajectories:** Finding families of curves that intersect a given family of curves at right angles.
* **Physics: Mass–Spring Systems:** The motion of a mass on a spring is described by a second-order linear ODE:
    * Free Oscillations: $my'' + cy' + ky = 0$ (damped harmonic motion).
    * Forced Oscillations & Resonance: $my'' + cy' + ky = F_0\cos(\omega t)$.
* **Physics: Electric Circuits:** The charge $q(t)$ in an RLC circuit is governed by:
    $Lq'' + Rq' + \dfrac{1}{C}q = E(t)$. This equation is mathematically identical to the damped, forced oscillator equation.

---
## Key Takeaways: The Power of Describing Change

Differential equations are one of the most powerful tools in mathematics, providing the language to model and analyze dynamic systems across all of science and engineering.

* **Modeling Change:** ODEs relate a function to its rates of change, capturing the essence of systems that evolve over time.
* **Classification is Key:** Understanding whether an equation is linear, non-homogeneous, first-order, or second-order guides the choice of solution method.
* **A Rich Toolkit for Solutions:** A wide array of analytical techniques exists, from separation of variables and integrating factors for first-order equations to using characteristic equations and variation of parameters for higher-order linear equations.
* **Systems and Stability:** Systems of ODEs model interacting components, and their behavior can be analyzed qualitatively in the phase plane, with stability determined by the eigenvalues of the system's matrix.
* **Numerical and Advanced Methods:** When exact solutions are not possible, numerical methods (like Euler's) provide approximations, while advanced techniques (like series solutions and Laplace transforms) solve a broader class of complex problems.
* **Ubiquitous Applications:** From the simple oscillations of a pendulum to the complex dynamics of ecosystems and electrical circuits, differential equations are the mathematical bedrock for modeling the world around us.

The study of differential equations is a journey into the heart of how systems change, offering profound insights and predictive power.