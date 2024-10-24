---
title: 2-SAT - Implication Graphs and Strongly Connected Components
description: Deciding a boolean formula in which every clause has exactly two literals, in linear time, by turning each clause into two implications and reading the answer off the strongly connected components of the resulting graph.
date: 2024-10-24
draft: false
slug: /cp/two-sat
tags:
  - Competitive Programming
  - Graphs
  - Boolean Satisfiability
---

General SAT is NP-complete, but when every clause has **at most two literals** it is solvable in linear time. The reduction is elegant: each clause $(a \lor b)$ is equivalent to the pair of implications $\lnot a \Rightarrow b$ and $\lnot b \Rightarrow a$. Build the graph of all those implications and the formula is satisfiable exactly when no variable shares a [strongly connected component](/citadel/cp/strongly-connected-components) with its own negation.

## The problem

Given boolean variables $x_0, \dots, x_{n-1}$ and a conjunction of clauses each of the form $(\ell_i \lor \ell_j)$ where each $\ell$ is a variable or its negation, decide satisfiability and produce a satisfying assignment if one exists.

Many constraints reduce to 2-SAT clauses:

- "exactly one of $x, y$" → $(x \lor y) \land (\lnot x \lor \lnot y)$.
- "$x$ implies $y$" → the single clause $(\lnot x \lor y)$.
- "force $x$ true" → $(x \lor x)$.
- placing objects with two choices each so that no two conflict (intervals on a line, flipping segments, 2-colouring with exceptions).

## The idea: the implication graph

Make $2n$ vertices, one for each literal: $x_i$ and $\lnot x_i$. For every clause $(a \lor b)$ add two directed edges:

$$\lnot a \to b \qquad \text{and} \qquad \lnot b \to a.$$

Both say "if one literal is false, the other must be true". Implication is transitive, so if there is a path $\ell \to \dots \to \lnot \ell$ **and** a path $\lnot \ell \to \dots \to \ell$, then $\ell$ forces $\lnot \ell$ and vice versa — a contradiction. That is exactly the statement "$\ell$ and $\lnot \ell$ are in the same SCC".

**Theorem.** The formula is satisfiable iff for every variable, $x_i$ and $\lnot x_i$ lie in different SCCs.

**Reading the assignment.** Condense the graph to its DAG of SCCs and take a topological order. For each variable, set the literal whose component comes **later** in topological order to true. (Later = "further from the sources" = "implies fewer things".) With Kosaraju's second pass, components are discovered in topological order, so "later" means the larger component id.

## Algorithm

```python
class TwoSat:
    def __init__(self, n):
        self.n = n
        self.g = [[] for _ in range(2 * n)]      # implication graph
        self.gt = [[] for _ in range(2 * n)]     # its transpose

    @staticmethod
    def _lit(var, val):                          # val=True -> 2*var, val=False -> 2*var+1
        return 2 * var + (0 if val else 1)

    def add_clause(self, x, xval, y, yval):      # (x == xval) OR (y == yval)
        a, b = self._lit(x, xval), self._lit(y, yval)
        self.g[a ^ 1].append(b);  self.gt[b].append(a ^ 1)   # not-a -> b
        self.g[b ^ 1].append(a);  self.gt[a].append(b ^ 1)   # not-b -> a

    def add_implication(self, x, xval, y, yval):  # (x == xval) -> (y == yval)
        self.add_clause(x, not xval, y, yval)

    def force(self, x, xval):                     # (x == xval) is required
        a = self._lit(x, xval)
        self.g[a ^ 1].append(a);  self.gt[a].append(a ^ 1)

    def solve(self):
        N = 2 * self.n
        visited = [False] * N
        order = []
        for s in range(N):                        # pass 1: finish order on g (iterative)
            if visited[s]:
                continue
            stack = [(s, 0)]; visited[s] = True
            while stack:
                v, i = stack.pop()
                if i < len(self.g[v]):
                    stack.append((v, i + 1))
                    w = self.g[v][i]
                    if not visited[w]:
                        visited[w] = True
                        stack.append((w, 0))
                else:
                    order.append(v)

        comp = [-1] * N
        c = 0
        for v in reversed(order):                 # pass 2: SCCs on the transpose
            if comp[v] != -1:
                continue
            stack = [v]; comp[v] = c
            while stack:
                u = stack.pop()
                for w in self.gt[u]:
                    if comp[w] == -1:
                        comp[w] = c; stack.append(w)
            c += 1

        assign = [False] * self.n
        for v in range(self.n):
            if comp[2 * v] == comp[2 * v + 1]:
                return None                       # unsatisfiable
            assign[v] = comp[2 * v] > comp[2 * v + 1]
        return assign
```

## Complexity

- **Time:** $O(n + m)$ for $n$ variables and $m$ clauses — building $2m$ edges and one SCC decomposition.
- **Space:** $O(n + m)$.

## Common pitfalls

- **Only one implication per clause.** $(a \lor b)$ needs *both* $\lnot a \to b$ and $\lnot b \to a$. Adding just one makes the graph asymmetric and the SCC test unsound.
- **Assignment sign convention.** "Set the literal in the later component" only if your SCC ids increase along topological order (Kosaraju's pass 2). Tarjan numbers components in *reverse* topological order, so with Tarjan you pick the **smaller** id. Verify against a brute enumeration once.
- **Recursion depth.** $2n$ can be $2 \times 10^6$; a recursive DFS overflows Python's stack. Both passes above are iterative.
- **Self-implications for forcing.** "$x$ must be true" is the edge $\lnot x \to x$ (equivalently the clause $(x \lor x)$), not deleting the $\lnot x$ vertex.
- **Literal encoding.** Keep a single convention — here `2*var` is the positive literal, `2*var+1` its negation, and `^ 1` flips. Mixing conventions between `add_clause` and `solve` is the classic bug.

## The keystone

Every 2-literal clause is a pair of implications; chaining them, a variable that forces its own negation and is forced back is unsatisfiable — precisely when the two literal-vertices share an SCC. Otherwise, choosing for each variable the literal whose component is later in topological order yields a consistent satisfying assignment, all in $O(n + m)$.
