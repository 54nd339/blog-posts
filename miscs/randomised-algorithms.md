---
title: Rolling the Dice - Your In-Depth Guide to Randomized Algorithms!
description: Explore Randomized Algorithms - from probability basics, Las Vegas vs. Monte Carlo, and Randomized Quick Sort to Chernoff's bounds, graph models, and Markov chains.
date: 2023-08-05
draft: false
slug: /pensieve/misc/randomised-algorithms
tags:
  - Randomised Algorithms
  - CS Basics
---

Hey algorithm aficionados and probability pioneers!  In the world of computer science, we often strive for deterministic solutions – algorithms that behave predictably and give the same output for the same input every time. But what if I told you that deliberately injecting **randomness** into our algorithms can often lead to simpler, faster, or even the *only known* solutions to some incredibly complex problems?

Welcome to the intriguing realm of **Randomized Algorithms**! These are algorithms that make random choices at certain points during their execution. Don't mistake this for haphazardness; it's a carefully controlled use of chance to gain an advantage, often in terms of average-case performance, simplicity of design, or the ability to tackle problems where deterministic approaches fall short.

From sorting lists more efficiently on average to verifying massive computations and navigating complex graphs, randomization is a powerful tool in the algorithmic toolkit. Let's explore how it works its magic!

---
## Part 1: Embracing Chance – Introduction to Randomization

Before we dive into specific algorithms, let's lay some groundwork.

* **A Whirlwind Tour of Probability Theory (The Essentials)**:
  To understand randomized algorithms, we need a few basic concepts from probability:

  * **Sample Space**: The set of all possible outcomes of an experiment.
  * **Event**: A subset of the sample space.
  * **Probability**: A number between 0 and 1 assigned to an event, indicating its likelihood.
  * **Random Variable**: A variable whose value is a numerical outcome of a random phenomenon.
  * **Expected Value (Expectation)**: The long-run average value of a random variable. A key tool is the **linearity of expectation**, which states that the expectation of a sum of random variables is the sum of their expectations ($E[X+Y] = E[X] + E[Y]$), regardless of whether they are independent!
