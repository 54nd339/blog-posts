---
title: Game Theory - Equilibrium, Information, and Mechanism Design
description: When your best move depends on someone else's best move, and theirs on yours. Strategic-form games and dominance, Nash equilibrium in pure and mixed strategies, the Prisoner's Dilemma and coordination games, games of incomplete information and Bayesian Nash equilibrium, signalling and bargaining, and mechanism design - engineering the rules to get the outcome you want.
date: 2023-07-11
draft: false
slug: /miscs/game-theory
tags:
  - Game Theory
  - Economics
---

Game theory studies decisions where your payoff depends on what other people choose, and theirs depends on what you choose. It is the standard tool for reasoning about strategy in economics, politics, biology, and computer science. This post builds up from the simplest representation of a game to games where players hide information, signal, bargain, and where a designer sets the rules to steer the outcome. The [applied maths](/citadel/maths/applied-maths) post has a compact version; this one goes into the structure.

## Strategic-form games

The **strategic form** (or normal form) specifies three things:

- **Players** — the decision-makers. They are assumed **rational** (maximising their own payoff), and rationality is often assumed to be **common knowledge**: everyone is rational, everyone knows everyone is rational, and so on without end.
- **Actions and strategies** — an **action** is a single move; a **strategy** is a complete plan specifying an action at every decision point a player could face. In a one-shot game they coincide.
- **Payoffs** — the value a player gets from an outcome (one strategy per player). Payoffs may be **ordinal** (a ranking) or **cardinal** (numbers).

### Dominance

- **Strict dominance** — strategy $S_i$ strictly dominates $S_i'$ if it pays more *whatever* the others do. A rational player never plays a strictly dominated strategy.
- **Weak dominance** — $S_i$ pays at least as much everywhere and strictly more somewhere. Avoided too, but the argument is weaker.
- **Iterated deletion of strictly dominated strategies (IDSDS)** — remove all strictly dominated strategies, then repeat in the smaller game. If one outcome survives, that is the solution. *Example:* if pricing Low always earns Firm A less than pricing Medium, whatever Firm B does, delete Low for A; then re-examine the reduced game.

### Nash equilibrium

A **Nash equilibrium** is a strategy profile where no player can improve by *unilaterally* deviating, given the others' strategies — mutual best response.

- A **pure strategy** is a definite choice; a **mixed strategy** randomises over pure strategies with fixed probabilities.
- Games with no pure-strategy equilibrium often have one in mixed strategies. In Rock–Paper–Scissors it is to play each option with probability $1/3$.

### Two classic games

**Prisoner's Dilemma** (payoffs are years in prison; lower is better):

| | Suspect 2 silent | Suspect 2 confesses |
| --- | :---: | :---: |
| **Suspect 1 silent** | $(-1, -1)$ | $(-10, 0)$ |
| **Suspect 1 confesses** | $(0, -10)$ | $(-5, -5)$ |

Confess strictly dominates silent for each player ($0 > -1$ if the other is silent; $-5 > -10$ if the other confesses). So $(\text{Confess}, \text{Confess})$ is the unique Nash equilibrium and the IDSDS outcome — even though both would be better off silent. It models arms races, price wars, and emissions cuts: individually rational choices give a collectively worse result.

**Battle of the Sexes** (coordination):

| | He: opera | He: football |
| --- | :---: | :---: |
| **She: opera** | $(2, 1)$ | $(0, 0)$ |
| **She: football** | $(0, 0)$ | $(1, 2)$ |

Two pure Nash equilibria — both-opera and both-football — plus a mixed one. The problem is *which* to coordinate on. **Focal points** (Schelling points) resolve it: an outcome that stands out as natural or precedented, without communication.

### Market examples

- **Cournot duopoly** — firms choose quantities; each firm's best-response quantity depends on the other's. Equilibrium is where the best-response curves cross.
- **Bertrand duopoly** — firms choose prices. With identical products and equal marginal cost, the Nash equilibrium is both pricing at marginal cost, as under perfect competition.

## Perfect versus imperfect information

**Perfect information** — every player knows the full history of moves whenever it is their turn (chess, Go). These games use the **extensive form** (a game tree), and a strategy specifies a choice at every node a player could reach.

- **Backward induction** — start at the terminal nodes; at each preceding decision node, pick the mover's optimal action given what follows; replace the node with the resulting payoffs; work up to the root. The result is a **subgame perfect Nash equilibrium (SPNE)** — a Nash equilibrium in every subgame.
- **Median voter theorem** — if voters have single-peaked preferences on one dimension and candidates only want votes, both candidates converge on the median voter's ideal point; any deviation lets the opponent capture a majority by moving closer to the median.

**Imperfect information** — at some decision a player does not know the full history (moves are simultaneous, or actions are hidden).

- **Information set** — a set of nodes a player cannot tell apart; the strategy must prescribe the same action across the whole set.
- **Chance moves** — random events controlled by "Nature" (the deal in poker). Players maximise **expected payoff**: $\sum (\text{payoff} \times \text{probability})$ over outcomes. See [probability](/citadel/maths/probablity-statistics).

