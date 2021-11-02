---
title: Design Add and Search Words - A Trie That Branches on Dots
description: A word dictionary where search patterns may contain a dot wildcard — a trie handles exact letters directly and, on a dot, recurses into every child.
date: 2021-11-02
draft: false
slug: /dsa/design-add-and-search-words-data-structure
tags:
  - Competitive Programming
  - Tries
  - Backtracking
---

This is [Implement Trie](/citadel/dsa/implement-trie-prefix-tree) plus one rule: a `.` in the query matches any single character. At a `.`, instead of following one edge, the search fans out to all children and succeeds if any branch does.

## Description

Implement `WordDictionary` with:

- `addWord(word)` — add a word.
- `search(word)` — return `true` if any added word matches; `word` may contain `.`, each matching any one letter.

**Example**

```
addWord("bad"); addWord("dad"); addWord("mad")
search("pad") -> false
search("bad") -> true
search(".ad") -> true
search("b..") -> true
```

**Constraints**

- $1 \le \text{word.length} \le 25$
- `addWord` words are lowercase letters; `search` patterns are letters or `.`.
- At most `2` dots per `search` pattern (LeetCode's stated limit); up to $10^4$ calls.

## Prerequisites

- [Implement Trie](/citadel/dsa/implement-trie-prefix-tree).
- DFS with branching (backtracking) when a step has multiple options.

## Approach 1: Brute Force

### Intuition

Store the words in a list. To search, compare the pattern against every word of the right length, treating `.` as a match.

### Algorithm

1. `addWord`: append to a list.
2. `search`: for each stored word of equal length, check position by position — `pattern[i] == word[i]` or `pattern[i] == '.'`. Return `true` if any word matches.

```python
class WordDictionary:
    def __init__(self):
        self.words: list[str] = []

    def addWord(self, word: str) -> None:
        self.words.append(word)

    def search(self, word: str) -> bool:
        for w in self.words:
            if len(w) != len(word):
                continue
            if all(a == b or a == "." for a, b in zip(word, w)):
                return True
        return False
```

### Complexity

- **Time:** $O(1)$ for `addWord`; $O(m \cdot n)$ for `search` (`m` words, `n` length).
- **Space:** $O(m \cdot n)$.

## Approach 2: Trie with Wildcard DFS

### Intuition

Insert words into a trie. Search recursively: for a normal character, follow that one child; for `.`, try every child. At the end of the pattern, succeed only if the current node marks a word end.

### Algorithm

1. `addWord`: standard trie insert; mark `end` on the last node.
2. `search`: `dfs(node, i)` over pattern index `i`.
   - If `i == len(word)`, return `node.end`.
   - If `word[i] == '.'`, return `True` if `dfs(child, i + 1)` succeeds for *any* child.
   - Else, if `word[i]` is a child, return `dfs(that child, i + 1)`; otherwise `False`.

```python
class TrieNode:
    def __init__(self):
        self.children: dict[str, "TrieNode"] = {}
        self.end = False

class WordDictionary:
    def __init__(self):
        self.root = TrieNode()

    def addWord(self, word: str) -> None:
        node = self.root
        for c in word:
            node = node.children.setdefault(c, TrieNode())
        node.end = True

    def search(self, word: str) -> bool:
        def dfs(node: TrieNode, i: int) -> bool:
            if i == len(word):
                return node.end
            c = word[i]
            if c == ".":
                return any(dfs(child, i + 1) for child in node.children.values())
            return c in node.children and dfs(node.children[c], i + 1)

        return dfs(self.root, 0)
```

### Complexity

- **Time:** $O(n)$ for a dot-free pattern; up to $O(26^d \cdot n)$ with `d` dots (bounded because `d` is small).
- **Space:** $O(t + n)$ — trie nodes plus recursion depth.

## Common Pitfalls

- **Terminating the search at any node, not a word-end node.** `i == len(word)` must return `node.end`, or `search("ba")` matches after inserting only `"bad"`.
- **On `.`, returning after the first child regardless of result.** You must try each child and succeed if *any* recursion returns `True` (`any(...)`), i.e. backtrack on failure.
- **Not guarding `c in node.children` for a normal character.** Indexing a missing child raises instead of returning `False`.

## The keystone

A wildcard converts a deterministic trie walk into a branching search — the same "try all options at a choice point, backtrack on failure" move as [Backtracking](/citadel/dsa/backtracking). The trie keeps each branch cheap by sharing prefixes, so the fan-out only happens where the pattern is genuinely ambiguous.
