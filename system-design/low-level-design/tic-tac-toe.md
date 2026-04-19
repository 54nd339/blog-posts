---
title: LLD - Board Games (Tic-Tac-Toe, Snake and Ladder, Chess)
description: One skeleton - a game loop, players, a board, a move, a referee, a game state - stretches from tic-tac-toe to snake-and-ladder to chess. What changes is move generation and the win check; what stays is the turn manager and the pluggable player.
date: 2026-04-19
draft: false
slug: /system-design/tic-tac-toe
tags:
  - System Design
  - Low-Level Design
  - OOD
---

Interviewers ask for tic-tac-toe, then say "now make it chess." If your tic-tac-toe was a `char[3][3]` and a hard-coded win check, that's a rewrite. If it was a game loop over an abstract `Board`, `Player`, and `Rules`, chess is a bigger `Board`, a `Piece` hierarchy for move generation, and a different `Rules` — the loop doesn't move.

This is the shared skeleton and where each game plugs into it.

## The skeleton

```java
enum GameStatus { IN_PROGRESS, WIN, DRAW }

interface Player {
    String id();
    Move nextMove(Board board);        // human (from input) or AI (computed)
}

interface Board {
    void apply(Move m);
    boolean isValid(Move m);
    List<Move> legalMoves(Player p);
}

interface Rules {
    boolean isWin(Board b, Move lastMove, Player mover);
    boolean isDraw(Board b);
}

class Game {
    private final Board board;
    private final Rules rules;
    private final List<Player> players;   // order = turn order
    private int turn = 0;
    private GameStatus status = GameStatus.IN_PROGRESS;

    void play() {
        while (status == GameStatus.IN_PROGRESS) {
            Player p = players.get(turn % players.size());
            Move m = p.nextMove(board);
            if (!board.isValid(m)) { /* reprompt human, or reject */ continue; }
            board.apply(m);
            if (rules.isWin(board, m, p))      status = GameStatus.WIN;
            else if (rules.isDraw(board))      status = GameStatus.DRAW;
            else                               turn++;
        }
        announce(status, players.get(turn % players.size()));
    }
}
```

`Player` being an interface means human vs AI vs random are all just implementations — the loop never asks "is this a bot?".

## Tic-tac-toe

- **Board** — `Symbol[n][n]` (`X`, `O`, `EMPTY`). `legalMoves` = empty cells. `apply` places the mover's symbol.
- **Rules.isWin** — after a move at `(r, c)`, check only the row `r`, column `c`, and (if on a diagonal) the diagonals. Keep per-line counters (`rowCount[r] += mover == X ? 1 : -1`) so the check is O(1): a line wins when `|count| == n`.
- **Rules.isDraw** — board full, no win.

~50 lines on top of the skeleton.

## Snake and Ladder

- **Board** — a linear array of cells `1..100`; a `Map<Integer,Integer>` of jumps (snake head → tail, ladder bottom → top). No 2D grid.
- **Move** — carries a dice roll. `Dice` is a **strategy** (`SingleDie`, `TwoDice`, `LoadedDie` for testing).
- **apply** — advance the player's token by the roll; if the landing cell is a jump key, move to its value; overshooting 100 stays put (or bounces, per rules).
- **Rules.isWin** — a token reached exactly 100.
- **Players** hold a position; there's no board ownership of cells beyond the jump map.

Same loop, different `Board` and a `Dice` strategy feeding the `Move`.

## Chess

- **Board** — 8×8 of `Optional<Piece>`. `Piece` is a small hierarchy (`King`, `Queen`, `Rook`, `Bishop`, `Knight`, `Pawn`), each with `pseudoLegalMoves(Board, Square)` — the moves the piece *can* make ignoring check.
- **legalMoves** = pseudo-legal moves filtered to those that don't leave the mover's king in check (apply the move to a copy, test `isKingAttacked`).
- **Rules.isWin** — the opponent is in **checkmate**: in check and `legalMoves` is empty.
- **Rules.isDraw** — stalemate (not in check, no legal moves), plus threefold repetition, the fifty-move rule, insufficient material.
- **Special moves** — castling, en passant, promotion are extra `Move` subtypes handled in the relevant `Piece.pseudoLegalMoves` and in `apply`. Polymorphism on `Piece` keeps the giant "what can move where" switch out of the board class.

The loop is unchanged. All the complexity is in `Piece` move generation and the check-detection filter.

## AI players

An AI `Player` computes `nextMove` instead of reading input. For tic-tac-toe, minimax over `board.legalMoves` (the game tree is tiny). For chess, minimax with alpha-beta pruning and an evaluation function, depth-limited. The `Player` interface means the `Game` loop doesn't care — `new Game(board, rules, List.of(new HumanPlayer(), new MinimaxPlayer(depth=4)))`.

For a random opponent (useful in tests), `RandomPlayer` picks uniformly from `board.legalMoves(this)`.

## Concurrency

Local play is single-threaded — the loop alternates. For **online** play, the `Game` lives on a server; each `nextMove` comes from a network message, and a move-validation service checks `board.isValid` against the authoritative board before applying (never trust the client). One game per lock; a matchmaking service pairs players into games.

## Extensions

- **Larger tic-tac-toe / Connect Four / Gomoku** — `n` and the win-length become parameters of `Rules`.
- **More than two players** — the `players` list and `turn % size` already handle it; Snake and Ladder needs nothing changed.
- **Undo** — a `Command`-style move history; `undo()` reverses the last `apply`.
- **Timers** — a per-player clock; running out is a loss condition in `Rules`.

## The one idea to keep

Every turn-based board game is the same loop — ask the current `Player` for a `Move`, validate it against the `Board`, apply it, ask `Rules` if it's a win or draw, advance the turn. What varies is `Board` (grid vs line vs 8×8), move generation (empty cells vs a dice roll vs per-piece rules), and the win check. Keep `Player` an interface so human, AI, and network players share the loop, and push chess's move logic onto the `Piece` subclasses so the board class stays small.
