---
title: Grammars and Parsing - From CFGs to Probabilistic CKY
description: Recovering a sentence's grammatical structure - context-free grammars and where they sit in the Chomsky hierarchy, the classical top-down and shift-reduce parsers and their failure modes, treebanks and parser evaluation, and probabilistic context-free grammars parsed with the CKY dynamic-programming algorithm.
date: 2023-03-29
draft: false
slug: /artificial-intelligence/grammar
tags:
  - Artificial Intelligence
  - NLP
  - Probabilistic Models
---

"The old man the boats." It reads like a fragment until you notice "man" is the verb and "the old" is the subject — then it parses. **Parsing** is recovering that structure: analysing a string of words against a formal grammar to find how they group into phrases. This post covers the grammars, the classical parsing algorithms and why they struggle, and the probabilistic approach that handles the ambiguity real language throws at them.

## Parsing

Syntactic parsing produces a **parse tree** for a sentence. The structure feeds information extraction (who did what to whom), machine translation (getting the target grammar right), question answering, and sentiment analysis. There are two ways to represent it — **constituency** (nested phrases: NP, VP, PP) and **dependency** (directed head-to-dependent links), both introduced in [natural language processing](/citadel/artificial-intelligence/nlp). This post is about constituency parsing.

## Context-free grammars

A **generative grammar** is a finite set of rules that defines exactly the grammatical sentences of a language. The standard one for phrase structure is the **context-free grammar** (CFG), a 4-tuple $G = (T, N, S, R)$:

- $T$ — **terminal symbols**: the words ("cat", "chased", "the").
- $N$ — **non-terminal symbols**: syntactic categories (`S`, `NP`, `VP`).
- $S \in N$ — the **start symbol**.
- $R$ — **rules** $X \rightarrow \gamma$, with $X$ a non-terminal and $\gamma$ a string of terminals and non-terminals.

```text
S  → NP VP
NP → DT N
VP → V NP
DT → the        N → cat        V → chased
```

These generate "the cat chased the cat" and assign it a tree. It's common to separate **grammatical rules** (expanding non-terminals) from **lexical rules** (expanding a category to a word).

## Regular versus context-free

Grammars sit in the **Chomsky hierarchy** (covered from the theory side in [regular languages](/citadel/automata/regular-languages) and [context-free languages](/citadel/automata/context-free-grammar)). **Regular grammars** — equivalent to finite automata and regular expressions — handle tokenisation, morphology, and simple pattern matching, but can't represent nested structure or long-range dependencies ("The mouse the cat chased ran"). **CFGs** can describe nesting (an NP containing a PP containing an NP), which covers the phrase structure of most natural language. Even CFGs have limits: cross-serial dependencies (Swiss German) exceed context-free power, and enforcing agreement (subject-verb number) needs features layered on top of the bare rules.

## Classical parsing algorithms

**Top-down with backtracking.** Start from `S` and expand the leftmost non-terminal using grammar rules, matching terminals against the input as they appear; on a dead end, backtrack to the last choice point and try another rule. Problems: **left recursion** (`NP → NP PP`) sends it into an infinite loop; it repeats work on shared sub-problems; and with an ambiguous grammar it either stops at the first parse or slowly explores them all.

**Shift-reduce (bottom-up).** Keep a **stack** of processed symbols and an **input buffer** of remaining words. Two moves: **shift** the next word onto the stack, or **reduce** — when the stack's top symbols match a rule's right-hand side, replace them with its left-hand side. **Accept** when the stack holds only `S` and the buffer is empty. Ambiguity shows up as **conflicts**: a *shift/reduce* conflict (shift another word or reduce now?) and a *reduce/reduce* conflict (the stack top matches two rules). This is the same machinery a compiler's [LR parser](/citadel/compiler-design/syntax-analysis) uses, driven by precomputed tables.

## Treebanks

A **treebank** is a corpus where every sentence is annotated with its parse tree. The **Penn Treebank** — parsed Wall Street Journal text — is the standard. Treebanks provide the supervised training data for statistical parsers, the gold standard for evaluating them, empirical rule and lexical frequencies for probabilistic parsing, and broad coverage of real-world sentence types.

