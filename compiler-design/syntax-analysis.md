---
title: Syntax Analysis - Parsing Tokens Into Structure
description: The parser's job - checking a token stream against a context-free grammar and building a parse tree, with top-down parsing (recursive descent, LL(1), FIRST and FOLLOW) and bottom-up parsing (shift-reduce, the LR family, LALR), plus error recovery and how ambiguous grammars get resolved.
date: 2022-10-23
draft: false
slug: /compiler-design/syntax-analysis
tags:
  - Compilers
  - Parsing
---

The [scanner](/citadel/compiler-design/lexical-analysis) gives you `id + id * id` — a flat list of tokens. Whether that means `(id + id) * id` or `id + (id * id)` is a question about *structure*, and answering it is the job of the **parser** (syntax analyser), the second phase of the front end. The parser checks that the tokens form a grammatically valid program and builds a **parse tree** that records how.

This post covers the grammar the parser works from, the two families of parsing algorithm, what happens on a syntax error, and how ambiguous grammars are tamed.

## The grammar

A parser needs a formal description of the language's syntax: a **context-free grammar** (CFG), covered in depth in [context-free languages](/citadel/automata/context-free-grammar). It has:

- **Terminals** — the tokens from the scanner (`if`, `id`, `+`, `(`).
- **Non-terminals** — names for syntactic structures (`expression`, `statement`, `declaration`).
- A **start symbol** — the non-terminal for a whole program.
- **Productions** — rules `A -> α` rewriting a non-terminal as a string of terminals and non-terminals.

The standard grammar for arithmetic expressions:

```text
E -> E + T | T
T -> T * F | F
F -> ( E ) | id
```

`E` (expression), `T` (term), `F` (factor) are non-terminals; `+`, `*`, `(`, `)`, `id` are terminals. The parser's goal is to show the token stream can be derived from `E`, and to produce the tree of that derivation.

## Top-down parsing

**Top-down** parsing builds the tree from the root down, trying to find a leftmost derivation by expanding non-terminals until they match the input.

### Recursive descent

Write one procedure per non-terminal. The procedure for `A` looks at the current token, picks a production `A -> α`, and processes `α` left to right — calling the procedure for each non-terminal, matching each terminal against the input and advancing.

### Backtracking

If the procedure picks the wrong production, it may need to reset the input pointer and try another. Backtracking is simple but can take exponential time, so real top-down parsers avoid it.

### Predictive parsing: LL(1)

A **predictive** parser chooses the right production by looking ahead at the next token(s), no backtracking. **LL(k)** means it scans **L**eft to right, builds a **L**eftmost derivation, and uses **k** tokens of lookahead; **LL(1)** is the common case.

LL(1) needs the grammar to cooperate:

- **No left recursion.** `E -> E + T` sends recursive descent into an infinite loop. Rewrite it (here, to a right-recursive or iterative form).
- **No undistinguishable common prefixes.** If `S -> if E then S` and `S -> if E then S else S`, one lookahead token (`if`) cannot pick between them; left-factor the grammar.

An LL(1) parser is driven by a table built from two sets:

- **FIRST(α)** — the terminals that can begin a string derived from `α`.
- **FOLLOW(A)** — the terminals that can appear immediately after `A` in some derivation.

Predictive parsing runs in linear time, but only for the restricted class of LL(1) grammars.

## Bottom-up parsing

**Bottom-up** parsing builds the tree from the leaves up: start with the tokens and repeatedly *reduce* a substring matching a production's right-hand side to its left-hand side non-terminal, until only the start symbol remains.

### Shift-reduce

Keep a **stack** of grammar symbols and an input buffer. Four actions:

1. **Shift** — push the next input token onto the stack.
2. **Reduce** — when the top of the stack matches the right-hand side `α` of a production `A -> α` (`α` is the **handle**), pop `α` and push `A`. This builds a tree node.
3. **Accept** — stack holds only the start symbol, input is empty.
4. **Error** — no action applies.

The hard part is deciding, at each step, whether to shift or reduce, and by which production.

### LR parsers

**LR(k)** parsers scan **L**eft to right and build a **R**ightmost derivation in reverse, with `k` tokens of lookahead (usually 0 or 1). They are table-driven:

- The **ACTION** table gives shift / reduce / accept / error from the current state (stack top) and next token.
- The **GOTO** table gives the next state after a reduction.

The family trades power against table size:

| Parser | Power | Notes |
| --- | --- | --- |
| LR(0) | weakest | no lookahead; too weak for most real grammars |
| SLR(1) | more | uses FOLLOW sets to resolve reductions |
| LALR(1) | nearly LR(1) | much smaller tables; used by YACC and Bison |
| LR(1) | strongest with 1 lookahead | tables can be very large |

LR parsers run in linear time and handle a broader class of grammars than LL parsers — including left-recursive ones, which is why bottom-up tools dominate compiler construction.

## Error recovery

A parser should not quit on the first syntax error; it should report it clearly and keep going to find more. Common strategies:

- **Panic mode** — discard input tokens until a synchronising token (`;`, `}`) appears, then resume. Simple and the most widely used.
- **Phrase-level recovery** — make a local repair (insert a missing `;`, delete a stray `,`) that lets the parse continue.
- **Error productions** — add grammar rules for common mistakes so the parser can give a specific diagnostic.
- **Global correction** — compute the minimal edit to make the input parse. Theoretically clean, too costly in practice.

## Ambiguity

An **ambiguous grammar** yields more than one parse tree for some input. `E -> E + E | E * E | id` parses `id + id * id` as both `(id + id) * id` and `id + (id * id)`. A compiler needs one reading, so:

1. **Rewrite the grammar** to encode precedence and associativity. The stratified `E / T / F` grammar at the top of this post admits only the `id + (id * id)` tree and makes both operators left-associative.
2. **Declare precedence in the tool.** YACC and Bison let you annotate operator tokens with precedence and associativity, keeping the grammar compact while resolving the conflicts.
3. **The dangling-else problem.** With `stmt -> if expr then stmt` and `stmt -> if expr then stmt else stmt`, `if E1 then if E2 then S1 else S2` is ambiguous about which `if` the `else` binds to. The convention is "nearest unmatched `then`", which an LR parser gets right by preferring to shift the `else` rather than reduce.

## The one idea to keep

Parsing is where a context-free grammar stops being theory and becomes a table. Top-down parsers predict the next production from lookahead and need a well-behaved grammar; bottom-up LR parsers recognise a handle on the stack and accept almost anything a programming language needs, which is why the parser in a real compiler is nearly always generated LALR(1). The output — a tree that says what groups with what — is what every later phase reasons about.
