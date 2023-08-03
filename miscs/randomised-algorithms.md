---
title: Randomized Algorithms - Las Vegas, Monte Carlo, and Tail Bounds
description: Deliberately making random choices inside an algorithm often buys simplicity, speed, or the only known solution. The probability you need, the Las Vegas / Monte Carlo split, randomized quicksort and its expected bound, Chebyshev and Chernoff tail bounds, balls-and-bins and random-graph models, and random walks.
date: 2023-08-03
draft: false
slug: /miscs/randomised-algorithms
tags:
  - Randomized Algorithms
  - Algorithms
  - Probability
---

Deterministic algorithms give the same output every run. A **randomized algorithm** makes random choices during execution — and that controlled use of chance often produces something simpler, faster on average, or the only solution anyone knows. Freivalds' check verifies a matrix product in $O(n^2)$ instead of recomputing it; a random pivot rescues quicksort from its worst case. This post covers the probability you need, the two flavours of randomized algorithm, and the analytical tools — expectation, Chebyshev, Chernoff, random walks — that make them work.

## The probability you need

- **Sample space** — all possible outcomes of an experiment. **Event** — a subset of it. **Probability** — a number in $[0,1]$ assigned to an event.
- **Random variable** — a numerical function of the outcome. **Expected value** $E[X]$ — its long-run average.
- **Linearity of expectation** — $E[X + Y] = E[X] + E[Y]$, whether or not $X$ and $Y$ are independent. This one identity does most of the analytical work below. See [probability](/citadel/maths/probablity-statistics).

## Where randomness solves a problem cheaply

**String equality by fingerprinting.** Two long strings $A$ and $B$ live on different machines; sending either is expensive. Pick a random prime $p$ and compare $A \bmod p$ with $B \bmod p$. If they differ, $A \neq B$ for certain. If they match, $A = B$ with high probability — a collision ($A \neq B$ but equal mod $p$) is rare when $p$ is drawn from a large range.

**Polynomial identity testing (Schwartz–Zippel).** Given a multivariate polynomial $P(x_1, \ldots, x_n)$ as an arithmetic circuit, is it identically zero? Evaluate it at a random point from a large enough set. If $P$ is not the zero polynomial, it is very unlikely to vanish there; if it keeps evaluating to zero at random points, it almost certainly is the zero polynomial.

