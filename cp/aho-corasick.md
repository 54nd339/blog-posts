---
title: Aho-Corasick - Matching Many Patterns at Once
description: A trie of all patterns plus KMP-style failure links, turning "find every occurrence of every dictionary word in a text" into one linear pass, and a DP automaton for forbidden-substring problems.
date: 2024-05-16
draft: false
slug: /cp/aho-corasick
tags:
  - Competitive Programming
  - String Processing
  - String Matching
---

[KMP](/citadel/cp/prefix-function-kmp) matches one pattern in linear time by following a failure link on a mismatch. Aho-Corasick does the same for a whole **set** of patterns at once: build a trie of the patterns, add a failure link from every node to the longest proper suffix that is also a trie node, and run the text through it. Total time is $O(\text{text} + \sum |\text{pattern}|)$ plus the number of matches.

## The problem

Given a dictionary of patterns $P_1, \dots, P_k$ and a text $T$, report every $(i, j)$ such that $P_j$ occurs in $T$ ending at position $i$. Or just count total occurrences, or the number of distinct patterns that appear.

Example: dictionary `{"he", "she", "his", "hers"}`, text `"ahishers"` (indices `a0 h1 i2 s3 h4 e5 r6 s7`) → `his` ends at 3, `she` ends at 5, `he` ends at 5, `hers` ends at 7. All four dictionary words except that `he` and `she` share the same end position.

## The idea

- **Trie.** Insert every pattern; mark end-of-pattern nodes. Node = "we have matched this string so far".
- **Failure link `fail(v)`.** The trie node for the longest proper suffix of `v`'s string that is itself a trie node (the root if none). Exactly KMP's fallback, generalised.
- **Goto / automaton `go(v, c)`.** The node reached by reading character `c` in state `v`: the child if it exists, else `go(fail(v), c)`. Precomputing this over the alphabet gives a true DFA — $O(1)$ per text character.
- **Output link `out(v)`.** The nearest node on the fail chain that is a pattern end — lets you enumerate all patterns ending at the current position without walking the whole chain each time.

Build `fail` and `go` with a BFS over the trie (parents before children).

## How it works

Dictionary `{he, she, his, hers}`. Trie paths: `h-e`, `s-h-e`, `h-i-s`, `h-e-r-s`. `fail` of the node for `"she"` points to the node for `"he"` (longest proper suffix that is a trie node), so its output list inherits `he`. Reading `"ahishers"`: at index 3 we are at the `his` end-node → report `his`. At index 5 we are at the `she` end-node, whose output link chains to `he` → report both `she` and `he`. At index 7 we are at the `hers` end-node → report `hers`.

## Algorithm

```python
from collections import deque

class AhoCorasick:
    def __init__(self, patterns: list[str]):
        self.next = [{}]                      # trie transitions (child map per node)
        self.fail = [0]
        self.out = [[]]                      # pattern indices ending exactly at this node
        for idx, p in enumerate(patterns):
            v = 0
            for ch in p:
                if ch not in self.next[v]:
                    self.next.append({})
                    self.fail.append(0)
                    self.out.append([])
                    self.next[v][ch] = len(self.next) - 1
                v = self.next[v][ch]
            self.out[v].append(idx)
        self._build()

    def _build(self):
        q = deque()
        for u in self.next[0].values():
            self.fail[u] = 0
            q.append(u)
        while q:
            v = q.popleft()
            for ch, u in self.next[v].items():
                f = self.fail[v]
                while f and ch not in self.next[f]:
                    f = self.fail[f]
                self.fail[u] = self.next[f].get(ch, 0)
                if self.fail[u] == u:                     # only when f == 0 and ch is a root child
                    self.fail[u] = 0
                self.out[u] = self.out[u] + self.out[self.fail[u]]   # inherit via output link
                q.append(u)

    def go(self, v: int, ch: str) -> int:
        while v and ch not in self.next[v]:
            v = self.fail[v]
        return self.next[v].get(ch, 0)

    def find_all(self, text: str):
        v = 0
        for i, ch in enumerate(text):
            v = self.go(v, ch)
            for idx in self.out[v]:
                yield (i, idx)                # pattern `idx` ends at text index i
```

For heavy use, precompute a full transition table `go[v][c]` for every alphabet character during the BFS, so matching is a flat array lookup per text char.

## The DP automaton

For problems like "count strings of length $L$ over an alphabet that contain **none** of the forbidden patterns", run a DP over `dp[step][node]` = number of ways to be in automaton state `node` after `step` characters, transitioning by `go`, and never entering a node whose `out` is non-empty. Matrix-exponentiate the transition for large $L$.

## Complexity

- **Build:** $O(\sum |P_j| \cdot |\Sigma|)$ with the full transition table, $O(\sum |P_j|)$ nodes.
- **Match:** $O(|T|)$ with the table (amortised $O(|T|)$ even with the lazy `go`), plus $O(\text{number of reported matches})$.
- **Space:** $O(\text{nodes} \cdot |\Sigma|)$ for the table, or $O(\text{nodes})$ with child maps.

## Common pitfalls

- **BFS order for `fail`.** Depth-1 nodes fail to the root; deeper nodes need their parent's `fail` already set — hence BFS, not DFS.
- **Output links vs walking `fail`.** Copying `out[fail[u]]` into `out[u]` (as above) makes enumeration $O(\text{matches})$; walking the raw `fail` chain per position can be $O(n\sqrt n)$ in the worst case.
- **`fail[u] == u` self-loop.** Guard against it (can arise from a bad root transition); force it to `0`.
- **Alphabet size in the table.** For $|\Sigma| = 26$ a table is fine; for Unicode, use child maps and the lazy `go`.
- **Reporting all occurrences vs distinct patterns.** If you only need "which patterns appear", mark a boolean per pattern and stop copying output lists.

## The keystone

Aho-Corasick is a trie of all patterns with a KMP failure link at every node, forming a DFA that consumes the text once. Output links let it list every pattern ending at each position in $O(1)$ amortised, and the automaton doubles as a DP state space for counting strings that avoid a set of substrings.
