---
title: Sprague-Grundy Theorem and Nim - Summing Games With XOR
description: The complete theory of impartial games that end - Nim and the XOR-of-pile-sizes rule, the Grundy number as the mex of a position's options, and why the Grundy number of a sum of games is the XOR of the parts.
date: 2024-11-02
draft: false
slug: /cp/sprague-grundy-nim
tags:
  - Competitive Programming
  - Miscellaneous
  - Game Theory
---

An **impartial game** is one where both players have exactly the same moves from any position (so the only asymmetry is whose turn it is), it always ends, and the player who cannot move loses. The Sprague-Grundy theorem says every such game is equivalent to a single Nim pile, and a collection of independent games is equivalent to Nim on the corresponding piles. That reduces "who wins?" to one XOR.

## Nim

$k$ piles of stones; a move removes one or more stones from one pile; last to move wins.

**Bouton's theorem.** The position is a **losing position for the player to move** (a P-position) iff the XOR of all pile sizes is $0$.

Why: from an XOR of $0$, any move changes exactly one pile and so makes the XOR non-zero. From a non-zero XOR $x$, let the highest set bit of $x$ be bit $b$; some pile $p$ has bit $b$ set, and $p \oplus x < p$, so you can reduce that pile to $p \oplus x$, making the XOR $0$. So the winner keeps handing the loser an all-zero XOR until the loser faces $0, 0, \dots, 0$ and cannot move.

Example: piles $3, 4, 5$. $3 \oplus 4 \oplus 5 = 2 \ne 0$, so the player to move wins — reduce the pile of $3$ to $3 \oplus 2 = 1$, leaving $1, 4, 5$ with XOR $0$.

## Grundy numbers (the general impartial game)

For any impartial game position $v$, define the **Grundy number** (nimber)

$$g(v) = \operatorname{mex}\{\, g(w) : v \to w \text{ is a legal move} \,\},$$

where [mex](/citadel/cp/mex-of-an-array) is the smallest non-negative integer not in the set. A terminal position has no options, so $g = \operatorname{mex}\{\} = 0$.

**Sprague-Grundy theorem.** A position with Grundy number $g$ is *interchangeable with a Nim pile of size $g$*. In particular $g(v) = 0$ iff $v$ is a losing position for the player to move.

## Summing games

If a move consists of picking one of several independent sub-games and moving in it (the game ends when no sub-game has a move), then

$$g(\text{whole}) = g(\text{game}_1) \oplus g(\text{game}_2) \oplus \cdots \oplus g(\text{game}_k).$$

So: compute each component's Grundy number, XOR them, and the first player wins iff the result is non-zero. This is exactly Bouton's theorem when every component is a Nim pile (a pile of size $n$ has Grundy number $n$).

```python
def grundy_table(max_state, moves):
    # moves(s) -> iterable of states reachable from s; states are 0..max_state
    g = [0] * (max_state + 1)
    for s in range(1, max_state + 1):
        reachable = {g[t] for t in moves(s)}
        m = 0
        while m in reachable:
            m += 1
        g[s] = m
    return g

def first_player_wins(components, g):
    x = 0
    for s in components:
        x ^= g[s]
    return x != 0
```

### Worked example: the subtraction game

Move: remove $1$, $2$, or $3$ stones. `moves(s) = [s-1, s-2, s-3]` (clipped at $0$). The Grundy sequence is $0, 1, 2, 3, 0, 1, 2, 3, \dots$ — so $g(s) = s \bmod 4$, and a single pile is losing iff $s \equiv 0 \pmod 4$. For several such piles, XOR the $s_i \bmod 4$.

## Complexity

- **Grundy table:** $O(\sum_s |\text{moves}(s)|)$, i.e. linear in the state graph.
- **Deciding a sum:** $O(k)$ XORs once the per-component Grundy values are known.
- For families of games, the Grundy sequence is often eventually periodic — detect the period and answer in $O(1)$.

## Common pitfalls

- **Using the Grundy number as a "score".** Only whether it is zero tells you the winner. Its magnitude matters solely for XOR-ing with other games.
- **mex vs "smallest option".** $\operatorname{mex}$ is the smallest non-negative integer *absent* from the option set, not `min(options) - 1` or `max(options) + 1`.
- **Misère play.** The XOR rule is for *normal* play (last move wins). Misère Nim has its own rule (play as normal Nim unless every pile is size $\le 1$, then invert), and misère theory in general is much harder.
- **Non-independent components.** The XOR sum needs the sub-games to not interact — a move in one must not constrain another. "Remove from pile $i$ *and* pile $i+1$" is not a sum of Nim piles.
- **Partisan games.** Chess, Hackenbush with coloured edges — different moves for each player — are outside Sprague-Grundy; they need combinatorial-game-theory (surreal) values.

## The keystone

Every impartial finite game position has a Grundy number, the mex of its options' Grundy numbers, and it plays identically to a Nim pile of that size. Independent games add by XOR, so the first player wins exactly when the XOR of all component Grundy numbers is non-zero — Bouton's Nim rule, generalised to everything.
