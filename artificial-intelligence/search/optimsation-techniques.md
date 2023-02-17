---
title: Optimisation Techniques - From Gradient Descent to Evolution and Annealing
description: How AI finds a best solution - what an optimisation problem is, why classical derivative methods fail on bumpy or discrete objectives, gradient descent for the smooth case, and the two nature-inspired derivative-free methods, genetic algorithms and simulated annealing, in full.
date: 2023-02-17
draft: false
slug: /artificial-intelligence/optimsation-techniques
tags:
  - Artificial Intelligence
  - Search
  - Machine Learning
---

A large share of AI is optimisation wearing a costume. Training a network is minimising a loss over the weights. Route planning is minimising distance over orderings. Circuit design is minimising area over layouts. When the objective is smooth you can follow its slope downhill; when it's bumpy, discontinuous, or defined over discrete choices, the slope tells you nothing and you need a method that only ever *evaluates* the objective, never differentiates it.

This post covers both cases: gradient descent for the smooth one, and the two standard derivative-free methods — genetic algorithms and simulated annealing — for the rest.

## What an optimisation problem is

An optimisation problem has:

- One or more **objective functions** $y_i = f_i(x_1, \dots, x_n)$ to minimise or maximise. The $x_j$ are the **decision variables** (design parameters).
- **Constraints**: functional ones $g_i(x_1, \dots, x_n)\ \text{ROP}_i\ c_i$ where $\text{ROP}$ is $\le$, $\ge$, or $=$; and variable bounds fixing each $x_j$'s allowed range.

Problems are classified as **constrained or unconstrained**, **linear or non-linear** (in the objective and constraints), and **integer, real-valued, or mixed** (in the variables).

## Classical methods and their limits

For a single-variable smooth function, the **analytical** approach finds the points where $f'(x) = 0$ — candidate minima, maxima, or inflection points — and classifies each by the first non-zero higher derivative: if the first non-vanishing derivative $f^{(n)}(x^*)$ has even $n$ it's an optimum, and its sign says minimum ($>0$) or maximum ($<0$). Numerical methods, linear programming, and non-linear programming extend this.

These break down when:

- the problem has many variables or an expensive objective — computation blows up;
- the objective is **discontinuous** or has **many local optima** — derivative methods stall or land in a bad one;
- variables are **discrete** — the derivative doesn't exist;
- conditions change over time and the method can't adapt;
- the method isn't naturally parallelisable.

Those gaps are what the computational-intelligence methods fill.

## Gradient descent

The workhorse for smooth, differentiable objectives. Move the parameters against the gradient of the error $E(\mathbf{w})$, which points uphill:

$$\mathbf{w}_{k+1} = \mathbf{w}_k - \eta\,\nabla E(\mathbf{w}_k)$$

with **learning rate** (step size) $\eta$. It's efficient and well understood on smooth, locally convex functions, and it's how [neural networks are trained](/citadel/artificial-intelligence/dl). Its weaknesses: it needs differentiability, it can settle into a local optimum on a non-convex landscape, and the choice of $\eta$ is delicate — too large diverges, too small crawls. The momentum and adaptive-rate variants (AdaGrad, RMSprop, Adam) that address this are covered in [optimisation and generalisation in deep learning](/citadel/artificial-intelligence/generalisation).

## Derivative-free optimisation

When the objective is non-differentiable, noisy, discrete, or a black box, **derivative-free** methods explore by evaluating $f$ at chosen points and steering by the results alone, usually with a stochastic strategy. Two nature-inspired ones dominate.

## Genetic algorithms

**Genetic algorithms** (GAs), from John Holland in the 1960s–70s, are population-based search inspired by natural selection. A candidate solution is a **chromosome**; the algorithm evolves a population of them.

**Encoding.** How a solution becomes a chromosome:

- **Binary** — bit strings (e.g. item-in-or-out for the [0/1 knapsack](/citadel/algorithms/01Kanpsack)).
- **Real-valued** — vectors of real numbers, for continuous problems.
- **Permutation** — orderings, for the travelling-salesman problem.
- **Tree** — for genetic programming, or structured problems like floor planning.

**The loop:**

1. **Initial population** — a diverse set of random chromosomes.
2. **Fitness evaluation** — a fitness function, derived from the objective, scores each individual.
3. **Selection** — pick parents, biased toward fitter individuals:
   - **Roulette wheel** — selection probability proportional to fitness.
   - **Rank-based** — probability from fitness *rank*, which slows premature convergence when one individual dominates.
   - **Tournament** — pick a random subset, keep the fittest of it.
4. **Crossover (recombination)** — combine two parents into offspring: single-point, multi-point, or uniform for binary encodings; blend or simulated-binary crossover for real-valued ones. The bet is that good sub-parts from two parents combine into a better whole.
5. **Mutation** — small random changes (bit flip, or a real-valued perturbation) that keep diversity in the population and let the search reach parts of the space crossover can't.
6. **New generation** — offspring, usually with **elitism** carrying the best few parents through unchanged, replace the old population; repeat.
7. **Termination** — a generation limit, a good-enough solution, a stall in best fitness, or an exhausted compute budget.

GAs handle high-dimensional, non-linear, discrete problems well and often escape local optima because the population explores many regions at once and mutation keeps jumping. They cost a lot of objective evaluations, their parameters (population size, crossover and mutation rates) need problem-specific tuning, and they give no guarantee of finding the global optimum — just, often, a very good solution.

## Simulated annealing

**Simulated annealing** (SA) borrows from metallurgy: heat a metal, then cool it slowly so its atoms settle into a low-energy crystal instead of freezing into a flawed one. Here the "energy" is the objective to minimise, and **temperature** $T$ is a control parameter that starts high and drops on a schedule.

1. **Initialise** with a random solution $S$ and a high temperature $T$.
2. **Iterate:**
   - Generate a neighbour $S'$ by a small random perturbation of $S$.
   - Compute $\Delta E = \text{Energy}(S') - \text{Energy}(S)$.
   - If $\Delta E < 0$ (better), accept: $S \leftarrow S'$.
   - If $\Delta E \ge 0$ (worse or equal), accept anyway with probability $e^{-\Delta E / T}$. This is the key move — SA can go *uphill*, and does so more readily when $T$ is high or the worsening is small.
3. **Cool** — after some iterations at $T$, reduce it, e.g. $T \leftarrow \alpha T$ with $0 < \alpha < 1$.
4. **Terminate** when $T$ is near zero or progress has stopped.

SA is good at escaping local optima on hard combinatorial problems and is easy to implement. It's sensitive to the annealing schedule — initial temperature, cooling rate, iterations per temperature — and cools slowly enough to be optimal only in the limit, which can be slow. A [runnable version](/citadel/algorithms/AISearch) sits alongside the other local-search methods.

## The one idea to keep

Smooth and differentiable: gradient descent, and reach for the momentum/adaptive variants before anything exotic. Rugged, multimodal, but continuous: simulated annealing, which needs only a neighbour function and an objective, and whose one non-obvious move is accepting a worse solution with probability $e^{-\Delta E/T}$ so it can climb out of local optima. Discrete, combinatorial, or where you want to explore many candidates in parallel: a genetic algorithm. All three trade the guarantee of the true optimum for the ability to make progress on a problem gradient methods can't touch — and all three live or die by their hyperparameters (step size, cooling schedule, population and mutation rate).