## Incomplete information: Bayesian games

When players do not know others' payoffs or preferences:

- Each player has a **type** — private information affecting their payoff (a firm's cost, a bidder's valuation).
- Players hold **beliefs** — prior distributions over others' types.
- A strategy maps *each possible type* of a player to an action.
- A **Bayesian Nash equilibrium** is a profile where each type of each player maximises expected payoff given its beliefs and the others' (type-contingent) strategies.

**Cournot with incomplete information.** Two firms choose quantities simultaneously; each knows its own cost ($c_L$ or $c_H$) but only a probability $p$ that the other is low-cost. Each firm's quantity is a function of its own cost, chosen to maximise expected profit given its belief about the other.

**Public goods with incomplete information.** Players simultaneously choose contributions to a shared good; contributing is individually costly. Private information about valuations worsens the **free-rider problem** — uncertainty makes players more conservative. Related: the **tragedy of the commons**, where a shared resource is overused because individual incentives diverge from collective good.

**Auctions with incomplete information.** Each bidder has a private valuation (their type); the strategy is how much to bid as a function of that valuation. In a **common-value auction** (the item is worth the same to everyone, but estimates differ — an oil lease), the winner is typically whoever most overestimated: the **winner's curse**. Rational bidders shade their bids down to compensate.

**Information structures.** *One-sided* — A knows B's type but not vice versa, and can exploit it. *Two-sided* — both have private information (the Cournot case). *Multi-sided* — many players each with private information; forming beliefs and best responses gets sharply harder, and players must weigh how their actions reveal or conceal their own type.

## Signalling

One informed player (Sender) takes an action that an uninformed player (Receiver) observes before acting; the Sender's type is private.

- **Credibility.** A signal is informative only if it is **costly** in a way that only some types would bear. In **Spence's job-market model**, high-ability workers invest in education as a costly signal; it works only because education costs less for high-ability types.
- **Equilibria.** *Separating* — different types send different signals, so the Receiver infers the type exactly. *Pooling* — all types send the same signal, so it conveys nothing and the Receiver falls back on priors. Partial pooling also occurs.

## Bargaining

**Rubinstein alternating offers.** Two players split a surplus by making alternating offers. Each has a **discount factor** $\delta_i \in (0,1)$ measuring patience — a dollar tomorrow is worth $\delta_i$ dollars today. The unique subgame perfect equilibrium is *immediate agreement*; the first mover gets more, and the more patient a player (higher $\delta$), the larger their share.

**Nash bargaining solution (axiomatic).** A non-strategic "fair" outcome defined by four axioms:

1. **Pareto efficiency** — no other agreement improves one player without hurting another.
2. **Symmetry** — identical players get equal gains.
3. **Independence of irrelevant alternatives** — removing unchosen options (keeping the solution and disagreement point) does not change the solution.
4. **Scale invariance** — a linear rescaling of a player's utility does not change the outcome.

The solution maximises the **Nash product** — the product of the players' gains over their disagreement points.

**The link.** The strategic Rubinstein model provides a non-cooperative foundation for the axiomatic Nash solution: as the delay between offers goes to zero (players almost equally patient), the Rubinstein SPNE division converges to the Nash bargaining division.

*Trade in a market:* a buyer values an item at $V_B$, a seller's cost is $C_S$, with $V_B > C_S$. Any price $P \in [C_S, V_B]$ realises a surplus $V_B - C_S$; the split depends on the bargaining protocol and patience, and breakdown destroys the surplus. *Spectrum licences:* firms holding adjacent spectrum blocks bargain over coordinated use (guard bands, standards); their relative patience and the protocol set the terms and the value of the licences.

## Mechanism design

Setting the rules of interaction so that self-interested participants produce a desired system-wide outcome.

- **Auctions.** The **Vickrey auction** (second-price sealed-bid) is **truthful** — bidding your true valuation is a dominant strategy — which makes it efficient.
- **The revelation principle.** If a social choice function can be implemented by *some* mechanism in Bayesian Nash equilibrium, it can be implemented by a **direct** mechanism in which participants truthfully report their types and the mechanism computes the outcome. This narrows the search for good mechanisms to truthful ones.
- **In computer science.** Resource allocation in cloud computing, task scheduling in [distributed systems](/citadel/miscs/distributed-algorithms), recommendation systems that elicit honest preferences, and ad auctions — Google's generalised second-price auction chooses allocation and pricing rules to raise revenue while keeping bidding near-truthful.

## The one idea to keep

A game is defined by its players, their strategies, their payoffs, and *who knows what*. Nash equilibrium — no profitable unilateral deviation — is the anchor solution, refined to subgame perfection when moves are sequential and to Bayesian Nash equilibrium when types are private. Mechanism design runs the logic backward: fix the outcome you want, and design the rules so that rational play produces it.
