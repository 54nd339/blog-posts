---
title: Knowledge Representation and Planning - Structuring What an Agent Knows
description: How an agent organises knowledge about the world - ontological engineering, categories and objects, frames and inheritance, reified events and scripts, and default reasoning with its conflicts - then how it turns that knowledge into action through STRIPS, state-space planning, partial-order plans, and planning as satisfiability.
date: 2023-02-07
draft: false
slug: /artificial-intelligence/knowledge-rep
tags:
  - Artificial Intelligence
  - Knowledge Representation
  - Planning
---

A [logical agent](/citadel/artificial-intelligence/logical-agents) can reason once its knowledge is written down — but *how* you write it down decides what reasoning is cheap and what is impossible. Represent "a cat is a mammal" as one fact and inheritance gives you "a cat has fur" for free; represent it badly and you re-state fur for every animal. This is **knowledge representation**: the design choices for storing what an agent knows so a machine can use it.

Once the world is represented, the agent still has to decide what to *do*. **Planning** — finding a sequence of actions from the current state to a goal — is the second half of this post.

## Ontological engineering

**Ontological engineering** is designing the formal vocabulary: the concepts, categories, and relationships of a domain, written so different systems can share and reuse them. Semantic networks and frame systems were the early tools; today the same job is done with description logics and OWL ontologies.

### Categories and objects

The base distinction is between **objects** — individual things ("Tom the cat", "my laptop") — and **categories** — groups sharing properties ("Cats", "Laptops"). Two link types connect them: `is_a` says an object instantiates a category ("Tom `is_a` Cat"); `a_kind_of` says one category is a subclass of another ("Cat `a_kind_of` Mammal").

**Frames** structure this. A frame for an object or category has **slots** for its attributes, each **filled** with a value or a pointer to another frame — a `Cat` frame with slots `color`, `sound`, `likes_cream`. The payoff is **inheritance**: if `Mammal` has `has_fur` and `Cat a_kind_of Mammal`, then `Tom` inherits `has_fur` without it being stored on `Tom` or on `Cat`.

### Events and scripts

To reason about a changing world you need to represent actions and events, not just static facts. One technique is **reification** — treat an event as an object in its own right, with roles attached:

- **Agent** — who acts (*John* in "John gave the book to Mary").
- **Object** — what the action is done to (*the book*).
- **Recipient** — who receives it (*Mary*).
- **Instrument** — the means (*post* in "sent it by post").
- **Locative** — where it happens (*school*).

A **script** captures a stereotyped sequence of events — the "restaurant script": enter, be seated, order, eat, pay, leave — so an agent can fill in steps that weren't stated.

Representing an agent's own **mental state** — its beliefs, desires, intentions — is harder and less settled. A logical agent's KB is already a representation of what it believes; the open problem is connecting that to goal-directed behaviour.

### A worked domain: internet shopping

Categories: `Product`, `Customer`, `Order`, `ShoppingCart`, `PaymentMethod`. Objects: `LaptopX200` (a `Product`), `Customer123`. Slots and fillers: `LaptopX200` has `price: 999`, `brand: "TechGiant"`, `screen: "14-inch"`; `Customer123` has `shipping_address`. Reified actions: `addToCart(Customer, Product)`, `placeOrder(Customer, ShoppingCart)`, `makePayment(Order, PaymentMethod)`, each with its own `timestamp` and `status`. Situations: the contents of a cart at a moment, the state of an order.

## Default reasoning

Most properties are typical, not universal. "Birds fly" — but penguins are birds and don't. This is handled with **defaults and overrides**: the `Birds` category carries `can_fly: true` as a default; the `Penguin` subclass overrides it with `can_fly: false`.

Conflicts arise under **multiple inheritance**. The **Nixon diamond**: Nixon is a Quaker (typically pacifist) and a Republican (typically not), so inheriting `pacifist` from both parents gives contradictory defaults. Resolving it needs a priority rule — most specific class wins, or an explicit ordering.

When new information contradicts a derived belief, the KB can become inconsistent. A **truth maintenance system** (TMS) tracks which facts were derived from which, so that retracting one premise automatically retracts everything that depended on it and nothing else.

## Planning

**Classical planning** assumes an environment that is fully observable, deterministic, finite, static, and discrete. A planning problem is three things:

1. **Initial state** — a conjunction of positive literals, e.g. $\text{On}(A, \text{Table}) \land \text{Clear}(A)$.
2. **Goal** — a partial state to achieve, also a conjunction of literals, e.g. $\text{On}(A, B) \land \text{On}(B, C)$.
3. **Actions**, each with **preconditions** (literals that must hold to apply it) and **effects** (literals it makes true or false).

The **STRIPS** notation writes an action like:

```text
Action( Move(block, from, to)
  PRECOND: On(block, from) ∧ Clear(block) ∧ Clear(to)
  EFFECT:  On(block, to) ∧ Clear(from) ∧ ¬On(block, from) ∧ ¬Clear(to) )
```

A **solution** is an action sequence that, run from the initial state, produces a state satisfying the goal.

### State-space planning

The direct approach is [state-space search](/citadel/artificial-intelligence/search): states are world configurations, operators are the actions, the goal test checks the goal description. Search forward from the initial state (progression) or backward from the goal (regression); the path you find *is* the plan.

**Totally ordered** plans are strict linear sequences: stacking A on B on C might come out as `[Move(B, Table, C), Move(A, Table, B)]`. **Partially ordered** planning only commits to the orderings the causal structure forces. Putting on socks and shoes: `[LeftSock, LeftShoe]` and `[RightSock, RightShoe]` are independent subplans, and the planner leaves them un-interleaved until it must decide. Fewer commitments means a smaller search.

### Planning graphs

A **planning graph** is a layered structure — alternating literal levels and action levels — that a planner builds cheaply from the initial state. It bounds how soon each literal and action could possibly appear, which gives strong heuristic estimates for a state-space search, and it records **mutex** (mutually exclusive) pairs at each level, letting the planner prune combinations that can't co-occur. GraphPlan is the algorithm built directly on it.

### Planning as satisfiability

The other route compiles the whole problem into propositional logic and hands it to a SAT solver. Encode the initial state as literals true at time 0, each action as implications linking time $t$ to $t+1$ (preconditions at $t$, effects at $t+1$), and the goal as literals required true at time $T$. A satisfying assignment to the action variables *is* a valid $T$-step plan. This is **SATPlan**, and it rides the enormous engineering effort that's gone into fast SAT solvers.

## The one idea to keep

Knowledge representation and planning are two sides of the same coin: structure the world so that the useful conclusions are easy to reach, then search that structure for a sequence of actions that reaches the goal. Inheritance, reification, and defaults are about making the representation carry its weight; STRIPS, partial-order planning, and SATPlan are about extracting a plan from it without enumerating every state.
