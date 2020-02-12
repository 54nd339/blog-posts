---
title: The Binomial Theorem - Every Term Without Multiplying It Out
description: Expanding (x+y)ⁿ term by term is really a counting problem — how many ways to pick y exactly k times. That view gives the coefficients, Pascal's triangle, the middle and greatest terms, series sums by calculus, and the infinite version for fractional powers.
date: 2020-02-12
draft: false
slug: /maths/binomial-theory
tags:
  - Mathematics
  - Combinatorics
---

You want the coefficient of $x^2$ in $(1 + x)^5$. Multiplying five brackets together and collecting terms works, but it is 32 partial products to sort. The binomial theorem gives the answer straight off — it is $\binom{5}{2} = 10$ — because the real question is not algebraic but combinatorial: *in how many of the five brackets do you pick the $x$?*

This post builds the theorem from that counting idea, then uses it as a tool: the binomial coefficients and their identities, Pascal's triangle, how to pull out the middle term or the numerically largest term of an expansion, three ways to sum a series of coefficients (substitution, differentiation, integration), the trick of feeding in a complex number, and the infinite series you get when the exponent is negative or fractional.

---
## Where it comes from

Write $(x + y)^n$ as $n$ copies of $(x + y)$ multiplied together. Expanding means going through every way to choose one term — $x$ or $y$ — from each of the $n$ brackets and multiplying the choices. A product ends up as $x^{n-k}y^k$ whenever you picked $y$ from exactly $k$ of the brackets. The number of ways to do that is the number of $k$-element subsets of an $n$-element set, written $\binom{n}{k}$. Collect the identical products and each $x^{n-k}y^k$ carries the coefficient $\binom{n}{k}$.

---
## The statement

For a non-negative integer $n$,
$$ (x + y)^n = \sum_{k=0}^{n} \binom{n}{k} x^{n-k} y^k = \binom{n}{0}x^n + \binom{n}{1}x^{n-1}y + \cdots + \binom{n}{n}y^n $$
The **general term** — the $(k{+}1)$th, since $k$ starts at $0$ — is
$$ T_{k+1} = \binom{n}{k}\, x^{n-k} y^k $$
Almost every question about an expansion is "which $k$?" plugged into this one formula.

---
## Binomial coefficients

$$ \binom{n}{k} = \frac{n!}{k!\,(n-k)!}, \qquad 0! = 1 $$
$\binom{n}{k}$ ("$n$ choose $k$") counts $k$-subsets of $n$ things. The identities you actually use:

- **Ends:** $\binom{n}{0} = \binom{n}{n} = 1$ — one way to take nothing, one way to take everything.
- **Symmetry:** $\binom{n}{k} = \binom{n}{n-k}$ — choosing $k$ to keep is choosing $n-k$ to drop.
- **Pascal's rule:** $\binom{n-1}{k-1} + \binom{n-1}{k} = \binom{n}{k}$ — split on whether a fixed element is in the subset.
- **Absorption:** $k\binom{n}{k} = n\binom{n-1}{k-1}$.
- **Row sum:** put $x = y = 1$: $\displaystyle\sum_{k=0}^{n}\binom{n}{k} = 2^n$ (every subset of an $n$-set).
- **Alternating sum:** put $x = 1$, $y = -1$: $\displaystyle\sum_{k=0}^{n}(-1)^k\binom{n}{k} = 0$ for $n \ge 1$, so the even-$k$ coefficients and odd-$k$ coefficients each total $2^{n-1}$.
- **Weighted sum:** $\displaystyle\sum_{k=0}^{n} k\binom{n}{k} = n\,2^{n-1}$.

**Pascal's triangle** stacks the rows $\binom{n}{0}, \binom{n}{1}, \ldots, \binom{n}{n}$, each entry the sum of the two above it — Pascal's rule made visual.

