---
title: Ternary Search - Optimizing a Unimodal Function
description: Finding the extremum of a function that rises then falls by evaluating two interior points and discarding the outer third that cannot hold the optimum, on reals by iteration and on integers with a careful endgame.
date: 2024-07-28
draft: false
slug: /cp/ternary-search
tags:
  - Competitive Programming
  - Numerical Methods
  - Searching
---

Binary search needs a **monotone** predicate. Ternary search handles the next case up: a **unimodal** function — one that strictly increases then strictly decreases (a single maximum), or the mirror. Each step evaluates two interior points and throws away a third of the interval, converging in $O(\log)$ evaluations.

## The problem

Given $f$ unimodal on $[lo, hi]$ (increasing to a peak, then decreasing — no plateaus, or handle them carefully), find the argmax (or the max value). Examples: the vertex of a parabola you can only sample, the distance from a point to a moving target minimised over time, the best split ratio in an optimisation, a [convex DP](/citadel/cp/divide-and-conquer-dp) cost as a function of one continuous parameter.

## The idea

Pick two points $m_1 < m_2$ inside $[lo, hi]$. Because $f$ is unimodal:

- if $f(m_1) < f(m_2)$, the maximum is **not** in $[lo, m_1)$ — discard it, set $lo = m_1$.
- if $f(m_1) > f(m_2)$, the maximum is not in $(m_2, hi]$ — set $hi = m_2$.
- if equal (strictly unimodal ⇒ this only happens when both are on the same side or straddle), you can safely shrink to $[m_1, m_2]$.

Choosing $m_1 = lo + (hi - lo)/3$, $m_2 = hi - (hi - lo)/3$ shrinks the interval by a factor $2/3$ per iteration — one $f$-evaluation reused per step if you keep the interior point that survives (golden-section search shrinks by $\approx 0.618$ with genuinely one eval per step).

## Algorithm (real-valued)

```python
def ternary_max(f, lo: float, hi: float, iters: int = 200) -> float:
    for _ in range(iters):
        m1 = lo + (hi - lo) / 3
        m2 = hi - (hi - lo) / 3
        if f(m1) < f(m2):
            lo = m1
        else:
            hi = m2
    return (lo + hi) / 2                          # argmax; f((lo+hi)/2) is the max value
```

200 iterations shrink the interval by $(2/3)^{200} \approx 10^{-35}$ — comfortably past any required precision, and a fixed count avoids an epsilon that is too small for the floating-point floor.

## Algorithm (integer-valued)

On integers, ternary search down to a small window, then scan:

```python
def ternary_max_int(f, lo: int, hi: int) -> int:
    while hi - lo > 2:
        m1 = lo + (hi - lo) // 3
        m2 = hi - (hi - lo) // 3
        if f(m1) < f(m2):
            lo = m1 + 1
        else:
            hi = m2 - 1
    best = lo
    for x in range(lo, hi + 1):
        if f(x) > f(best):
            best = x
    return best
```

An alternative integer form does a binary search on the discrete derivative $f(x+1) - f(x)$: it is positive before the peak, negative after — a monotone predicate, so plain binary search applies and is often less error-prone.

## Complexity

- **Time:** $O(\log((hi - lo)/\varepsilon))$ evaluations of $f$ for the reals; $O(\log(hi - lo))$ for integers. Each evaluation may itself be expensive (a simulation, a feasibility check) — that dominates.
- **Space:** $O(1)$.

## Common pitfalls

- **The function is not actually unimodal.** Ternary search silently returns a local, non-global answer on a bimodal or noisy $f$. Verify unimodality (a proof, or a dense sample on small cases).
- **Plateaus.** If $f$ can be flat at the top, `f(m1) == f(m2)` is ambiguous. Shrink to $[m_1, m_2]$ (safe if the flat region is connected), or perturb, or use the derivative-sign binary search.
- **Integer off-by-one.** `lo = m1 + 1` / `hi = m2 - 1` can overshoot past the optimum if the window is tiny; that is why you stop at `hi - lo <= 2` and scan.
- **Too few iterations.** For reals, 100–200 is standard; 50 may leave $10^{-3}$ error. Fixed count, not epsilon.
- **Minimisation vs maximisation.** Flip the comparison, or negate $f$ — do not half-convert.

## The keystone

For a unimodal $f$, sampling two interior points tells you which outer third of the interval cannot contain the optimum; discarding it each step converges in $O(\log)$ evaluations. On integers, ternary-search to a width-2 window and scan, or binary-search the sign of $f(x+1) - f(x)$ instead.
