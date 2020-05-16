---
title: Stochastic Processes - Markov Chains to Brownian Motion
description: Probability that evolves in time. A Markov chain forgets its past given its present, which makes long-run behaviour computable from a transition matrix. Poisson processes count rare events; martingales formalise a fair game; and Brownian motion is the scaling limit that ties random walks to the heat equation.
date: 2020-05-16
draft: false
slug: /maths/stochastic-processes
tags:
  - Mathematics
  - Probability
---

Static [probability](/citadel/maths/probablity-statistics) asks about one random outcome. A **stochastic process** is a collection of random variables indexed by time — $\{X_t\}$ — modelling something that keeps changing randomly: a queue's length, a stock price, a gene's frequency, the state of a machine. The questions change too: not just "what's the distribution of $X_t$" but "does it settle into a steady state," "how long until it hits zero," "given the path so far, what's the best guess for later."

The processes that are actually tractable are the ones with structure. The most important structure is the **Markov property** — the future depends on the past only through the present — and most of this post is what that buys you.

---

## Markov chains

A discrete-time chain on a finite or countable **state space** is defined by a **transition matrix** $P$ with $P_{ij} = \Pr(X_{n+1} = j \mid X_n = i)$; rows sum to $1$. The Markov property is the assumption baked in: $\Pr(X_{n+1} = j \mid X_n = i, X_{n-1}, \dots) = P_{ij}$ — history beyond the current state is irrelevant.

**Chapman–Kolmogorov.** The $n$-step transition probabilities are just $P^n$: $\Pr(X_n = j \mid X_0 = i) = (P^n)_{ij}$. Matrix powers *are* the evolution.

If the initial distribution is a row vector $\pi_0$, then $\pi_n = \pi_0 P^n$.

### Classifying states

- **Accessible / communicating** — $j$ is accessible from $i$ if $(P^n)_{ij} > 0$ for some $n$; if also $i$ from $j$, they **communicate**. Communication partitions the states into classes.
- **Irreducible** — the whole chain is one communicating class.
- **Recurrent vs transient** — starting from $i$, is return to $i$ certain (recurrent) or does the chain have positive probability of never coming back (transient)? A finite irreducible chain is recurrent; on infinite state spaces it can go either way (the simple random walk is recurrent in 1D and 2D, transient in 3D — "a drunk man finds his way home, a drunk bird may not").
- **Periodic** — state $i$ has period $d$ if returns are only possible at multiples of $d$; period $1$ is **aperiodic**.

### Stationary distributions and convergence

A distribution $\pi$ is **stationary** if $\pi P = \pi$ — apply a step and nothing changes. It's a left eigenvector of $P$ with eigenvalue $1$.

**Fundamental theorem for Markov chains.** An irreducible, aperiodic chain on a finite state space has a *unique* stationary distribution $\pi$, and $\pi_n \to \pi$ from *any* start. Moreover $\pi_i = 1 / \mathbb{E}[\text{return time to } i]$ — states visited often have large stationary probability.

**Detailed balance.** If $\pi_i P_{ij} = \pi_j P_{ji}$ for all $i, j$, then $\pi$ is stationary and the chain is **reversible** (the movie of the chain run backward has the same statistics). Detailed balance is easy to check and is how you *design* a chain with a target stationary distribution — the basis of [MCMC](/citadel/artificial-intelligence/theorotical-ml).

**Mixing time** — how many steps until $\pi_n$ is within $\varepsilon$ of $\pi$ in total variation. Governed by the **spectral gap** ($1 - |\lambda_2|$, where $\lambda_2$ is the second-largest eigenvalue in modulus): a big gap means fast mixing.

### A worked example

Two-state weather chain: state $S$ (sunny), $R$ (rainy), $P = \begin{pmatrix} 0.9 & 0.1 \\ 0.5 & 0.5 \end{pmatrix}$. Solve $\pi P = \pi$ with $\pi_S + \pi_R = 1$: $0.9\pi_S + 0.5\pi_R = \pi_S \Rightarrow 0.5\pi_R = 0.1\pi_S \Rightarrow \pi_R = 0.2\pi_S$. So $\pi = (5/6, 1/6)$ — in the long run, sunny five days out of six, regardless of today's weather.

---

## Continuous-time Markov chains

Time is now continuous; the chain sits in a state for an $\text{Exponential}(\lambda_i)$ holding time, then jumps. The object replacing $P$ is the **generator matrix** $Q$: off-diagonal $q_{ij} \ge 0$ is the rate of jumping $i \to j$, and $q_{ii} = -\sum_{j \ne i} q_{ij}$. The distribution evolves by $\frac{d\pi}{dt} = \pi Q$, solved by the matrix exponential $\pi(t) = \pi(0)\,e^{Qt}$. Stationarity is $\pi Q = 0$.

**Birth–death processes** — jumps only to neighbouring states, rate $\lambda_i$ up and $\mu_i$ down. Queues are birth–death chains: **M/M/1** (Poisson arrivals rate $\lambda$, exponential service rate $\mu$, one server) has stationary queue-length distribution geometric with ratio $\rho = \lambda/\mu$, so the mean number in system is $\rho/(1 - \rho)$ — which blows up as $\rho \to 1$, the reason a server at 95% utilisation has terrible latency. **Little's law**, $L = \lambda W$ (average number in system = arrival rate × average time in system), holds far more generally and is the one queueing identity to remember.

