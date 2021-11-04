---
title: Word Search II - One Trie Walk Instead of Many
description: Finding which of many words appear in a grid — searching each word separately is far too slow, so build a trie of all words and let one DFS over the board match them all at once.
date: 2021-11-04
draft: false
slug: /dsa/word-search-ii
tags:
  - Competitive Programming
  - Tries
  - Backtracking
---

Running a board DFS once per word wastes work — many words share prefixes and would re-walk the same cells. Put all words in a trie, then a single DFS from each cell follows the trie: it explores a path only while some word still has that prefix.

## Description

Given an `m x n` board of letters and a list of `words`, return all words that can be formed by a path of horizontally or vertically adjacent cells, using each cell at most once per word.

**Example**

```
Input:  board = [["o","a","a","n"],
                 ["e","t","a","e"],
                 ["i","h","k","r"],
                 ["i","f","l","v"]]
        words = ["oath","pea","eat","rain"]
Output: ["oath","eat"]
```

**Constraints**

- $1 \le m, n \le 12$
- $1 \le \text{words.length} \le 3 \times 10^4$; $1 \le \text{words}[i]\text{.length} \le 10$
- Lowercase letters; all words unique.

## Prerequisites

- [Implement Trie](/citadel/dsa/implement-trie-prefix-tree).
- Grid DFS with a visited marker and backtracking ([Word Search](/citadel/dsa/word-search)).

## Approach 1: Backtracking (per word)

### Intuition

For each word, try every cell as a start and DFS to spell it out, marking cells used along the current path.

### Algorithm

1. For each `word`: for each cell matching `word[0]`, run `dfs(r, c, k)` where `k` is the index into `word`.
2. `dfs`: if `k == len(word)`, matched. Reject out-of-bounds, mismatch, or visited cells. Mark visited, recurse into 4 neighbours with `k + 1`, unmark.
3. Collect words that matched from any start.

```python
def findWords(board, words):
    rows, cols = len(board), len(board[0])
    res = []

    def dfs(r, c, word, k):
        if k == len(word):
            return True
        if r < 0 or c < 0 or r >= rows or c >= cols or board[r][c] != word[k]:
            return False
        tmp, board[r][c] = board[r][c], "#"
        found = (dfs(r + 1, c, word, k + 1) or dfs(r - 1, c, word, k + 1) or
                 dfs(r, c + 1, word, k + 1) or dfs(r, c - 1, word, k + 1))
        board[r][c] = tmp
        return found

    for word in words:
        if any(dfs(r, c, word, 0) for r in range(rows) for c in range(cols)):
            res.append(word)
    return res
```

### Complexity

- **Time:** $O(w \cdot m \cdot n \cdot 4 \cdot 3^{t-1})$ — `w` words, `t` the max word length.
- **Space:** $O(t)$ recursion.

## Approach 2: Backtracking with a Trie

### Intuition

Insert all words into a trie. DFS from every cell, advancing a trie pointer in lockstep with the path. Stop as soon as the current letter is not a child of the trie node — that prunes every dead prefix for *all* words at once. When a trie node marks a word end, record it.

### Algorithm

1. Build a trie; store the full word on its end node (or reconstruct it).
2. `dfs(r, c, node)`: if the cell is out of bounds, visited, or `board[r][c]` not in `node.children`, return.
3. Descend to `child = node.children[board[r][c]]`. If `child.word` is set, add it to a result set and clear it (avoid duplicates).
4. Mark the cell visited; recurse into 4 neighbours with `child`; unmark.
5. Start `dfs` from every cell with the trie root.

```python
def findWords(board, words):
    root = {}
    for w in words:
        node = root
        for c in w:
            node = node.setdefault(c, {})
        node["$"] = w

    rows, cols = len(board), len(board[0])
    found = set()

    def dfs(r, c, node):
        if r < 0 or c < 0 or r >= rows or c >= cols:
            return
        ch = board[r][c]
        if ch not in node:
            return
        nxt = node[ch]
        if "$" in nxt:
            found.add(nxt["$"])
        board[r][c] = "#"
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            dfs(r + dr, c + dc, nxt)
        board[r][c] = ch

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, root)
    return list(found)
```

### Complexity

- **Time:** $O(m \cdot n \cdot 4 \cdot 3^{t-1} + s)$ where `s` is the total length of all words.
- **Space:** $O(s)$ for the trie.

## Approach 3: Trie with Pruning

### Intuition

As words are found, prune the trie: a leaf whose word has been collected, and any node with no remaining children, can be removed. This keeps later DFS branches from exploring exhausted parts of the trie.

### Algorithm

1. Build the trie as in Approach 2.
2. In `dfs`, after recursing from a child node, if that child now has no children and no word, delete it from its parent (`del node[ch]`).
3. When a word is found, remove its `"$"` marker so it is only reported once.

```python
def findWords(board, words):
    root = {}
    for w in words:
        node = root
        for c in w:
            node = node.setdefault(c, {})
        node["$"] = w

    rows, cols = len(board), len(board[0])
    res = []

    def dfs(r, c, parent):
        ch = board[r][c]
        node = parent[ch]
        word = node.pop("$", None)
        if word:
            res.append(word)

        board[r][c] = "#"
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] in node:
                dfs(nr, nc, node)
        board[r][c] = ch

        if not node:
            parent.pop(ch)

    for r in range(rows):
        for c in range(cols):
            if board[r][c] in root:
                dfs(r, c, root)
    return res
```

### Complexity

- **Time:** $O(m \cdot n \cdot 4 \cdot 3^{t-1} + s)$, with a smaller constant as branches are pruned.
- **Space:** $O(s)$, shrinking as words are found.

## Common Pitfalls

- **Per-word DFS times out.** With up to $3 \times 10^4$ words, only the shared-trie walk is fast enough.
- **Reporting duplicates.** A word can be reachable by several paths; store results in a set or delete the end marker on first find.
- **Visited marker not restored.** Overwrite `board[r][c]` for the current path and always set it back after recursing, or later paths see a corrupted grid.
- **Pruning during iteration carelessly.** Only delete a child after its subtree is fully explored and confirmed empty.

## The keystone

When you must match *many* patterns against one text (or grid), index the patterns into a trie and drive a single traversal by it — every shared prefix is walked once. This is the same idea as Aho–Corasick for multi-pattern string search, applied to a 2-D board.
