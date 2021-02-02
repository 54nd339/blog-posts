---
title: B-Trees - Wide, Shallow Trees Built for Disk
description: A binary tree with a billion keys is 30 levels deep, and on disk each level is a separate seek. B-trees pack hundreds of keys per node so the tree is only three or four levels tall, which is why every relational database indexes with one.
date: 2021-02-02
draft: false
slug: /data-structures/b-tree
tags:
  - Data Structures
  - Balanced Trees
  - Trees
---

A balanced binary search tree with a billion keys is about 30 levels deep. In memory that's 30 pointer hops — fine. On disk, where the index doesn't fit in RAM, each level you descend is a separate seek, and a seek is milliseconds. Thirty seeks per lookup is not fine.

The **B-tree**, from Bayer and McCreight in 1970, fixes this by making each node hold *many* keys — hundreds, sized to a disk block — so it has hundreds of children instead of two. The same billion keys now fit in a tree three or four levels deep, which is three or four block reads per lookup. This is why [PostgreSQL](/citadel/tech/postgres), MySQL, Oracle, and SQL Server all build their indexes on B-trees (or the B+ tree variant).

## The shape

A B-tree of **minimum degree** $t$ (with $t \ge 2$) obeys:

- Every node holds between $t - 1$ and $2t - 1$ keys, sorted within the node. The root is exempt from the lower bound.
- A node with $k$ keys has exactly $k + 1$ children (leaves have none). So every internal node has between $t$ and $2t$ children — the **fanout**.
- **All leaves are at the same depth.** This is what keeps it balanced; there's no separate rotation machinery.
- The $k$ keys in an internal node partition the key space into $k + 1$ ranges, one per child: everything in child $i$ falls between key $i-1$ and key $i$.

Set $t$ so that a full node fills one disk block. With 4 KB blocks and 16-byte entries, $t$ is around 128, and a three-level tree indexes millions of keys.

```python
class Node:
    def __init__(self, leaf):
        self.keys = []
        self.children = []
        self.leaf = leaf

class BTree:
    def __init__(self, t):
        self.t = t                    # minimum degree
        self.root = Node(leaf=True)
```

## Search

Within a node, scan (or binary-search) for the first key $\ge$ the target. If it matches, done. If the node is a leaf, the key isn't in the tree. Otherwise descend into the child on that boundary and repeat.

```python
    def search(self, key, node=None):
        node = node or self.root
        i = 0
        while i < len(node.keys) and key > node.keys[i]:
            i += 1
        if i < len(node.keys) and node.keys[i] == key:
            return True
        return False if node.leaf else self.search(key, node.children[i])
```

Depth is $O(\log_t n)$. The logarithm base is the fanout, so with $t \approx 128$ the tree is dozens of times shallower than a binary one — that's the whole point, measured in disk reads.

## Insertion: split full nodes on the way down

A key always goes into a leaf. The complication is keeping every node within $2t - 1$ keys. The trick is **proactive splitting**: as you descend toward the target leaf, whenever you're about to enter a *full* child, split it first.

Splitting a full node (exactly $2t - 1$ keys) pushes its **median key** up into the parent and divides the remaining $2t - 2$ keys into two nodes of $t - 1$ each. Because you only ever descend into a non-full node, the parent always has room for that promoted key, and splits never cascade upward. The one place the tree grows taller is when the *root* is full: it's split into a new root holding a single key, and the height goes up by one.

```python
    def _split_child(self, parent, i):
        t = self.t
        full = parent.children[i]
        new = Node(leaf=full.leaf)
        parent.keys.insert(i, full.keys[t - 1])         # promote the median
        parent.children.insert(i + 1, new)
        new.keys = full.keys[t:]                        # right half
        full.keys = full.keys[:t - 1]                   # left half
        if not full.leaf:
            new.children = full.children[t:]
            full.children = full.children[:t]

    def insert(self, key):
        root = self.root
        if len(root.keys) == 2 * self.t - 1:            # root full: grow upward
            new_root = Node(leaf=False)
            new_root.children.append(root)
            self._split_child(new_root, 0)
            self.root = new_root
        self._insert_nonfull(self.root, key)

    def _insert_nonfull(self, node, key):
        i = len(node.keys) - 1
        if node.leaf:
            node.keys.append(None)
            while i >= 0 and key < node.keys[i]:        # shift to keep sorted
                node.keys[i + 1] = node.keys[i]
                i -= 1
            node.keys[i + 1] = key
        else:
            while i >= 0 and key < node.keys[i]:
                i -= 1
            i += 1
            if len(node.children[i].keys) == 2 * self.t - 1:
                self._split_child(node, i)
                if key > node.keys[i]:
                    i += 1
            self._insert_nonfull(node.children[i], key)
```

### A walkthrough

Take $t = 2$: nodes hold 1 to 3 keys. Insert **1, 2, 3, 4, 5, 6**:

- `1, 2, 3` → the root fills up: `[1, 2, 3]`.
- `4` → the root is full, so it splits first. Median `2` goes up into a new root; the old node becomes `[1]` and `[3]`. Now the tree is `[2]` over `[1]` and `[3]`, and it's one level taller. Then `4` descends past `2` into `[3]`, giving `[3, 4]`.
- `5` → descends into `[3, 4]` (not full) → `[3, 4, 5]`.
- `6` → the root `[2]` points at a full child `[3, 4, 5]`, so split it: median `4` goes up, leaving `[3]` and `[5]`. Root is now `[2, 4]` over `[1]`, `[3]`, `[5]`. Then `6` descends past `4` into `[5]` → `[5, 6]`.