![Pascal's triangle, rows 0 through 5: a single 1 at the apex, then 1 1, then 1 2 1, then 1 3 3 1, then 1 4 6 4 1, then 1 5 10 10 5 1. The edges are all ones and every interior entry is the sum of the two above it.](../images/pascals-triangle.png "Pascal's triangle: entry k of row n is the binomial coefficient (n choose k), and equals the sum of the two entries above it. Source: Wikimedia Commons.")

---
## Standard expansions

Setting $x = 1$ is the common case:
$$ (1 + x)^n = 1 + nx + \frac{n(n-1)}{2!}x^2 + \frac{n(n-1)(n-2)}{3!}x^3 + \cdots + x^n $$
Replace $x$ by $-x$ for $(1 - x)^n = \sum_k \binom{n}{k}(-1)^k x^k$, and $y$ by $-y$ for $(x - y)^n = \sum_k (-1)^k \binom{n}{k} x^{n-k} y^k$ — odd-power terms flip sign.

When $|x| \ll 1$, every term past the linear one is negligible, giving the workhorse approximation
$$ (1 + x)^n \approx 1 + nx $$
— the reason a relativistic factor $\gamma = (1 - v^2/c^2)^{-1/2}$ becomes $1 + \tfrac12 v^2/c^2$ at low speed.

---
## Reading off one term

An expansion of $(x + y)^n$ has $n + 1$ terms.

**Middle term.** If $n$ is even there is one, the $\left(\tfrac{n}{2} + 1\right)$th, namely $\binom{n}{n/2}x^{n/2}y^{n/2}$. If $n$ is odd there are two, the $\tfrac{n+1}{2}$th and $\tfrac{n+3}{2}$th.

**Ratio of consecutive terms.** From the general term,
$$ \frac{T_{k+1}}{T_k} = \frac{\binom{n}{k}}{\binom{n}{k-1}}\cdot\frac{y}{x} = \frac{n - k + 1}{k}\cdot\frac{y}{x} $$

**Numerically greatest term** of $(1 + X)^n$: the terms grow while $\left|\dfrac{T_{k+1}}{T_k}\right| = \left|\dfrac{n-k+1}{k}X\right| > 1$ and shrink after. Solving the crossover, let
$$ m = \frac{(n + 1)\,|X|}{1 + |X|} $$
If $m$ is an integer, $T_m$ and $T_{m+1}$ tie for largest; otherwise $T_{\lfloor m \rfloor + 1}$ is it. For $(x + y)^n$, factor out $x^n$ and apply this to $(1 + y/x)^n$.

**Term independent of $x$.** In something like $\left(ax^p + b x^{-q}\right)^n$, write the general term, collect the total power of $x$ as a function of $k$, set it to zero, and solve for $k$.

---
## Summing series with it

Treating $(1 + x)^n = C_0 + C_1 x + \cdots + C_n x^n$ (with $C_k = \binom{n}{k}$) as an identity in $x$ lets calculus act on the coefficients.

- **Substitute.** $x = 1$: $\sum C_k = 2^n$. $x = -1$: $\sum (-1)^k C_k = 0$. Add and halve: $C_0 + C_2 + \cdots = C_1 + C_3 + \cdots = 2^{n-1}$.
- **Differentiate**, then set $x = 1$: from $n(1 + x)^{n-1} = C_1 + 2C_2 x + \cdots + nC_n x^{n-1}$,
  $$ \sum_{k=1}^{n} k\,C_k = n\,2^{n-1} $$
- **Integrate** from $0$ to $1$: from $\dfrac{(1+x)^{n+1} - 1}{n+1} = C_0 x + C_1\dfrac{x^2}{2} + \cdots$,
  $$ \sum_{k=0}^{n} \frac{C_k}{k + 1} = \frac{2^{n+1} - 1}{n + 1} $$
- **Multiply two expansions.** Compare the coefficient of $x^r$ on both sides of $(1 + x)^m (1 + x)^n = (1 + x)^{m+n}$:
  $$ \sum_{k=0}^{r} \binom{m}{k}\binom{n}{r - k} = \binom{m + n}{r} \qquad \text{(Vandermonde)} $$
  With $m = n = r$ and symmetry, $\displaystyle\sum_{k=0}^{n}\binom{n}{k}^2 = \binom{2n}{n}$.

---
## Feeding in a complex number

The theorem holds for complex $x, y$, and that unlocks trigonometric identities. Expanding $(\cos\theta + i\sin\theta)^n$ by the binomial theorem and matching it to $\cos n\theta + i\sin n\theta$ from [De Moivre's theorem](/citadel/maths/complex-numbers) gives $\cos n\theta$ and $\sin n\theta$ as polynomials in $\cos\theta, \sin\theta$.

For coefficient-weighted trig sums, put $x = e^{i\theta}$ into $(1 + x)^n$:
$$ (1 + e^{i\theta})^n = \sum_{k=0}^{n} \binom{n}{k}\big(\cos k\theta + i\sin k\theta\big) $$
The left side simplifies using $1 + e^{i\theta} = 2\cos(\theta/2)\,e^{i\theta/2}$, so it equals $2^n \cos^n(\theta/2)\,\big(\cos\tfrac{n\theta}{2} + i\sin\tfrac{n\theta}{2}\big)$. Matching real parts,
$$ \sum_{k=0}^{n} \binom{n}{k}\cos k\theta = 2^n \cos^n\!\frac{\theta}{2}\,\cos\frac{n\theta}{2} $$
and the imaginary parts give the companion sine sum.

---
## More than two terms

For $(x_1 + x_2 + \cdots + x_m)^n$, the same "pick a term from each bracket" argument gives the **multinomial theorem**:
$$ (x_1 + \cdots + x_m)^n = \sum_{n_1 + \cdots + n_m = n} \frac{n!}{n_1!\,n_2!\cdots n_m!}\; x_1^{n_1}\cdots x_m^{n_m} $$
summed over non-negative $n_i$. The multinomial coefficient $\dfrac{n!}{n_1!\cdots n_m!}$ counts the ways to label the $n$ brackets with which variable they contributed. Setting every $x_i = 1$ gives $m^n$.

---
## When the exponent isn't a positive integer

Newton's generalisation: for any real $n$ and $|x| < 1$,
$$ (1 + x)^n = 1 + nx + \frac{n(n-1)}{2!}x^2 + \frac{n(n-1)(n-2)}{3!}x^3 + \cdots $$
an *infinite* series with general term $\dfrac{n(n-1)\cdots(n-r+1)}{r!}x^r$. If $n$ is a positive integer the factor $n - r$ eventually hits zero and the series terminates — the ordinary theorem is the special case. Otherwise it runs forever and converges only for $|x| < 1$. Useful instances:

- $(1 - x)^{-1} = 1 + x + x^2 + \cdots$ — the geometric series.
- $(1 + x)^{-1} = 1 - x + x^2 - \cdots$
- $(1 - x)^{-2} = 1 + 2x + 3x^2 + 4x^3 + \cdots$
- $(1 + x)^{1/2} = 1 + \tfrac12 x - \tfrac18 x^2 + \tfrac{1}{16}x^3 - \cdots$

---
## The one idea to keep

The theorem is small but it wears many hats: it is a counting statement about subsets, the generating identity behind Pascal's triangle, the source of the binomial distribution's $\binom{n}{k}p^k(1-p)^{n-k}$, and — in its infinite form — the first few terms of a Taylor series for $(1 + x)^n$. The counting view is the one to keep: every coefficient in sight is the answer to "how many ways?" The natural next stop is the algebra of the counts themselves, in [permutations and combinations](/citadel/maths/permutation-combination), and the series machinery continues in [sequences and progressions](/citadel/maths/progression).
