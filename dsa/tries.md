---
title: Tries - Sharing Prefixes to Search Faster
description: A guide to the Tries section of NeetCode 150 — a character-indexed tree gives O(length) insert and prefix queries, and drives branching search when patterns have wildcards or must match a grid.
date: 2021-10-25
draft: false
slug: /dsa/tries
tags:
  - Competitive Programming
  - Tries
  - Design
---

A trie (prefix tree) stores a set of strings so that shared prefixes are stored once. Each node has one child per possible next character and a flag for "a word ends here". Insert and exact lookup are `O(length)`, with no hashing of the whole string.

## When a trie earns its place

- You need **prefix** queries — "is any word starting with `app`?" — which a hash set cannot answer without checking every entry.
- You are matching **many** words against a text or grid, and they share prefixes.
- The query has **wildcards** or partial structure, so you need to branch mid-string.

If you only ever do exact `in` checks on whole strings, a hash set is simpler and just as fast.

## The three problems

- [Implement Trie](/citadel/dsa/implement-trie-prefix-tree) — the data structure itself: `insert`, `search` (path exists **and** ends in a word), `startsWith` (path exists). Children as a 26-array or a hash map.
- [Design Add and Search Words](/citadel/dsa/design-add-and-search-words-data-structure) — `search` patterns contain `.`; at a dot the walk fans out to every child and succeeds if any branch does. Trie + backtracking.
- [Word Search II](/citadel/dsa/word-search-ii) — build a trie of all target words, then one DFS over the board follows the trie, exploring a cell only while some word still has that prefix. Optionally prune found/exhausted branches.

## The core operations

```python
# insert
node = root
for c in word:
    node = node.children.setdefault(c, TrieNode())
node.end = True

# walk a prefix
node = root
for c in prefix:
    if c not in node.children:
        return None
    node = node.children[c]
return node
```

Everything else — wildcard search, grid matching — is this walk with branching added at choice points.

## Recognising it

- "prefix", "startsWith", "autocomplete" → trie.
- "search patterns with `.`" or "match against a dictionary" → trie + DFS.
- "find all these words in the grid/text" → one trie, one traversal.

## Where this goes next

The wildcard and grid variants are [Backtracking](/citadel/dsa/backtracking) with the trie acting as a feasibility oracle — it says, in $O(1)$ per step, whether the current partial string can still lead to a real word, which is exactly the pruning that makes backtracking fast.
