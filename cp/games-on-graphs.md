---
title: Games on Arbitrary Graphs - Retrograde Win/Lose/Draw Analysis
description: Deciding the outcome of a two-player game whose position graph may contain cycles, by a backward breadth-first search from the terminal positions that labels every state WIN, LOSE, or DRAW.
date: 2024-10-31
draft: false
slug: /cp/games-on-graphs
tags:
  - Competitive Programming
  - Miscellaneous
  - Game Theory
---

[Sprague-Grundy](/citadel/cp/sprague-grundy-nim) assumes the game always ends: the position graph is a DAG. When moves can return to an earlier position — a token on a general directed graph, a pursuit game — cycles appear, and with them a third outcome: a **draw**, where the player who would lose can instead play forever. Retrograde analysis handles all three outcomes in $O(V + E)$.

## The problem

A game state is a vertex; a legal move is a directed edge to the next state (the mover chooses the edge). Under **normal play**, a player who cannot move loses. Classify every vertex as:

- **WIN** — the player to move can force a win;
- **LOSE** — the player to move loses against optimal defence;
- **DRAW** — with optimal play from both sides the game never ends.

Example: a single token on the cycle $0 \to 1 \to 2 \to 0$ with an extra edge $1 \to 3$ and $3$ a dead end. From $3$: LOSE (no move). From $1$: WIN (move to $3$). From $2$: its only move is to $0$; from $0$: its only move is to $1$ (WIN), so $0$ is LOSE... wait $0 \to 1$ and $1$ is WIN, so every move from $0$ leads to WIN, so $0$ is LOSE. Then $2 \to 0$ (LOSE) makes $2$ WIN.

## The idea: label backward from the terminals

Work in the **reverse** graph. A vertex's label follows two rules:

- it is **LOSE** if it has no moves, **or** if *every* move leads to a WIN vertex;
- it is **WIN** if *some* move leads to a LOSE vertex;
- otherwise (every move leads to WIN or DRAW, and at least one leads to DRAW) it is **DRAW**.

Process this as a BFS. Seed the queue with all terminal (out-degree $0$) vertices as LOSE. When a vertex $v$ is finalised, scan its predecessors $u$:

- if $v$ is LOSE, then $u$ has a move to a losing position → mark $u$ **WIN** and enqueue it;
- if $v$ is WIN, decrement $u$'s counter of "unresolved moves"; if it hits $0$, every move from $u$ leads to WIN → mark $u$ **LOSE** and enqueue it.

Anything still unlabelled when the queue drains is **DRAW** — it never got enough losing evidence to be WIN, nor did all its moves resolve to WIN.

## Algorithm

```python
from collections import deque

WIN, LOSE, DRAW = 1, -1, 0

def retrograde(n, moves):
    # moves[v] = list of states reachable from v in one move
    radj = [[] for _ in range(n)]
    degree = [0] * n
    for v in range(n):
        degree[v] = len(moves[v])
        for w in moves[v]:
            radj[w].append(v)

    result = [DRAW] * n
    remaining = degree[:]                 # unresolved moves per vertex
    q = deque()
    for v in range(n):
        if degree[v] == 0:
            result[v] = LOSE
            q.append(v)

    while q:
        v = q.popleft()
        for u in radj[v]:
            if result[u] != DRAW:
                continue                 # already finalised
            if result[v] == LOSE:
                result[u] = WIN
                q.append(u)
            else:                        # result[v] == WIN
                remaining[u] -= 1
                if remaining[u] == 0:
                    result[u] = LOSE
                    q.append(u)
    return result                        # leftover DRAW stays DRAW
```

## Handling "who moves" and scoring variants

- **Positions encode the player.** If the two players have different move sets, make a vertex $(\text{state}, \text{turn})$ and let `moves` respect whose turn it is. The analysis is unchanged.
- **Misère play** (last to move *loses*): flip the terminal seeding — out-degree-$0$ vertices are WIN — and swap the two rules.
- **Draw by repetition only for one side:** the DRAW label already captures "the player who cannot win can avoid losing by cycling".
- **Shortest win / longest survival:** the BFS layer at which a vertex is labelled is the game length under optimal play; store it alongside `result`.

## Complexity

- **Time:** $O(V + E)$ — each edge is examined once from the reverse side.
- **Space:** $O(V + E)$.

## Common pitfalls

- **Not using a per-vertex counter.** "Every move leads to WIN" must be detected incrementally with `remaining[u]`; rescanning all of $u$'s moves each time a successor resolves is $O(VE)$.
- **Overwriting a finalised label.** Once a vertex is WIN or LOSE it is fixed; guard with `if result[u] != DRAW`. A late LOSE successor must not flip an already-WIN vertex (it was already correct).
- **Forgetting DRAW is the default.** Initialise everything to DRAW; the algorithm only ever *promotes* to WIN/LOSE. Vertices on a cycle with no escape to a LOSE position stay DRAW correctly.
- **Terminal definition.** Under normal play, "no moves" is a LOSE for the player to move. Seeding it as WIN (or skipping it) breaks everything downstream.
- **Building the forward graph only.** The propagation is over predecessors — you need the reverse adjacency.

## The keystone

Label terminals LOSE, then BFS backward: a predecessor becomes WIN the moment one successor is LOSE, and LOSE only once *all* its successors are WIN (tracked by a countdown). Whatever the sweep never labels is a DRAW — a position from which the non-winning side escapes into an infinite loop. $O(V + E)$ for the whole game.
