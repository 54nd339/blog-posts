---
title: Reinforcement Learning - Learning From Reward
description: Supervised learning needs labelled answers; reinforcement learning needs only a reward signal and the freedom to try things. The framework is a Markov decision process, the goal is a policy that maximizes long-run reward, and the central equations - Bellman's - relate the value of a state to the values of the states it leads to.
date: 2023-04-04
draft: false
slug: /artificial-intelligence/reinforcement-learning
tags:
  - Artificial Intelligence
  - Machine Learning
---

[Supervised learning](/citadel/artificial-intelligence/ml) needs a teacher — every training example comes with the right answer. Reinforcement learning removes the teacher. An **agent** takes actions in an **environment**, receives a scalar **reward**, and has to work out for itself which sequences of actions lead to more reward over time. Nobody tells it the right move; it only learns that a move was good, sometimes long after making it.

This is how a program learns to play Go or StarCraft, control a robot arm, tune a data-centre's cooling, or — via RLHF — make a language model prefer helpful responses. The framework underneath all of it is the Markov decision process, and the load-bearing idea is the Bellman equation.

## Markov decision processes

An MDP is:

- **States** `s` — the situations the agent can be in.
- **Actions** `a` — what it can do in each state.
- **Transition** `P(s' | s, a)` — the probability of landing in `s'` after doing `a` in `s`. (The agent may not know this.)
- **Reward** `R(s, a, s')` — the immediate scalar feedback.
- **Discount** `γ ∈ [0, 1)` — how much future reward is worth relative to immediate reward. `γ` close to 1 = far-sighted; smaller = myopic.

"Markov" means the transition and reward depend only on the current state and action, not the full history — the state captures everything relevant.

The agent's behaviour is a **policy** `π(a | s)` — a mapping from states to (a distribution over) actions. The goal is the policy that maximizes the expected **return**: the discounted sum of future rewards, `G_t = R_{t+1} + γR_{t+2} + γ²R_{t+3} + …`.

## Value functions and the Bellman equations

Two functions quantify "how good":

- **State value** `V^π(s)` — expected return starting from `s` and following `π` thereafter.
- **Action value** `Q^π(s, a)` — expected return from taking `a` in `s`, then following `π`.

The **Bellman expectation equation** says the value of a state is the immediate reward plus the discounted value of where you end up:

$$V^\pi(s) = \sum_a \pi(a|s) \sum_{s'} P(s'|s,a)\big[R(s,a,s') + \gamma V^\pi(s')\big]$$

The **Bellman optimality equation** replaces "average over the policy" with "take the best action":

$$V^*(s) = \max_a \sum_{s'} P(s'|s,a)\big[R(s,a,s') + \gamma V^*(s')\big]$$

Every RL algorithm is, in some form, a way of solving or approximating this recursion.

## When you know the model: dynamic programming

If `P` and `R` are known, you can compute the optimal policy directly:

- **Value iteration** — initialize `V` arbitrarily, repeatedly apply the Bellman optimality update to every state; `V` converges to `V*`, then read off the greedy policy.
- **Policy iteration** — alternate *policy evaluation* (solve the Bellman expectation equation for the current policy) and *policy improvement* (make the policy greedy w.r.t. the new values). Converges in few iterations.

Both need the full model and a state space small enough to sweep — a toy setting. Real RL is **model-free**: learn from experience without knowing `P`.

## Model-free prediction: Monte Carlo and TD

- **Monte Carlo** — play a full episode, then update each visited state's value toward the *actual* return observed. Unbiased, but you must wait for the episode to end and returns are high-variance.
- **Temporal-difference (TD)** — update after *every step*, toward the *estimated* return `R_{t+1} + γV(s_{t+1})` — a **bootstrap**: you update a guess toward another guess. Lower variance, works on continuing tasks, learns online. The TD(0) update:

