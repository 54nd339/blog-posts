---
title: Tries - Prefix Trees for Fast Autocomplete and Lookup
description: A trie stores a set of strings so that insert, exact-match search, and "does any word start with this prefix" all cost O(length) and don't slow down as the set grows. The key it's storing lives in the path from the root, not in any node.
date: 2021-02-04
draft: false
slug: /data-structures/trie
tags:
  - Data Structures
  - Trees
  - Strings
---

When a search box shows suggestions as you type, something is answering "which stored words start with `rec`?" on every keystroke. A hash set can't — it only does exact match. Sorting the words and binary-searching works but costs $O(\log n)$ per lookup and grows with the dictionary.

A **trie** (from re*trie*val, usually said "try"), also called a prefix tree or digital tree, does it in $O(\text{length of the query})$ regardless of how many words are stored. The trick: a node doesn't hold a key. The *path* from the root to a node spells the key.

## The structure

Each edge is labelled with a character. Walk down from the root and concatenate the edge labels, and you've spelled a string. A node carries:

- A map (or array) from the next character to the child node for it.
- A boolean `is_end` — true if the path from the root to *this* node is a complete stored word, not just a prefix of one.

Strings that share a prefix share the nodes for that prefix. A dictionary full of `un-`, `re-`, and `-ing` reuses those paths once each, which is where the space savings come from.

## A walkthrough

Insert `cat`, `car`, `card`, `dog`:

```
        root
        /  \
      c     d
      |     |
      a     o
     / \    |
    t   r   g*
    *   |*
        d*
```

`cat` and `car` share the `c → a` path and branch at the third character. `card` extends `car` by one node. `dog` shares nothing, so it's a separate branch. Nodes marked `*` have `is_end = true`.

Now the queries:

- `search("car")` → follow `c → a → r`, land on a node with `is_end = true` → **true**.
- `search("ca")` → follow `c → a`, land on a node with `is_end = false` (it's only a prefix) → **false**.
- `search("care")` → follow `c → a → r`, then look for `e` — no such child → **false**.
- `starts_with("car")` → follow `c → a → r`, path exists → **true**. `is_end` doesn't matter for a prefix check.

## The code

Using a dictionary per node, which handles any alphabet:

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word):
        node = self._walk(word)
        return node is not None and node.is_end

    def starts_with(self, prefix):
        return self._walk(prefix) is not None

    def _walk(self, s):
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node


t = Trie()
for w in ["cat", "car", "card", "dog"]:
    t.insert(w)
assert t.search("car") and not t.search("ca") and not t.search("care")
assert t.starts_with("car") and not t.starts_with("z")
```

All three operations are the same walk: follow one child per character. Insert creates missing nodes as it goes; `search` and `starts_with` bail out the moment a character has no child. `search` additionally checks `is_end` at the end; `starts_with` doesn't.

### Autocomplete: enumerate a subtree

The reason a trie powers a suggestion box: once you've walked to the prefix node, every word starting with that prefix is a marked node in its subtree. Collect them with a depth-first walk.

```python
def words_with_prefix(trie, prefix):
    node = trie._walk(prefix)
    found = []
    if node is None:
        return found

    def dfs(n, suffix):
        if n.is_end:
            found.append(prefix + suffix)
        for ch, child in n.children.items():
            dfs(child, suffix + ch)

    dfs(node, "")
    return found

# words_with_prefix(t, "car") -> ["car", "card"]
```

This costs $O(p)$ to reach the prefix plus the size of the subtree beneath it — you pay only for the matches, not the whole dictionary.

### Array nodes for a fixed alphabet

If the alphabet is small and fixed — lowercase `a`–`z`, say — a node can use a length-26 array instead of a map, indexing with `ord(ch) - ord('a')`. Direct array indexing is faster than a hash lookup, but every node pays for 26 slots whether it uses them or not, so this wins only when nodes are densely populated.

```python
class TrieNode:
    def __init__(self):
        self.children = [None] * 26      # index = ord(ch) - ord('a')
        self.is_end = False
```

## Cost

| Operation | Time |
| --- | --- |
| Insert a word of length $m$ | $O(m)$ |
| Search for a word of length $m$ | $O(m)$ |
| Prefix check of length $p$ | $O(p)$ |

None of these depend on $n$, the number of stored words — that's the property a hash set and a sorted array don't have for prefixes. Space is $O(\text{total characters})$ in the worst case, less when prefixes are shared. The weakness: strings with few common prefixes get a node per character with little reuse, so a trie can use more memory than just storing the strings.

## Where tries are used

- **Autocomplete and typeahead** — walk to the prefix node, then enumerate the subtree.
- **Spell check and autocorrect** — membership tests against a dictionary trie, plus nearby-word suggestions.
- **IP routing** — routers match a destination address against a trie of route prefixes and take the *longest* match to pick the next hop.
- **Dictionaries and word games** — fast membership and prefix tests.

A **radix tree** (or Patricia trie) is the compressed variant: chains of single-child nodes are collapsed into one node holding a substring, which cuts the node count for sparse sets. A **suffix tree** stores every suffix of one text and powers substring search and bioinformatics work.

## The one idea to keep

The trie's defining move is that the key is not *in* a node — it is the path of edge labels from the root. That is what makes every operation cost $O(\text{query length})$ and never scale with the number of stored words, and it is what makes "which words start with this prefix?" a one-line subtree walk instead of a full scan. The price is memory when strings share few prefixes: a hash set stores the bytes once, a sparse trie spends a node per character. Reach for a trie when prefixes are the query and sharing is real; fall back to a hash set when you only need exact match.
