---
title: Suffix Tree - Ukkonen's Linear-Time Construction
description: The compressed trie of every suffix, built left to right in O(n) with the active-point trio, suffix links, and the "rule 3 shows stop" skip, plus why a suffix automaton usually substitutes.
date: 2024-05-17
draft: false
slug: /cp/suffix-tree-ukkonen
tags:
  - Competitive Programming
  - String Processing
  - Suffix Structures
---

A suffix tree is the trie of all $n$ suffixes of a string with every non-branching path compressed into a single edge labelled by a substring. It has $\le 2n$ nodes and answers substring queries, longest repeated substring, and longest common substring directly. Ukkonen's algorithm builds it online in $O(n)$ — famously fiddly, which is why many contestants reach for a [suffix automaton](/citadel/cp/suffix-automaton) or [suffix array](/citadel/cp/suffix-array) instead.

## The problem

Build, for $s$ (append a unique sentinel `$`), a tree where:

- each edge is labelled by a substring of $s$ (stored as a `(start, end)` index pair, not a copy),
- each leaf corresponds to one suffix, its root-to-leaf path spelling that suffix,
- internal nodes are exactly the branching points — strings that are the longest common prefix of $\ge 2$ suffixes.

Example: `"xabxa$"` — the branching nodes are `""`, `"a"`, `"xa"`, and `"x"`... actually the distinct branch points are `""`, `"a"`, `"xa"`; the tree has $6$ leaves and $3$ internal nodes.

## The idea: the active point and three rules

Process prefixes $s[0 \dots i]$ for $i = 0 \dots n-1$. At step $i$ you must ensure all suffixes of $s[0 \dots i]$ are represented. Track:

- **`active_node`** — where the next insertion starts,
- **`active_edge`** — which child edge (by first character),
- **`active_length`** — how far along that edge,
- **`remainder`** — how many suffixes still need inserting this step.

For each new character $c = s[i]$:

- **Rule 1 (leaf extension) is automatic:** leaves store `end` as a shared "current position" pointer, so every leaf edge grows for free.
- **Rule 2 (split / new leaf):** if the active point cannot continue with $c$, split the edge there, add a new internal node, hang a new leaf for $c$, decrement `remainder`, and add a **suffix link** from the previous internal node created this step to the new one.
- **Rule 3 (already there — "show stop"):** if $c$ already continues from the active point, just advance `active_length` and **break** — everything shorter is already present too.

After a Rule 2 split, move the active point: if `active_node` is the root, decrement `active_length` and set `active_edge = s[i - remainder + 1]`; otherwise follow `active_node`'s suffix link (or to the root). The **skip/count trick** — jumping over whole edges when `active_length` exceeds an edge length — is what keeps the walk $O(n)$ total.

## Algorithm

```python
class SuffixTree:
    def __init__(self, s: str):
        self.s = s
        self.n = len(s)
        # node arrays
        self.start = [0]          # edge start into s (root: unused)
        self.end = [0]            # edge end (exclusive); leaves share self.leaf_end via -1 sentinel
        self.link = [0]           # suffix link
        self.child = [dict()]     # first-char -> child node
        self.leaf_end = 0         # global "current end" for all leaves
        self._build()

    def _new_node(self, start, end):
        self.start.append(start); self.end.append(end)
        self.link.append(0); self.child.append(dict())
        return len(self.start) - 1

    def _edge_len(self, v):
        e = self.leaf_end if self.end[v] == -1 else self.end[v]
        return e - self.start[v]

    def _build(self):
        root = 0
        active_node, active_edge, active_length = root, 0, 0
        remainder = 0
        last_new = 0
        for i in range(self.n):
            self.leaf_end = i + 1
            remainder += 1
            last_new = 0
            while remainder > 0:
                if active_length == 0:
                    active_edge = i
                ch = self.s[active_edge]
                if ch not in self.child[active_node]:
                    # Rule 2: new leaf directly off active_node
                    leaf = self._new_node(i, -1)
                    self.child[active_node][ch] = leaf
                    if last_new:
                        self.link[last_new] = active_node
                        last_new = 0
                else:
                    nxt = self.child[active_node][ch]
                    elen = self._edge_len(nxt)
                    if active_length >= elen:
                        # skip/count: walk past this whole edge
                        active_edge += elen
                        active_length -= elen
                        active_node = nxt
                        continue
                    if self.s[self.start[nxt] + active_length] == self.s[i]:
                        # Rule 3: character already present -> show stop
                        active_length += 1
                        if last_new:
                            self.link[last_new] = active_node
                        break
                    # Rule 2: split the edge
                    split = self._new_node(self.start[nxt], self.start[nxt] + active_length)
                    self.child[active_node][ch] = split
                    leaf = self._new_node(i, -1)
                    self.child[split][self.s[i]] = leaf
                    self.start[nxt] += active_length
                    self.child[split][self.s[self.start[nxt]]] = nxt
                    if last_new:
                        self.link[last_new] = split
                    last_new = split
                remainder -= 1
                if active_node == root and active_length > 0:
                    active_length -= 1
                    active_edge = i - remainder + 1
                elif active_node != root:
                    active_node = self.link[active_node]

    def distinct_substrings(self) -> int:
        total = 0
        stack = [0]
        while stack:
            v = stack.pop()
            if v != 0:
                total += self._edge_len(v)
            stack.extend(self.child[v].values())
        return total          # = number of distinct substrings of s (the sentinel-augmented string)
```

Call it on `text + "$"` with a sentinel not in `text`. `total_edge_len` then counts distinct substrings of `text + "$"`; the distinct substrings of `text` itself is that minus `len(text) + 1` (the substrings that contain `$`).

## What it answers

- **Substring test:** walk from the root matching characters; $O(|w|)$.
- **Longest repeated substring:** the deepest internal node (by total string depth).
- **Number of distinct substrings:** sum of all edge lengths (each edge contributes its new characters); with a `$` sentinel, subtract the sentinel-only contributions or count on `s` directly.
- **Longest common substring of two strings:** build over `s + "#" + t + "$"`, find the deepest internal node whose subtree contains leaves from both parts.
- **Matching statistics / all occurrences of a pattern:** the pattern's locus subtree; its leaves are the occurrence positions.

## Complexity

- **Build:** $O(n)$ for a constant alphabet (amortised — `remainder`, `active_length`, and node-depth each move $O(n)$ total).
- **Space:** $O(n)$ nodes; edges are `(start, end)` pairs, not substrings.

## Common pitfalls

- **Not using a sentinel.** Without a terminal character not in $s$, some suffixes end inside edges and there is no leaf for them — several queries break.
- **Leaf `end` handling.** Leaves must share a global "current position"; storing a fixed `end` on a leaf loses the automatic Rule 1 growth.
- **Suffix link wiring.** Exactly one pending `last_new` per step; link it to the node you create or land on *next*, then clear it.
- **Skip/count guard.** Use `active_length >= elen` (walk past) vs `<` (stop inside). Off by one here corrupts the active point.
- **Reaching for it at all.** For most contest tasks a suffix array + LCP or a suffix automaton is shorter and less bug-prone; use Ukkonen only when you specifically need the tree structure.

## The keystone

Ukkonen builds the suffix tree left to right, keeping an active point `(node, edge, length)` and a `remainder` count; leaves grow for free (Rule 1), a missing character splits an edge and adds a leaf with a suffix link (Rule 2), and a present character stops the round early (Rule 3). The skip/count trick bounds the walk at $O(n)$ — but the [suffix automaton](/citadel/cp/suffix-automaton) is the friendlier linear-size substring structure in practice.
