---
title: Constraint Satisfaction and Adversarial Search - Structured Problems and Games
description: When a state stops being a black box - constraint satisfaction problems, the constraint graph, backtracking search and the heuristics that make it fast (MRV, forward checking, arc consistency), then adversarial search for two-player games with minimax and alpha-beta pruning.
date: 2023-01-16
draft: false
slug: /artificial-intelligence/csp
tags:
  - Artificial Intelligence
  - Search
  - Games
---

Plain [state-space search](/citadel/artificial-intelligence/search) treats a state as a black box: you can generate successors and test for a goal, and that's all you're allowed to know. A **constraint satisfaction problem** (CSP) opens the box. A state is a set of variables with values, and the goal test is a set of constraints on those values. Once the solver can see that structure, it can prune whole regions of the space that a blind search would trudge through.

This post covers CSPs and the backtracking search that solves them, then turns to **adversarial search** — the version of the problem where a second agent is trying to make you lose.

## What a CSP is

A CSP has three parts:

- **Variables** $X_1, \dots, X_n$.
- A non-empty **domain** $D_i$ of allowed values for each $X_i$.
- **Constraints** $C_1, \dots, C_m$, each naming a subset of the variables and listing the value combinations that subset is allowed to take.

A **state** is an assignment of values to some or all variables. An assignment that breaks no constraint is **consistent**; one that assigns every variable is **complete**; a **solution** is a complete, consistent assignment.

When every constraint involves just two variables — a **binary CSP** — you can draw a **constraint graph**: variables are nodes, and an edge joins two variables that share a constraint. The graph's shape tells you a lot about how hard the problem is (a tree-structured constraint graph is solvable in linear time).

## Two examples

**Map colouring.** Colour the regions of a map so no two adjacent regions share a colour. Variables: the regions (for Australia, WA, NT, Q, NSW, V, SA, T). Domains: `{red, green, blue}`. Constraints: adjacent regions differ, e.g. $WA \ne NT$. This is [graph colouring](/citadel/algorithms/GraphColoring) with three colours, and the constraint graph is literally the map's adjacency graph.

**Cryptarithmetic.** Each letter stands for a distinct digit; the sum must work.

```text
    S E N D
  + M O R E
  ---------
  M O N E Y
```

Variables: `S E N D M O R Y` plus carry digits $C_1, C_2, C_3$. Domains: `{0..9}` for letters, `{0,1}` for carries. Constraints: an `Alldiff` over the letters; one arithmetic equation per column ($D + E = Y + 10 C_1$, and so on); and the leading letters can't be zero ($S \ne 0$, $M \ne 0$). The column constraints are **higher-order** — they involve three or more variables — which is what makes cryptarithmetic more than a colouring problem.

Real CSPs of this shape: timetabling, assigning staff to shifts, transport and factory scheduling, hardware configuration.

## Varieties of CSP

Domains and constraints come in flavours:

- **Discrete, finite domains** — $d^n$ possible assignments for $n$ variables of domain size $d$. Boolean satisfiability (SAT) is the special case with $d = 2$, and it is NP-complete, so finite-domain CSPs are NP-complete in general. See [complexity classes](/citadel/algorithms/ComplexityClasses).
- **Discrete, infinite domains** — integers, strings (job scheduling over start dates). You can't enumerate the domain, so constraints must be written in a constraint language like $\text{Start}_1 + 5 \le \text{Start}_3$.
- **Continuous domains** — scheduling Hubble observations, for instance. When the constraints are linear, linear programming solves the CSP in polynomial time.

Constraints themselves are **unary** (one variable, e.g. $SA \ne \text{green}$), **binary** (two), or **higher-order** (three or more; any higher-order constraint can be rewritten with auxiliary variables into binary ones).

## Backtracking search

The workhorse is **backtracking search** — depth-first search that assigns one variable at a time and undoes the assignment when it hits a dead end. The key observation is that variable assignments **commute**: `WA=red then NT=green` reaches the same state as `NT=green then WA=red`, so at each node the solver only needs to consider assignments to *one* variable, not all of them. That drops the branching factor from $nd$ to $d$.