---

## The Poisson process

The canonical model of "rare events happening at a steady average rate" — arrivals, radioactive decays, requests. Equivalent definitions:

- The number of events in any interval of length $t$ is $\text{Poisson}(\lambda t)$, and disjoint intervals are independent.
- The inter-arrival times are i.i.d. $\text{Exponential}(\lambda)$.

Key properties: **superposition** (merging independent Poisson processes of rates $\lambda_1, \lambda_2$ gives rate $\lambda_1 + \lambda_2$), **thinning** (keep each event with probability $p$ → Poisson rate $\lambda p$), and **memorylessness** (given no event yet, the wait for the next is still $\text{Exponential}(\lambda)$ — the exponential is the only continuous memoryless distribution).

---

## Martingales

A process $\{M_n\}$ is a **martingale** with respect to the history if $\mathbb{E}[M_{n+1} \mid \text{history up to } n] = M_n$ — the best prediction of the next value is the current value. It formalises a **fair game**: your expected fortune after the next bet equals your fortune now.

Random walk with mean-zero steps is a martingale. So is $S_n^2 - n$ for a $\pm 1$ walk (a compensated martingale).

**Optional stopping theorem.** Under mild conditions (bounded stopping time, or bounded increments), $\mathbb{E}[M_\tau] = \mathbb{E}[M_0]$ for a stopping time $\tau$ — you can't beat a fair game with a clever stopping rule.

*Gambler's ruin* falls straight out: bet $\pm 1$ fairly, start at $k$, stop at $0$ or $N$. The walk $S_n$ is a martingale, so $k = \mathbb{E}[S_\tau] = 0 \cdot \Pr(\text{ruin}) + N \cdot \Pr(\text{win})$, giving $\Pr(\text{reach } N \text{ before } 0) = k/N$. Using the martingale $S_n^2 - n$ and optional stopping gives the *expected duration*, $k(N - k)$.

Martingales are the backbone of modern probability: concentration inequalities (Azuma–Hoeffding for bounded-increment martingales), convergence theorems (a bounded martingale converges almost surely), and the pricing of financial derivatives (an arbitrage-free price is a martingale under a risk-neutral measure).

---

## Brownian motion

Take a simple random walk, speed it up and scale it down: $B_t^{(n)} = \frac{1}{\sqrt n} S_{\lfloor nt \rfloor}$. As $n \to \infty$ this converges (Donsker's theorem — a functional [central limit theorem](/citadel/maths/probablity-statistics)) to **Brownian motion** $\{B_t\}$:

- $B_0 = 0$; increments $B_t - B_s \sim \mathcal{N}(0, t - s)$; disjoint increments independent.
- Paths are continuous everywhere but **differentiable nowhere** — infinitely jagged at every scale (self-similar: $B_{ct}$ has the same law as $\sqrt c\, B_t$).
- $\mathbb{E}[B_t^2] = t$ — displacement grows like $\sqrt t$, the diffusive scaling.

![A bell curve widening as time increases, the Gaussian density of Brownian motion spreading with standard deviation proportional to the square root of time.](../images/normal-distribution.png "The position at time t is Gaussian with variance t; this density spreading as the square root of time is exactly the fundamental solution of the heat equation. Source: Wikimedia Commons.")

Brownian motion is a martingale *and* a Markov process, and it's the bridge to PDEs: the density of $B_t$ solves the [heat equation](/citadel/maths/pde) $u_t = \tfrac12 u_{xx}$, and more generally $u(x, t) = \mathbb{E}_x[\phi(B_t)]$ solves it with initial data $\phi$ (the **Feynman–Kac** correspondence). Harmonic functions are exactly those for which $B_t$-averages are preserved — probability and potential theory are the same subject.

**Stochastic calculus** builds integrals $\int f\, dB$ against this nowhere-differentiable path (the **Itô integral**), with a modified chain rule — **Itô's lemma**, $df(B_t) = f'(B_t)\,dB_t + \tfrac12 f''(B_t)\,dt$, the extra $\tfrac12 f''\,dt$ coming from the quadratic variation $(dB)^2 = dt$. **Stochastic differential equations** $dX_t = \mu(X_t)\,dt + \sigma(X_t)\,dB_t$ model everything from asset prices (geometric Brownian motion, the Black–Scholes model) to noisy physical systems.

---

## The one idea to keep

The Markov property — the future depends on the past only through the present — is what makes a random process computable: for a chain, the whole evolution is powers of the transition matrix, and an irreducible aperiodic chain converges to a unique stationary distribution $\pi$ solving $\pi P = \pi$, no matter where it started. Poisson processes model steady-rate rare events through memoryless exponential gaps; martingales formalise a fair game and, via optional stopping, solve hitting-probability problems like gambler's ruin in one line; and Brownian motion is the $\sqrt t$-scaling limit of a random walk that connects all of this to the heat equation.
