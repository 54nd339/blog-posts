---
title: Trees - Terminology, the Main Variants, and Traversals
description: A tour of tree data structures - the vocabulary, the binary search tree and its self-balancing descendants, heaps, and the string and range trees - plus the four traversal orders and what each one is good for.
date: 2021-01-12
draft: false
slug: /data-structures/trees
tags:
  - Data Structures
  - Trees
---

Hierarchy is everywhere in computing — file systems, the DOM, org charts, parse trees — and the **tree** is the data structure that captures it. A tree is a set of **nodes**, each holding a value and references to child nodes, with one node designated the **root** and every other node having exactly one parent.

This is a map of the family: the words you need, the binary search tree and the self-balancing variants built on it, heaps, the specialised string and range trees, and the traversal orders that visit every node. Each major variant has its own post; this one places them.

## Vocabulary

| Term | Meaning |
| --- | --- |
| Root | The one node with no parent |
| Leaf | A node with no children |
| Edge | The link between a parent and a child |
| Path | A sequence of nodes connected by edges |
| Depth of a node | Number of edges from the root to it (root is depth 0) |
| Height of a tree | Longest root-to-leaf path (single node: 0; empty: −1 by convention) |
| Degree of a node | Its number of children |
| Subtree | Any node together with all its descendants |

## Binary search trees

A **binary tree** gives each node at most two children. A **binary search tree (BST)** adds an ordering rule: for every node, all keys in its left subtree are smaller and all keys in its right subtree are larger. That rule turns search into a sequence of left/right decisions.

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

class BST:
    def __init__(self):
        self.root = None

    def insert(self, value):
        if self.root is None:
            self.root = Node(value)
        else:
            self._insert(self.root, value)

    def _insert(self, node, value):
        if value < node.value:
            if node.left is None:
                node.left = Node(value)
            else:
                self._insert(node.left, value)
        else:
            if node.right is None:
                node.right = Node(value)
            else:
                self._insert(node.right, value)

    def search(self, value):
        node = self.root
        while node is not None:
            if value == node.value:
                return True
            node = node.left if value < node.value else node.right
        return False
```

Search, insert, and delete are $O(h)$ where $h$ is the height. If the tree stays balanced, $h = O(\log n)$. If keys arrive in sorted order, the tree degenerates into a linked list and $h = O(n)$ — every operation back to linear.

### Self-balancing variants

These keep $h = O(\log n)$ no matter the insertion order, by restructuring after each update:

- **[AVL tree](/citadel/data-structures/avl-tree)** — every node's two subtrees stay within one level of each other. Strictest balance, fastest lookups, more rotations on update. Named for Adelson-Velsky and Landis, 1962.
- **[Red-black tree](/citadel/data-structures/red-black-tree)** — nodes coloured red or black under five rules that cap the longest path at twice the shortest. Looser balance, fewer rotations, so it's what `std::map`, Java's `TreeMap`, and the Linux scheduler use.
- **[B-tree](/citadel/data-structures/b-tree)** — not binary: each node holds many keys and has many children. Wide and shallow, so a lookup is a handful of disk-block reads. The standard structure for database and filesystem indexes.

## Heaps

A **binary heap** is a complete binary tree (every level full except possibly the last, filled left to right) with the *heap property*: in a max-heap, every node is $\ge$ its children; in a min-heap, every node is $\le$ its children. The complete shape means it packs into an array with no pointers — node $i$'s children are at $2i+1$ and $2i+2$.

```python
class MaxHeap:
    def __init__(self):
        self.a = []

    def push(self, x):
        self.a.append(x)
        i = len(self.a) - 1
        while i > 0 and self.a[i] > self.a[(i - 1) // 2]:      # bubble up
            self.a[i], self.a[(i - 1) // 2] = self.a[(i - 1) // 2], self.a[i]
            i = (i - 1) // 2

    def pop_max(self):
        top = self.a[0]
        last = self.a.pop()
        if self.a:
            self.a[0] = last
            i, n = 0, len(self.a)
            while 2 * i + 1 < n:                               # sift down
                child = 2 * i + 1
                if child + 1 < n and self.a[child + 1] > self.a[child]:
                    child += 1
                if self.a[i] >= self.a[child]:
                    break
                self.a[i], self.a[child] = self.a[child], self.a[i]
                i = child
        return top
```

Push and pop-max are $O(\log n)$; reading the max is $O(1)$; searching for an arbitrary element is $O(n)$. Heaps implement **priority queues** and drive [heapsort](/citadel/algorithms/SortingSearching).

A **Fibonacci heap** is a lazier design with better amortised bounds — insert, merge, and decrease-key in $O(1)$, extract-min in $O(\log n)$ — which speeds up [Dijkstra's shortest paths](/citadel/algorithms/PathFinding) and [Prim's MST](/citadel/algorithms/MinimumSpanningTree) on dense graphs, though the constant factors keep it mostly theoretical.

## String and range trees

- **[Trie](/citadel/data-structures/trie)** (prefix tree) — the key is spelled by the path from the root, not stored in a node. Insert, search, and prefix-check all cost $O(\text{string length})$, independent of how many strings are stored. Autocomplete, spell check, IP routing.
- **Suffix tree** — a compressed trie of every suffix of one text; powers substring search and DNA analysis.
- **[Segment tree](/citadel/data-structures/segment-tree)** — each node stores an aggregate (sum, min, max) over an interval of an array, giving $O(\log n)$ range queries *and* point updates.

## Traversals

A traversal visits every node exactly once; the order is what varies.

**Depth-first** goes as deep as possible before backtracking. The three orders differ only in *when* the node itself is visited relative to its subtrees:

```python
def preorder(node, visit):      # node, then left, then right
    if node is None:
        return
    visit(node)
    preorder(node.left, visit)
    preorder(node.right, visit)

def inorder(node, visit):       # left, then node, then right
    if node is None:
        return
    inorder(node.left, visit)
    visit(node)
    inorder(node.right, visit)

def postorder(node, visit):     # left, then right, then node
    if node is None:
        return
    postorder(node.left, visit)
    postorder(node.right, visit)
    visit(node)
```

- **Inorder** on a BST visits keys in sorted order — the reason `inorder` is the one you reach for most.
- **Preorder** visits a node before its children, so it's used to copy a tree or serialise its structure.
- **Postorder** visits children before the node, which is what you want when freeing a tree or evaluating an expression tree bottom-up.

**Breadth-first** (level-order) visits all nodes at one depth before the next, using a queue:

```python
from collections import deque

def level_order(root, visit):
    if root is None:
        return
    q = deque([root])
    while q:
        node = q.popleft()
        visit(node)
        if node.left:
            q.append(node.left)
        if node.right:
            q.append(node.right)
```

## Key takeaways

- A tree is nodes linked parent-to-child from a single root; height determines the cost of most operations.
- A **BST** gives $O(\log n)$ operations only while balanced; **AVL**, **red-black**, and **B-trees** guarantee it by restructuring on every update, trading rotation cost for a height bound.
- A **heap** is an array-backed complete tree for priority queues; **tries** index strings by path; **segment trees** answer range queries with point updates.
- **Inorder** traversal sorts a BST; **preorder** serialises structure; **postorder** processes children first; **level-order** goes breadth-first with a queue.

The [broad data-structure survey](/citadel/data-structures/data-structures) places trees alongside arrays, hash tables, and graphs.