```python
def backtrack(assignment, csp):
    if len(assignment) == len(csp.variables):
        return assignment
    var = select_unassigned_variable(assignment, csp)
    for value in order_domain_values(var, assignment, csp):
        if consistent(var, value, assignment, csp):
            assignment[var] = value
            result = backtrack(assignment, csp)
            if result is not None:
                return result
            del assignment[var]          # undo and try the next value
    return None
```

[N-queens](/citadel/algorithms/NQueens) and [Sudoku](/citadel/algorithms/SudokuSolver) are the classic runnable instances of exactly this loop.

## Making backtracking fast

Plain backtracking is often far too slow. Four refinements, each attacking a different decision in that loop:

**Which variable next? — Minimum Remaining Values (MRV).** Pick the unassigned variable with the fewest legal values left. It's the "fail fast" heuristic: if a variable has only one option, deal with it now rather than discover the contradiction after fifty more assignments.

**Tie-break with the degree heuristic.** Among MRV ties, pick the variable involved in the most constraints on other unassigned variables — it prunes the most.

**Which value first? — Least Constraining Value.** For the chosen variable, try the value that rules out the fewest options for its neighbours, leaving maximum room for the rest of the search.

**Detect failure early — forward checking.** When you assign $X$, look at each unassigned neighbour $Y$ and delete from $Y$'s domain any value inconsistent with $X$'s. If a domain goes empty, backtrack immediately instead of continuing.

**Propagate further — arc consistency.** Forward checking only checks neighbours of the just-assigned variable. Arc consistency does more: an arc $X \rightarrow Y$ is consistent if every value in $X$'s domain has *some* compatible value in $Y$'s. The **AC-3** algorithm enforces this across the whole graph, re-queuing an arc's neighbours whenever a domain shrinks. Run as a preprocessor or after each assignment, it catches contradictions that forward checking misses, and it can solve large instances — even $n$-queens for big $n$ — with little backtracking.

## Adversarial search

Games break two assumptions of ordinary search. There's an **opponent** whose moves you don't control, so you need a *strategy* — a move for every reply they might make — not just a path. And there's a **time limit**: you usually can't search to the end of the game, so you approximate.

A two-player game is a search problem with: an **initial state** (board plus whose turn it is), **actions** (legal moves), a **terminal test** (is the game over?), and a **utility function** giving the numeric outcome from one player's view (+1 win, 0 draw, −1 loss in a zero-sum game). One player, **MAX**, maximises utility; the other, **MIN**, minimises it.

### Minimax

**Minimax** computes optimal play against an optimal opponent. Build the game tree to the terminal states, score them with the utility function, then back values up: a MAX node takes the maximum of its children's values, a MIN node the minimum. MAX then plays toward the highest-valued child.

It's complete on a finite tree and optimal against an optimal opponent. Time is $O(b^m)$ and space $O(bm)$ for the depth-first version. For chess ($b \approx 35$, $m \approx 100$) that's hopeless — exact minimax is out of reach.

### Alpha-beta pruning

Minimax explores branches no rational player would ever enter. **Alpha-beta pruning** skips them without changing the answer. Carry two values down the search:

- $\alpha$ — the best (highest) value MAX can already guarantee on the path so far.
- $\beta$ — the best (lowest) value MIN can already guarantee.

Prune whenever $\alpha \ge \beta$: at a MIN node whose value has dropped to $\le \alpha$, MAX would never let play reach here, so the remaining children don't matter; symmetrically at a MAX node that reaches $\ge \beta$.

Alpha-beta returns exactly the minimax move. Its payoff depends entirely on **move ordering** — explore likely-best moves first and you prune more. With perfect ordering the time complexity drops to $O(b^{m/2})$, which doubles the depth you can search in the same time.

Even so, real games are too big for a full search. Practical engines add a **cutoff test** (stop at depth $d$) and an **evaluation function** that estimates a position's utility without searching further — for chess, a weighted sum of material, mobility, king safety, pawn structure. Deep Blue (chess) and Chinook (checkers) were built on exactly this: alpha-beta, a depth cutoff, and a hand-tuned evaluation.

## The one idea to keep

Both halves of this post win by exploiting structure that a black-box search can't see. A CSP exposes its variables and constraints, so MRV and constraint propagation can prove a branch hopeless before exploring it. A game tree exposes the alternation of MAX and MIN, so alpha-beta can prove a branch irrelevant to the root decision. In both cases the algorithm is still depth-first search — it's just search that knows enough to look away.
