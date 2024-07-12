---
title: Numerical Methods for Competitive Programming - Search and Integration
description: The small numeric toolkit contests actually use — binary and ternary search on a continuous parameter, Newton's method, and Simpson integration — and where each already lives on this blog.
date: 2024-07-12
draft: false
slug: /cp/numerical-methods
tags:
  - Competitive Programming
  - Numerical Methods
  - Searching
---

Contest "numerical methods" is a short list: search for the value of a real parameter that satisfies a monotone or unimodal condition, and occasionally integrate or root-find a function you cannot solve in closed form. Most of it is covered elsewhere on this blog; this hub is the index and the "which one" guide.

## Binary search on the answer

When the feasibility of a parameter $x$ is **monotone** — true for all $x$ above (or below) a threshold — binary search finds the threshold. On integers this is the standard [binary search patterns](/citadel/dsa/binary-search-patterns); on reals, iterate a fixed number of times (100 iterations halves the interval to $2^{-100}$, far past any precision you need) or until the width is below an epsilon.

```python
def binary_search_real(feasible, lo: float, hi: float, iters: int = 100) -> float:
    for _ in range(iters):
        mid = (lo + hi) / 2
        if feasible(mid):
            hi = mid
        else:
            lo = mid
    return hi
```

Typical uses: "minimum speed to finish in time" ([Koko](/citadel/dsa/koko-eating-bananas)), "largest radius that fits", "smallest ratio", parametric-search problems where you binary-search a value and run a linear feasibility check.

## Ternary search on a unimodal function

When $f$ **rises then falls** (or falls then rises) — one interior optimum, no closed form — [ternary search](/citadel/cp/ternary-search) narrows the bracket by evaluating two interior points and discarding the third of the interval that cannot contain the optimum. $O(\log)$ evaluations; on integers, finish the last few with a linear scan.

## Root-finding and integration

- **Newton's method** — quadratically convergent root-finding for a differentiable $f$: $x \leftarrow x - f(x)/f'(x)$. Covered in [numerical analysis](/citadel/maths/numerical-analysis) (root-finding by iteration). Contest uses: high-precision $\sqrt[k]{x}$, inverting a monotone function faster than binary search when the derivative is cheap.
- **Simpson's rule** — approximate $\int_a^b f$ by fitting parabolas over subintervals; error $O(h^4)$. Also in the [numerical analysis post](/citadel/maths/numerical-analysis). Contest uses: area of a region bounded by curves, expected-value integrals, adaptive Simpson when $f$ has sharp features.
- **Simulated annealing** — randomised optimisation for non-convex objectives (geometry median, Steiner points, min enclosing shapes); covered under [optimisation techniques](/citadel/artificial-intelligence/optimsation-techniques).

## Recognising which tool

- Feasibility of $x$ is monotone → binary search on the answer.
- Objective is unimodal in $x$ → ternary search.
- Need a root of a smooth $f$ to many digits, derivative available → Newton.
- Need a definite integral of a smooth $f$ → Simpson (adaptive if not smooth).
- Objective is bumpy / combinatorial with a continuous relaxation → simulated annealing, as a last resort.

## Precision notes

- Prefer a **fixed iteration count** over an epsilon-width stopping test for real binary/ternary search — it is immune to a too-small epsilon looping forever near the machine precision floor.
- Compare with a **relative** tolerance when values span orders of magnitude.
- For "output with $10^{-6}$ absolute error", 100 halvings of any sane starting interval is overkill and safe.

## The keystone

Contest numerical work is almost entirely *search on a real parameter*, not analysis: if feasibility is monotone you binary-search the threshold, if the objective is unimodal you ternary-search the optimum, and a fixed iteration count beats an epsilon test every time. Newton, Simpson, and annealing are the occasional extras for a root, an integral, or a bumpy objective with no structure to exploit.
