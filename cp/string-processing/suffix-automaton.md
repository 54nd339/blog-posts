---
title: Suffix Automaton - The Smallest Automaton of All Substrings
description: The minimal DFA that accepts exactly the substrings of a string, built online in O(n) with the clone-on-split rule, and the endpos-tree it induces for counting occurrences and distinct substrings.
date: 2024-05-19
draft: false
slug: /cp/suffix-automaton
tags:
  - Competitive Programming
  - String Processing
  - Suffix Structures
---

A suffix automaton for $s$ is the smallest deterministic automaton that recognises exactly the set of substrings of $s$. It has at most $2n - 1$ states and $3n - 4$ transitions, builds **online** in $O(n)$ (amortised, for a fixed alphabet), and is the cleanest tool for "number of distinct substrings", "how many times does substring $w$ occur", and "longest common substring of several strings".

## The problem

Build a structure over $s$ that, in $O(|w|)$, tells you whether $w$ is a substring and — with a linear post-pass — how many times each substring occurs and how many distinct substrings there are.

Example: $s = \texttt{"abcbc"}$ has $\binom{6}{2} = 15$ non-empty substrings by position, but only $12$ distinct ones (`bc` appears twice). The automaton reports $12$.

## The idea

Group substrings by their **endpos set** — the set of ending positions where the substring occurs. Two substrings with the same endpos set behave identically for future matching, so they share one state. Within a state, the substrings form a contiguous range of lengths $[\text{len}(\text{link}) + 1,\ \text{len}]$, where `link` (the **suffix link**) points to the state of the next-shorter endpos class (a strict superset endpos).

Building online, appending character $c$:

1. Create a new state `cur` with `len = len(last) + 1`.
2. Walk suffix links from `last`; for each state `p` with no `c`-transition, add `p --c--> cur`.
3. If you reach the root, `link(cur) = root`.
4. Else let `q = trans(p, c)`. If `len(q) == len(p) + 1`, `link(cur) = q`.
5. Otherwise **clone** `q` into `clone` (same transitions and link, `len = len(p) + 1`), redirect the `c`-transitions on the suffix-link walk from `p` that pointed to `q` so they point to `clone`, and set `link(q) = link(cur) = clone`.
6. `last = cur`.

The clone step is what keeps the automaton minimal; each character adds $O(1)$ amortised states and transitions.

## How it works

Build `"aba"`:

- append `a`: state 1, `len 1`, link → root. `last = 1`.
- append `b`: state 2, `len 2`. From `1`, no `b` → `1 --b--> 2`. From root, no `b` → `root --b--> 2`. Reached root → `link(2) = root`. `last = 2`.
- append `a`: state 3, `len 3`. From `2`, no `a` → `2 --a--> 3`. From root, `a` exists → `q = 1`. `len(1) == len(root) + 1 == 1` → `link(3) = 1`. `last = 3`.

Distinct substrings = $\sum_v \big(\text{len}(v) - \text{len}(\text{link}(v))\big)$ over all non-root states $= (1-0) + (2-0) + (3-1) = 5$: `a, b, ab, ba, aba`. Correct.

## Algorithm

```python
class SuffixAutomaton:
    def __init__(self):
        self.nxt = [dict()]       # transitions
        self.link = [-1]
        self.length = [0]
        self.last = 0
        self.cnt = [0]            # occurrence base counts (set at creation of non-clones)

    def extend(self, c: str) -> None:
        cur = len(self.nxt)
        self.nxt.append(dict()); self.link.append(-1)
        self.length.append(self.length[self.last] + 1)
        self.cnt.append(1)
        p = self.last
        while p != -1 and c not in self.nxt[p]:
            self.nxt[p][c] = cur
            p = self.link[p]
        if p == -1:
            self.link[cur] = 0
        else:
            q = self.nxt[p][c]
            if self.length[p] + 1 == self.length[q]:
                self.link[cur] = q
            else:
                clone = len(self.nxt)
                self.nxt.append(dict(self.nxt[q]))
                self.link.append(self.link[q])
                self.length.append(self.length[p] + 1)
                self.cnt.append(0)                      # clones carry no base count
                while p != -1 and self.nxt[p].get(c) == q:
                    self.nxt[p][c] = clone
                    p = self.link[p]
                self.link[q] = self.link[cur] = clone
        self.last = cur

    def build(self, s: str) -> "SuffixAutomaton":
        for ch in s:
            self.extend(ch)
        return self

    def distinct_substrings(self) -> int:
        return sum(self.length[v] - self.length[self.link[v]] for v in range(1, len(self.nxt)))

    def occurrences(self) -> list[int]:
        # propagate cnt up the suffix-link tree, longest states first
        order = sorted(range(1, len(self.nxt)), key=lambda v: self.length[v], reverse=True)
        for v in order:
            if self.link[v] > 0:
                self.cnt[self.link[v]] += self.cnt[v]
        return self.cnt          # cnt[state] = number of occurrences of any substring in that state
```

## What it answers

- **Is $w$ a substring:** follow transitions from state $0$; success iff you consume all of $w$. $O(|w|)$.
- **Distinct substrings:** $\sum_v \text{len}(v) - \text{len}(\text{link}(v))$.
- **Occurrences of $w$:** locate $w$'s state, return its propagated `cnt`.
- **$k$-th smallest substring:** DP the number of paths from each state (a substring per path), then descend.
- **Longest common substring of $s$ and $t$:** build the automaton of $s$, run $t$ through it tracking current matched length, resetting via suffix links on a mismatch; the max length seen is the answer. Extends to several strings.
- **Total length of all distinct substrings:** $\sum_v \binom{\text{len}(v)+1}{2} - \binom{\text{len}(\text{link}(v))+1}{2}$-style sum.

## Complexity

- **Build:** $O(n)$ for a constant alphabet with array transitions; $O(n \log |\Sigma|)$ with `dict`/`map` transitions (Python default).
- **States:** $\le 2n - 1$; **transitions:** $\le 3n - 4$.

## Common pitfalls

- **Clone gets `cnt = 0`.** Only the states created directly on `extend` (the `cur` for each real prefix) carry a base occurrence count of $1$. Cloned states must start at $0$, or occurrence counts inflate.
- **Propagation order.** Add `cnt` up the suffix-link tree from **longest to shortest** `len`; a topological order by `len` descending is enough (no full tree DFS needed).
- **`len(q) == len(p) + 1` check.** This decides clone-or-not. Getting the comparison wrong builds a non-minimal or incorrect automaton.
- **Suffix-link redirection loop.** In the clone step, keep redirecting `p --c--> q` to `clone` while walking links, **only** while `nxt[p][c] == q`.
- **Root link.** `link[0] = -1`; treat state $0$ as the empty string.

## The keystone

The suffix automaton merges substrings with identical endpos sets into one state, giving the minimal substring-recognising DFA in $\le 2n$ states, built online with a clone-on-split rule that keeps it minimal. The suffix-link tree it induces turns "count occurrences" and "count distinct substrings" into one linear pass.