**Matrix multiplication verification (Freivalds' algorithm).** To check $A \times B = C$ for $n \times n$ matrices without recomputing the product, pick a random vector $r$ and compare $A(Br)$ with $Cr$. If they differ, $AB \neq C$. If they match, $AB = C$ with high probability. The check is $O(n^2)$, against $O(n^{2.37})$ to multiply.

## Two flavours

- **Las Vegas** — *always* correct; the running time is a random variable. You analyse the expected time. *Example:* randomized quicksort.
- **Monte Carlo** — bounded running time; *possibly* wrong, with small probability. Repeat and take a majority vote to drive the error down. *Example:* Freivalds' algorithm, Karger's min-cut.

Supporting concepts: the **geometric distribution** — the number of independent trials until the first success, with success probability $p$, has expectation $1/p$ (useful for "repeat a random step until it works" analyses); **Jensen's inequality** — for a convex $\phi$, $E[\phi(X)] \ge \phi(E[X])$, so $E[X^2] \ge (E[X])^2$, which is exactly $\mathrm{Var}(X) \ge 0$.

## Randomness in sorting and selection

**Randomized quicksort.** Deterministic quicksort degrades to $\Theta(n^2)$ when the pivot is consistently near an extreme. Choosing the pivot uniformly at random from the subarray makes that outcome vanishingly unlikely, and the expected number of comparisons is $O(n \log n)$ — a random pivot gives a balanced-enough partition often enough. See [sorting and searching](/citadel/algorithms/SortingSearching).

**Randomized selection (quickselect).** To find the $k$-th smallest of $n$ elements, pick a random pivot and partition. If the pivot lands at position $k$, done; otherwise recurse into the side that contains rank $k$. The expected number of comparisons is $O(n)$ — optimal.

**Coupon collector.** With $n$ distinct coupons, one drawn uniformly at random per trial, the expected number of trials to collect them all is $n H_n \approx n \ln n$, where $H_n$ is the $n$-th harmonic number. It bounds how long a randomized process takes to cover every state.

## Bounding deviations

**Chebyshev's inequality.** Using only the mean and variance,
$$\Pr\bigl(|X - E[X]| \ge k\bigr) \le \frac{\mathrm{Var}(X)}{k^2}.$$
Large deviations are unlikely when the variance is small — no knowledge of the full distribution required. Here $\mathrm{Var}(X) = E[(X - E[X])^2] = E[X^2] - (E[X])^2$, and the $j$-th **moment** is $E[X^j]$.

**Chernoff bounds.** For a sum $X = \sum_i X_i$ of *independent* random variables (often $0/1$ indicators) with $\mu = E[X]$, and any $\delta > 0$:
$$\Pr\bigl(X \ge (1+\delta)\mu\bigr) \le e^{-\mu\delta^2/3}, \qquad \Pr\bigl(X \le (1-\delta)\mu\bigr) \le e^{-\mu\delta^2/2}.$$
The tails decay *exponentially* in $\mu$, far tighter than Chebyshev's $1/k^2$.

*Where the exponential comes from:* apply Markov's inequality ($\Pr(Y \ge a) \le E[Y]/a$ for non-negative $Y$) to $Y = e^{tX}$ for a chosen $t > 0$. Independence gives $E[e^{tX}] = \prod_i E[e^{tX_i}]$, a product you can bound term by term; then minimise the resulting bound over $t$.

**Applications.** Bounding the number of heads in $n$ fair coin flips tightly around $n/2$. **Set balancing** — given vectors $v_i \in \{-1,1\}^m$, choosing signs $x_i \in \{-1,1\}$ at random makes every coordinate of $\sum x_i v_i$ small with high probability. **Packet routing** — randomized routing spreads load and avoids the congestion hotspots that deterministic routing creates; **Valiant's two-phase scheme** routes each packet first to a random intermediate node, then to its true destination, and Chernoff bounds show congestion stays low on the hypercube or butterfly.

## Randomness in structures

**Birthday paradox.** In a room of just 23 people, the probability that two share a birthday exceeds $1/2$. Collisions happen sooner than intuition expects — the basis of birthday attacks on hash functions and of [Bloom filter](/citadel/data-structures/bloom-filters) false-positive analysis.

**Balls and bins.** Throw $m$ balls independently and uniformly into $n$ bins. With $m = n$, the fullest bin holds $\Theta(\log n / \log\log n)$ balls with high probability. This is the maximum chain length in a hash table with chaining, and the per-bucket cost in bucket sort. See [data structures](/citadel/data-structures/data-structures).

**Poisson distribution.** The number of events in a fixed interval, given a constant mean rate $\lambda$ and independence. For large $n$ and small $p$, the binomial $B(n,p)$ is well approximated by Poisson with $\lambda = np$ — which is why, when hashing $n$ keys into $m \approx n/\lambda$ buckets, a bucket's load behaves like a Poisson variable with mean $\lambda$, simplifying collision analysis.

**Random graphs (Erdős–Rényi $G(n,p)$).** $n$ vertices, each of the $\binom{n}{2}$ edges present independently with probability $p$. Many properties — connectivity, a giant component, a Hamiltonian cycle — show **threshold phenomena**: as $p$ rises, the probability of the property jumps sharply from near $0$ to near $1$ at a critical $p$. A $G(n,p)$ graph likely has a [Hamiltonian cycle](/citadel/maths/optimization-graph-theory) once $p \gtrsim (\ln n + \ln\ln n)/n$.

## Markov chains and random walks

A **Markov chain** is a sequence of states where the next state depends only on the current one (the memoryless property). States are **recurrent** (return with probability 1), **transient** (positive probability of never returning), or **absorbing** (cannot be left). An **ergodic** chain can reach any state from any state and has a unique stationary distribution. *Gambler's ruin:* starting with $\$i$, betting $\$1$ per round (win with probability $p$), the game ends at $\$0$ or $\$N$ — both absorbing — and you can compute the ruin probability exactly.

A **random walk** on an undirected graph moves to a uniformly random neighbour each step. Quantities of interest: **hitting time** $H(u,v)$ (expected steps from $u$ to $v$), **commute time** $C(u,v) = H(u,v) + H(v,u)$, and **cover time** (expected steps to visit every vertex).

**$s$–$t$ connectivity by random walk.** To test whether $s$ and $t$ are connected in an undirected graph, walk randomly from $s$; if $t$ is reached within $O(m|V|)$ steps, report connected. If they are in the same component, a walk of that length reaches $t$ with high probability; if not, it never does. This Monte Carlo algorithm needs only $O(\log n)$ space — enough to hold the current vertex — which is why it beats BFS/DFS on graphs too large to hold a visited set.

## The one idea to keep

Randomness buys two things. It **breaks adversarial structure**: a random pivot means no fixed input is the worst case. And it makes **rare events provably rare**: linearity of expectation gives the average, Chebyshev bounds deviation from it with just the variance, and Chernoff bounds it exponentially for sums of independent variables — so "with high probability" becomes a number you can put in a proof.