Final tree: `[2, 4]` with children `[1]`, `[3]`, `[5, 6]`. All leaves at the same depth.

## Deletion: keep every node at $\ge t - 1$ keys

Deletion is the mirror problem: after removing a key a node might **underflow** (drop below $t - 1$ keys). Two repairs:

- **Borrow.** If an adjacent sibling has a key to spare ($\ge t$), rotate one through the parent: the separating key in the parent moves down into the short node, and a key from the sibling moves up to replace it.
- **Merge.** If neither sibling can spare one, fuse the short node, a separating key pulled down from the parent, and a sibling into a single node of $2t - 1$ keys. The parent loses a key, so a merge can cascade upward. If the root ends up empty, its only child becomes the new root and the tree shrinks by a level.

Deleting a key from an *internal* node is reduced to a leaf deletion: replace it with its predecessor (largest key in the left subtree) or successor (smallest in the right), then delete that from the leaf. As with insertion, the clean way is to ensure a child has $\ge t$ keys *before* descending into it.

```python
    def delete(self, key):
        self._delete(self.root, key)
        if not self.root.keys and not self.root.leaf:   # root emptied by a merge
            self.root = self.root.children[0]

    def _delete(self, node, key):
        t = self.t
        i = 0
        while i < len(node.keys) and key > node.keys[i]:
            i += 1
        if i < len(node.keys) and node.keys[i] == key:
            if node.leaf:
                node.keys.pop(i)
            else:
                self._delete_internal(node, key, i)
        elif not node.leaf:
            self._descend(node, key, i)

    def _delete_internal(self, node, key, i):
        t = self.t
        left, right = node.children[i], node.children[i + 1]
        if len(left.keys) >= t:                          # predecessor
            pred = self._max_key(left)
            node.keys[i] = pred
            self._delete(left, pred)
        elif len(right.keys) >= t:                       # successor
            succ = self._min_key(right)
            node.keys[i] = succ
            self._delete(right, succ)
        else:                                            # both minimal: merge
            self._merge(node, i)
            self._delete(node.children[i], key)

    def _descend(self, node, key, i):
        if len(node.children[i].keys) < self.t:
            self._fill(node, i)
        j = 0
        while j < len(node.keys) and key > node.keys[j]:
            j += 1
        if j < len(node.keys) and node.keys[j] == key:
            self._delete(node, key)
        else:
            self._delete(node.children[min(j, len(node.children) - 1)], key)

    def _fill(self, node, i):
        t = self.t
        if i > 0 and len(node.children[i - 1].keys) >= t:
            self._borrow_prev(node, i)
        elif i < len(node.children) - 1 and len(node.children[i + 1].keys) >= t:
            self._borrow_next(node, i)
        elif i < len(node.children) - 1:
            self._merge(node, i)
        else:
            self._merge(node, i - 1)

    def _borrow_prev(self, node, i):
        child, sib = node.children[i], node.children[i - 1]
        child.keys.insert(0, node.keys[i - 1])
        node.keys[i - 1] = sib.keys.pop()
        if not sib.leaf:
            child.children.insert(0, sib.children.pop())

    def _borrow_next(self, node, i):
        child, sib = node.children[i], node.children[i + 1]
        child.keys.append(node.keys[i])
        node.keys[i] = sib.keys.pop(0)
        if not sib.leaf:
            child.children.append(sib.children.pop(0))

    def _merge(self, node, i):
        left, right = node.children[i], node.children[i + 1]
        left.keys.append(node.keys.pop(i))
        left.keys.extend(right.keys)
        left.children.extend(right.children)
        node.children.pop(i + 1)

    def _max_key(self, node):
        while not node.leaf:
            node = node.children[-1]
        return node.keys[-1]

    def _min_key(self, node):
        while not node.leaf:
            node = node.children[0]
        return node.keys[0]
```

```python
bt = BTree(t=3)
for k in [10, 20, 5, 6, 12, 30, 7, 17, 3, 8, 25, 1, 40, 15]:
    bt.insert(k)
assert bt.search(17) and not bt.search(99)
for k in [6, 12, 30, 1]:
    bt.delete(k)
# an in-order walk of the tree stays sorted; all leaves stay at one depth
```

Search, insert, and delete are all $O(\log_t n)$ node visits, each visit touching one disk block.

## B-trees vs B+ trees

The **B+ tree** variant keeps *all* keys in the leaves — internal nodes hold only separator keys for routing — and links the leaves in a list. Two consequences: internal nodes fit more separators (higher fanout, shallower tree), and a range scan becomes "find the start leaf, then follow the links". That's ideal for `WHERE age BETWEEN 20 AND 40`, so it's what most databases actually use.

## Where they're used

- **Database indexes** — the primary structure for finding rows by key in essentially every relational database.
- **Filesystems** — NTFS, HFS+, and others store directory and metadata trees as B-trees, so a directory with a million files still looks up in a few reads.
- **Anywhere** a large sorted dataset lives on disk and gets searched, inserted, and deleted.

## The one idea to keep

B-trees exist because on disk the cost that matters isn't comparisons, it's block reads, and a high fanout — hundreds of keys per node, one node per block — is how you drive a billion-key index from 30 levels down to 3 or 4. All leaves sit at the same depth, so balance comes from split-and-merge on the way down rather than rotations. In memory, where every node is a cache line away, that pressure reverses and a binary [red-black](/citadel/data-structures/red-black-tree) or [AVL tree](/citadel/data-structures/avl-tree) is the right shape instead. The B+ tree — keys only in linked leaves — is the refinement databases actually ship, because it makes range scans a leaf walk.