* **When Randomness Solves Puzzles (Classic Examples)**:

  * **Verification of String Equality**: Suppose you have two extremely long strings, A and B, stored on different machines, and you want to check if they are identical. Sending the entire strings can be costly. Instead, you can use **fingerprinting**: pick a random prime $p$, compute $A \pmod p$ and $B \pmod p$. If $A \pmod p \neq B \pmod p$, then $A \neq B$. If $A \pmod p = B \pmod p$, then $A$ is *likely* equal to $B$. The probability of error (a "collision" where $A \neq B$ but $A \pmod p = B \pmod p$) can be made very small by choosing $p$ appropriately from a large enough range.
  * **Polynomial Identity Testing (Schwartz-Zippel Lemma)**: Given a multivariate polynomial $P(x_1, ..., x_n)$ (represented, say, as an arithmetic circuit), is it identically zero? Evaluating it at random points chosen from a sufficiently large set can tell us: if $P$ is non-zero, it will likely evaluate to a non-zero value. If it's always zero at random points, it's highly likely to be the zero polynomial.
  * **Matrix Multiplication Verification (Freivalds' Algorithm)**: To check if $A \times B = C$ for $n \times n$ matrices, instead of re-computing $A \times B$ (which takes $O(n^3)$ or $O(n^{2.373})$ time), pick a random $n \times 1$ vector $r$. Compute $A(Br)$ and $Cr$. If $A(Br) - Cr \neq 0$, then $A \times B \neq C$. If they are equal, $A \times B = C$ with high probability. This check takes only $O(n^2)$ time!
* **Two Flavors of Randomization: Las Vegas and Monte Carlo**:
  Randomized algorithms generally fall into two categories:

  1. **Las Vegas Algorithms**: These algorithms *always* produce the correct answer. However, their running time is a random variable; they might be fast on average but occasionally slow. The expected running time is often analyzed.
     * *Example*: Randomized Quick Sort.
  2. **Monte Carlo Algorithms**: These algorithms have a deterministic running time (or one that's bounded), but they might produce an incorrect answer with a certain (hopefully small) probability. This error probability can often be reduced by running the algorithm multiple times and taking a majority vote or other consensus.
     * *Example*: Freivalds' algorithm for matrix multiplication verification, Karger's min-cut algorithm (initial versions).
* **Key Probabilistic Concepts Revisited**:

  * **Expectations (Expected Value)**: Crucial for analyzing the average-case performance of Las Vegas algorithms. As mentioned, linearity of expectation is a workhorse.
  * **Jensen's Inequality**: For a convex function $\phi$, $E[\phi(X)] \ge \phi(E[X])$. For instance, if $\phi(x) = x^2$ (which is convex), then $E[X^2] \ge (E[X])^2$. This is related to variance, as $Var(X) = E[X^2] - (E[X])^2 \ge 0$.
  * **Geometric Distribution**: Describes the probability of the number of trials needed to get the first success in a sequence of independent Bernoulli trials (each trial has success probability $p$). The expected number of trials is $1/p$. Useful for analyzing algorithms that repeat a random step until a condition is met.

---
## Part 2: Shuffling and Searching with Randomness
Randomness is particularly effective in algorithms for sorting and searching.

* **Randomized Quick Sort and its Expected Run-time**:

  * **The Problem with Deterministic Quick Sort**: If the pivot selection is bad (e.g., always picking the smallest or largest element), Quick Sort degrades to $O(n^2)$ performance.
  * **The Randomized Fix**: Choose the pivot element *randomly* from the subarray (or shuffle the input array initially). This makes it highly unlikely to consistently pick bad pivots.
  * **Expected Run-time Analysis**: With random pivot selection, the expected number of comparisons is $O(n \log n)$. The intuition is that a random pivot is likely to be "good enough" (not too close to the extremes) often enough to achieve balanced partitions on average.
* **Diving Deeper into Random Variables – Understanding Deviations**:

  * **Variance ($Var(X)$ or $\sigma^2$)**: Measures the spread or dispersion of a random variable around its mean. $Var(X) = E[(X - E[X])^2] = E[X^2] - (E[X])^2$. A small variance means values tend to be close to the mean.
  * **Moments**: The $k$-th moment of a random variable $X$ is $E[X^k]$. The first moment is the mean. Variance uses the first two moments.
  * **Chebyshev's Inequality**: A powerful tool that bounds the probability that a random variable $X$ deviates from its expected value $E[X]$ by more than a certain amount, using only its mean and variance. Specifically, $P(|X - E[X]| \ge k) \le Var(X)/k^2$. This gives us a way to say that large deviations are unlikely if the variance is small, without knowing the full distribution.
* **Classic Probabilistic Puzzles with Algorithmic Implications**:

  * **Coupon Collector's Problem**: Suppose there are $n$ distinct coupons, and you get one random coupon at each trial (with replacement). How many trials do you expect to make before collecting all $n$ coupons?
    * **Expected Time**: The expected number of trials is $n H_n \approx n \ln n$, where $H_n$ is the $n$-th Harmonic number.
    * *Applications*: Analyzing the expected time for certain randomized processes to cover all states or complete a task, like aspects of randomized broadcast in networks.
  * **Randomized Median Finding (QuickSelect variation)**: The problem is to find the $k$-th smallest element in an unsorted list of $n$ elements.
    * **Algorithm Idea**: Choose a random pivot, partition the array around it (like in Quick Sort). If the pivot is at the $k$-th position, you're done. If the pivot is at position $j > k$, recursively search in the left subarray. If $j < k$, recursively search in the right subarray for the $(k-j)$-th element.
    * **Analysis**: The expected number of comparisons is $O(n)$, which is optimal. Randomization helps ensure the pivot choices lead to good reductions in problem size on average.

---
## Part 3: Bounding Deviations – Chernoff's Bounds and Applications
While Chebyshev's inequality gives a general bound, for sums of independent random variables (which occur often in algorithm analysis), **Chernoff bounds** provide much tighter (exponentially decreasing) tail probabilities.

* **Chernoff Bounds: What They Are**:
  They provide strong upper bounds on the probability that the sum $X = \sum X_i$ of independent random variables $X_i$ (often 0-1 indicator variables, i.e., Bernoulli trials) deviates significantly from its expectation $E[X] = \mu$.

  * There are various forms, but generally, for $\delta > 0$:
    * $P(X \ge (1+\delta)\mu) \le \exp(-\mu \delta^2 / 3)$ (for "not too large $\delta$")
    * $P(X \le (1-\delta)\mu) \le \exp(-\mu \delta^2 / 2)$
      These show that the probability of large deviations decreases exponentially fast.
* **Conceptual Idea of Derivation**: The proof technique often involves applying Markov's inequality ($P(Y \ge a) \le E[Y]/a$ for non-negative $Y$) to $e^{tX}$ for some well-chosen $t > 0$, using the independence of $X_i$ to handle $E[e^{tX}] = E[\prod e^{tX_i}] = \prod E[e^{tX_i}]$, and then optimizing over $t$.
* **Illustrative Applications**:

  * **Sum of Poisson Trials / Coin Flips**: If you flip $n$ fair coins, the expected number of heads is $n/2$. Chernoff bounds can give a very tight bound on the probability of getting, say, more than $0.6n$ heads or fewer than $0.4n$ heads.
  * **Set Balancing**: Given $n$ vectors $v_i \in \{-1, 1\}^m$. Find signs $x_i \in \{-1, 1\}$ such that the length of the sum vector $X = \sum x_i v_i$ is minimized. If we choose $x_i$ randomly, Chernoff bounds can show that with high probability, each coordinate of $X$ will be small, thus the overall vector length will be small.
* **Randomness in Networks**:

  * **Packet Routing in Sparse Networks**: In networks like grids or hypercubes, randomized routing algorithms can distribute packets more evenly and avoid congestion hotspots that might arise with deterministic routing, leading to better average delivery times. Chernoff bounds are used to analyze the probability of high congestion on any single edge.
  * **Permutation Routing on the Hypercube/Butterfly**: The task is to route $N$ packets, where each processor is the source of one packet and the destination of one packet (a permutation). **Valiant's two-phase randomized routing scheme** is a classic example:
    1. Each packet is sent to a random intermediate destination.
    2. From the random intermediate destination, the packet is sent to its actual final destination.
       With high probability (analyzed using Chernoff bounds), this scheme avoids congestion and routes all packets efficiently on networks like the hypercube or butterfly.

---
## Part 4: Randomness in Structures – Graph Models & Hashing ️
Randomness is fundamental in generating and analyzing certain combinatorial structures.

* **Probabilistic Surprises**:

  * **Birthday Paradox**: How many people must be in a room for the probability that at least two share a birthday to be greater than 50%? The answer is surprisingly low: just 23 people.
    * *Implications*: This illustrates that collisions can happen more frequently than one might intuitively expect, which is relevant for hash function analysis (hash collisions).
* **Balls and Bins Model**: A classic probabilistic model where $m$ balls are thrown independently and uniformly at random into $n$ bins.

  * **Key Question**: What is the maximum number of balls in any single bin (maximum load)?
  * **Application to Bucket Sort**: If numbers are hashed into $n$ buckets and then sorted within buckets, the maximum load in a bucket affects the overall sorting time.
  * **Application to Hashing (Chained Hashing)**: When $m$ keys are hashed into an $n$-slot hash table with chaining, the length of the longest chain is related to the maximum load in the balls and bins model. With $m=n$ balls/keys, the maximum load is $O(\log n / \log \log n)$ with high probability.
* **Poisson Distribution**:
  A discrete probability distribution that expresses the probability of a given number of events occurring in a fixed interval if these events occur with a known constant mean rate $\lambda$ and independently.

  * **Poisson Approximation to Binomial**: If $n$ is large and $p$ is small, the binomial distribution $B(n,p)$ can be well-approximated by the Poisson distribution with $\lambda = np$.
  * **Application to Hashing**: When hashing $n$ keys into $m$ buckets where $n/m = \lambda$ is constant (and $m$ is large), the number of keys hashing to a particular bucket often behaves like a Poisson random variable with mean $\lambda$. This simplifies collision analysis.
* **Random Graph Models**: Studying the properties of graphs generated by a random process.

  * **Erdős–Rényi Model $G(n,p)$**: A graph with $n$ vertices where each of the $\binom{n}{2}$ possible edges exists independently with probability $p$.
  * **Threshold Phenomena**: Many graph properties (e.g., connectivity, containing a giant component, containing a Hamiltonian cycle) appear quite suddenly. As $p$ increases, the probability of the property holding transitions sharply from near 0 to near 1 around a critical threshold value of $p$.
  * **Hamiltonian Cycles in Random Graphs**: For example, a $G(n,p)$ graph is likely to contain a Hamiltonian cycle if $p$ is around $(\ln n + \ln \ln n)/n$.

---
## Part 5: Advanced Probabilistic Tools & Techniques ‍

Some more sophisticated probabilistic tools provide powerful ways to analyze complex randomized algorithms.

* **Markov Chains**:
  A sequence of random states where the probability of transitioning to the next state depends *only* on the current state, not on the sequence of states that preceded it (the **memoryless property**).

  * **Key Components**: States, transition probabilities.
  * **Classification of States**:
    * **Recurrent**: If the chain starts in this state, it will eventually return to this state with probability 1.
    * **Transient**: If the chain starts in this state, there is a non-zero probability it will never return.
    * **Absorbing**: Once entered, cannot be left.
  * **Ergodic Chains**: Chains where it's possible to get from any state to any other state, and which have a unique stationary distribution (long-term probabilities of being in each state).
  * **Gambler's Ruin**: A classic example. A gambler starts with $i$ dollars and makes $1 bets, winning with probability $p$ and losing with probability $1-p$. The game ends if the gambler is ruined (0 dollars) or reaches a target of $N$ dollars. These are absorbing states. We can calculate the probability of ruin.
* **Random Walks on Undirected Graphs**:
  A process where at each step, you move from the current vertex to one of its neighbors, typically chosen uniformly at random.

  * **Properties Analyzed**:
    * **Hitting Time $H(u,v)$**: Expected number of steps to reach vertex $v$ starting from vertex $u$.
    * **Commute Time $C(u,v)$**: Expected number of steps to go from $u$ to $v$ and then back to $u$. $C(u,v) = H(u,v) + H(v,u)$.
    * **Cover Time $C(G)$**: Expected number of steps to visit every vertex in the graph, starting from a given vertex (or worst-case starting vertex).
  * Applications in network exploration, sampling, and search algorithms.
* **S-T Connectivity Algorithm (Randomized Approach for Undirected Graphs)**:

  * **Problem**: Given two vertices $s$ and $t$ in an undirected graph $G=(V,E)$, determine if there is a path connecting them.
  * **Random Walk Approach**: Start a random walk from vertex $s$. If vertex $t$ is reached within a certain number of steps (e.g., $2m|V|$, where $m$ is number of edges), then $s$ and $t$ are connected.
  * **Analysis**: If $s$ and $t$ are in the same connected component, a random walk of sufficient length is guaranteed to reach $t$ with high probability. If they are not connected, the walk will never reach $t$. This gives a Monte Carlo algorithm with one-sided error. This algorithm is particularly useful for its very small space complexity ($O(\log n)$ space), making it suitable for analyzing connectivity in massive graphs where BFS/DFS might be too memory-intensive.

---
## Conclusion: The Surprising Power of a Coin Toss!
Randomized algorithms demonstrate that embracing uncertainty can lead to remarkably elegant, simple, and efficient solutions. We've seen how a dash of randomness can:

* **Verify complex computations** quickly (string/polynomial/matrix verification).
* **Improve average-case performance** dramatically (Randomized Quick Sort, Median Finding).
* Provide **strong probabilistic guarantees** about deviations from expectation (Chernoff bounds).
* Help analyze and design algorithms for **networks and complex structures** (routing, random graphs, hashing).
* Form the basis for powerful analytical tools like **Markov chains and random walks**.

While they might not always give a guaranteed correct answer in a fixed time (Monte Carlo) or a fixed time every run (Las Vegas), their expected performance or high probability of success often makes them the algorithms of choice. Randomness is not just a fallback; it's a fundamental design principle in modern computer science!