$$V(s_t) \leftarrow V(s_t) + \alpha\big[R_{t+1} + \gamma V(s_{t+1}) - V(s_t)\big]$$

The bracket is the **TD error** — the surprise. **TD(λ)** and eligibility traces interpolate between one-step TD and Monte Carlo.

## Model-free control: SARSA and Q-learning

Learn `Q` (so you can act greedily without a model):

- **SARSA** (on-policy) — update `Q(s, a)` toward `R + γQ(s', a')` where `a'` is the action the current policy *actually took*. Learns the value of the policy it's following, exploration and all.
- **Q-learning** (off-policy) — update toward `R + γ max_{a'} Q(s', a')` — the *best* next action, regardless of what was taken. Learns the optimal policy while behaving more exploratively. The workhorse.

Both need **exploration** — always acting greedily means never discovering a better option. **ε-greedy** (act randomly with probability ε), **optimistic initialization**, **UCB**, and **Thompson sampling** balance exploiting what you know against exploring what you don't. The pure version of that trade-off is the **multi-armed bandit** — an MDP with one state.

## Function approximation and deep RL

Tabular `Q` needs one entry per state-action pair — impossible for images or continuous states. Replace the table with a **function approximator** `Q(s, a; θ)`, typically a neural network.

This breaks the nice convergence guarantees — the **deadly triad** of function approximation + bootstrapping + off-policy learning can diverge. **DQN** (2015, Atari from pixels) made it work with two tricks:

- **Experience replay** — store transitions in a buffer and train on random minibatches, breaking the correlation between consecutive samples.
- **Target network** — compute the bootstrap target with a slowly-updated copy of the network, so you're not chasing a moving target.

Refinements: **Double DQN** (decouple action selection from evaluation to reduce over-estimation), **Dueling DQN** (separate state-value and advantage streams), **prioritized replay**.

## Policy gradient methods

Instead of learning values and acting greedily, directly parameterize the policy `π(a | s; θ)` and ascend the gradient of expected return.

- **REINFORCE** — increase the log-probability of actions in proportion to the return that followed them. Unbiased, high variance.
- **Baselines and advantage** — subtract a state-dependent baseline (usually `V(s)`) so you reinforce actions by how much *better than average* they were (the **advantage** `A = Q − V`), cutting variance without adding bias.
- **Actor–critic** — an **actor** (the policy) and a **critic** (a value estimate) trained together; the critic supplies the advantage. **A2C/A3C** parallelize it.
- **PPO** — the current default for continuous control and RLHF: a policy-gradient method that clips the update so the new policy can't move too far from the old one in a single step, which makes training stable and forgiving. **SAC** and **DDPG** are the off-policy continuous-action counterparts.

## Model-based, MCTS, and RLHF

- **Model-based RL** learns `P` and plans with it (imagined rollouts), which is sample-efficient — important when real interaction is expensive (robots, healthcare).
- **Monte Carlo Tree Search** + a learned value/policy net is **AlphaGo/AlphaZero**: use the net to guide a search over future move sequences, and use the search results to train the net. Self-play with no human data in AlphaZero.
- **RLHF** (reinforcement learning from human feedback) aligns language models: humans rank model outputs, a **reward model** is trained to predict those rankings, and the language model is fine-tuned with PPO to maximize that reward (with a penalty for drifting too far from the original model). It's a small, careful application of the machinery above, and much of a modern assistant's helpfulness comes from it.

## The one idea to keep

Reinforcement learning replaces labelled answers with a reward signal and trial-and-error. Frame the problem as a Markov decision process, and the Bellman equations tie the value of a state to the discounted values of its successors — every algorithm approximates that recursion. Model-free control learns `Q` from experience (Q-learning) or ascends a policy gradient directly (PPO), always trading exploration against exploitation. Swap the lookup table for a neural network and, with experience replay and a target network to keep it stable, you get deep RL — the same math behind Atari, AlphaZero, and RLHF.
