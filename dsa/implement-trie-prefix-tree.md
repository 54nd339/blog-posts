---
title: Implement Trie - A Tree Keyed by Characters
description: Building a prefix tree with insert, search, and startsWith — each node holds one child per next letter and an end-of-word flag, backed by a fixed array or a hash map.
date: 2021-10-27
draft: false
slug: /dsa/implement-trie-prefix-tree
tags:
  - Competitive Programming
  - Tries
  - Design
---

A trie stores a set of strings as a tree where each edge is a character and each root-to-node path spells a prefix. Insert and lookup cost `O(length of the key)` with no hashing of the whole string and no collisions.

## Description

Implement `Trie` with:

- `insert(word)` — add `word` to the trie.
- `search(word)` — return `true` if `word` was inserted.
- `startsWith(prefix)` — return `true` if any inserted word begins with `prefix`.

**Example**

```
insert("apple"); search("apple") -> true; search("app") -> false
startsWith("app") -> true; insert("app"); search("app") -> true
```

**Constraints**

- $1 \le \text{word.length}, \text{prefix.length} \le 2000$
- Lowercase English letters only.
- Up to $3 \times 10^4$ calls.

## Prerequisites

- Tree nodes with multiple children.
- The difference between "a prefix path exists" and "a word ends here".

## Approach 1: Prefix Tree (Array children)

### Intuition

Each node has a fixed array of 26 child slots (one per lowercase letter) and a boolean `end`. Walk the word character by character, creating nodes as needed.

### Algorithm

1. `TrieNode`: `children = [None] * 26`, `end = False`.
2. `insert`: from the root, for each char `c` index `i = ord(c) - ord('a')`; if `children[i]` is `None`, create it; descend. Set `end = True` on the last node.
3. `search`: descend along the word; return `False` if a slot is missing; at the end return the node's `end`.
4. `startsWith`: same descent; return `True` if the whole prefix path exists.

```python
class TrieNode:
    def __init__(self):
        self.children = [None] * 26
        self.end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for c in word:
            i = ord(c) - ord("a")
            if not node.children[i]:
                node.children[i] = TrieNode()
            node = node.children[i]
        node.end = True

    def search(self, word: str) -> bool:
        node = self._walk(word)
        return node is not None and node.end

    def startsWith(self, prefix: str) -> bool:
        return self._walk(prefix) is not None

    def _walk(self, s: str):
        node = self.root
        for c in s:
            i = ord(c) - ord("a")
            if not node.children[i]:
                return None
            node = node.children[i]
        return node
```

### Complexity

- **Time:** $O(L)$ per call, `L` the string length.
- **Space:** $O(26 \cdot t)$ where `t` is the number of nodes.

## Approach 2: Prefix Tree (Hash Map children)

### Intuition

Replace the 26-slot array with a dictionary from character to child node. Same logic, smaller memory when nodes are sparse, and it works for any alphabet.

### Algorithm

1. `TrieNode`: `children = {}`, `end = False`.
2. `insert` / `search` / `startsWith`: identical walks, using `c in node.children` and `node.children[c]`.

```python
class TrieNode:
    def __init__(self):
        self.children: dict[str, "TrieNode"] = {}
        self.end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for c in word:
            node = node.children.setdefault(c, TrieNode())
        node.end = True

    def search(self, word: str) -> bool:
        node = self._walk(word)
        return node is not None and node.end

    def startsWith(self, prefix: str) -> bool:
        return self._walk(prefix) is not None

    def _walk(self, s: str):
        node = self.root
        for c in s:
            if c not in node.children:
                return None
            node = node.children[c]
        return node
```

### Complexity

- **Time:** $O(L)$ per call.
- **Space:** $O(t)$ nodes, each with a small dict.

## Common Pitfalls

- **`search` not checking `end`.** Without inserting `"app"`, `search("app")` must be `False` even though the path exists as a prefix of `"apple"`. That flag is the only thing separating `search` from `startsWith`.
- **Reusing child nodes across letters.** Each character gets its own slot/key; a shared node would merge unrelated words.
- **Array version on non-lowercase input.** `ord(c) - ord('a')` goes out of `[0, 26)` — use the hash-map version for arbitrary characters.

## The keystone

A trie trades $O(26)$-per-node space for $O(L)$ prefix operations with no hashing of the full key. Its shape — descend one character per level, branch on wildcards or neighbours — is the substrate for [Design Add and Search Words](/citadel/dsa/design-add-and-search-words-data-structure) and [Word Search II](/citadel/dsa/word-search-ii).