## Evaluating a parser

Compare a parser's tree to the gold tree, counting **constituents** (labelled brackets like `(NP ...)`):

- **Labelled precision** — $\dfrac{\text{correct constituents in output}}{\text{total constituents in output}}$
- **Labelled recall** — $\dfrac{\text{correct constituents in output}}{\text{total constituents in gold tree}}$
- **Labelled F1** — $\dfrac{2 \cdot LP \cdot LR}{LP + LR}$

POS tagging accuracy is usually reported alongside, since the pre-terminal nodes are part of the tree.

## Probabilistic parsing

Real sentences have many parses — "Fed raises interest rates 0.5% in effort to control inflation" had 36 under a minimal grammar. A **probabilistic context-free grammar** (PCFG) resolves the choice by attaching a probability $P(\beta \mid A)$ to each rule $A \rightarrow \beta$, with $\sum_\beta P(A \rightarrow \beta) = 1$ for every non-terminal $A$.

The probability of a parse tree is the product of the probabilities of the rules used to build it:

$$P(t) = \prod_{(A \rightarrow \beta)\, \in\, t} P(A \rightarrow \beta)$$

and the probability of a sentence is the sum over all its parses, $P(s) = \sum_{t:\ \text{yield}(t) = s} P(t)$. The parser's job is the most probable parse, $t^* = \arg\max_t P(t \mid s)$, which — since a tree either yields $s$ or has probability zero for it — reduces to $\arg\max_t P(t)$.

## The CKY algorithm

**CKY** (Cocke-Kasami-Younger) is dynamic programming for parsing with a (P)CFG. It needs the grammar in **Chomsky Normal Form** — every rule is $A \rightarrow B\,C$ (two non-terminals) or $A \rightarrow w$ (one terminal).

Build a chart indexed by span: `chart[i][j]` holds the non-terminals that can span words $i$ to $j$ (for a PCFG, each with its best probability and a backpointer).

- **Initialise** the width-1 cells from the lexical rules.
- **Fill** cells by increasing span width: for each span $[i, j)$, try every split point $k$ and every binary rule $A \rightarrow B\,C$ where $B$ spans $[i, k)$ and $C$ spans $[k, j)$, adding $A$ (and, for a PCFG, keeping the split that maximises $P(A \rightarrow B\,C)\cdot P(B\text{-subtree})\cdot P(C\text{-subtree})$).
- The sentence parses iff the start symbol is in `chart[0][N]`; backpointers reconstruct the tree.

As a recogniser (membership test, no probabilities):

```python
def cky(words, grammar):
    # grammar[(B, C)] = set of A for rules A -> B C
    # grammar[w]      = set of A for rules A -> w
    n = len(words)
    chart = [[set() for _ in range(n + 1)] for _ in range(n + 1)]
    for i, w in enumerate(words):
        chart[i][i + 1] |= grammar.get(w, set())
    for span in range(2, n + 1):
        for i in range(n - span + 1):
            j = i + span
            for k in range(i + 1, j):
                for B in chart[i][k]:
                    for C in chart[k][j]:
                        chart[i][j] |= grammar.get((B, C), set())
    return "S" in chart[0][n]

grammar = {
    "the": {"DT"}, "dog": {"NN"}, "cat": {"NN"}, "chased": {"VBD"},
    ("DT", "NN"):  {"NP"},
    ("VBD", "NP"): {"VP"},
    ("NP", "VP"):  {"S"},
}
print(cky(["the", "dog", "chased", "the", "cat"], grammar))   # True
print(cky(["the", "dog", "chased"], grammar))                 # False
```

The three nested loops over start, split, and span give CKY its time complexity of $O(N^3 \cdot |G|)$ for a sentence of $N$ words and a grammar of $|G|$ rules.

## The one idea to keep

A CFG says which structures are legal; a PCFG adds which are *likely*, so a parser can pick one out of dozens. The classical parsers search for a legal tree and get lost in ambiguity and left recursion; CKY sidesteps both by filling a table bottom-up — every substring's possible categories computed once — and reading the answer, and its best-probability variant, straight off the chart.
