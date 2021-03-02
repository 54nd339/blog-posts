---
title: Huffman Coding - Optimal Prefix Codes by Greedy Merging
description: Give frequent symbols short bit-strings and rare ones long ones, so no code is a prefix of another. Repeatedly merging the two least frequent nodes builds the tree that minimizes total encoded length, and it's provably optimal.
date: 2021-03-02
draft: false
slug: /algorithms/HuffmanCoding
tags:
  - Algorithms
  - Greedy
---

Fixed-length encoding spends 8 bits on every character whether it's a common `e` or a rare `z`. **Huffman coding**, from David Huffman's 1952 MIT term paper, spends fewer bits on frequent symbols and more on rare ones, and does it optimally.

The codes it produces are **prefix-free**: no symbol's code is a prefix of another's. That's what lets a decoder read a bitstream left to right and know exactly where each symbol ends, without separators.

## Building the tree

Model the code as a binary tree: symbols at the leaves, and the path from the root (0 for left, 1 for right) is a symbol's code. A frequent symbol should be near the root, a rare one deep.

The greedy construction, using a min-heap keyed on frequency:

1. Make a leaf node for each symbol, holding its frequency.
2. While more than one node remains: remove the two lowest-frequency nodes, make them the children of a new internal node whose frequency is their sum, and put that node back.
3. The last node left is the root.

Each merge buries the two currently-rarest symbols one level deeper — exactly the ones that can afford it.

## The code

```python
import heapq, itertools
from collections import Counter

def huffman_codes(text):
    freq = Counter(text)
    if len(freq) == 1:                       # single-symbol edge case
        return {next(iter(freq)): "0"}

    tie = itertools.count()                  # unique id, so heap never compares subtrees
    heap = [[f, next(tie), sym, None, None] for sym, f in freq.items()]
    heapq.heapify(heap)
    while len(heap) > 1:
        lo = heapq.heappop(heap)
        hi = heapq.heappop(heap)
        heapq.heappush(heap, [lo[0] + hi[0], next(tie), None, lo, hi])
    root = heap[0]

    codes = {}
    def walk(node, prefix):
        _, _, sym, left, right = node
        if sym is not None:
            codes[sym] = prefix
        else:
            walk(left, prefix + "0")
            walk(right, prefix + "1")
    walk(root, "")
    return codes


def encode(text, codes):
    return "".join(codes[c] for c in text)


text = "this is an example of a huffman tree"
codes = huffman_codes(text)
bits = encode(text, codes)
assert len(bits) < 8 * len(text)                       # beats fixed 8-bit
# prefix-free: no code starts with another
assert all(a == b or not a.startswith(b)
           for a in codes.values() for b in codes.values())
```

## Why it's optimal

Two properties, the greedy-algorithm pair again:

- **Greedy choice.** In *some* optimal tree, the two least-frequent symbols are siblings at the deepest level. If they weren't, you could swap them with whatever is down there at no extra cost (moving a lower frequency deeper and a higher frequency shallower can only help), so merging them first loses nothing.
- **Optimal substructure.** After merging symbols $x$ and $y$ into a node $z$ of frequency $f_x + f_y$, an optimal tree for the smaller alphabet (with $z$ in place of $x, y$) becomes an optimal tree for the original when you re-expand $z$ into $x$ and $y$.

Together these say the greedy merges build a minimum-total-length tree.

## Cost

Building the heap is $O(n)$ for $n$ distinct symbols; the loop runs $n-1$ times with two pops and a push, each $O(\log n)$: **$O(n \log n)$**. Counting frequencies from a text of length $N$ adds $O(N)$. The tree has $2n - 1$ nodes, so space is $O(n)$.

## Where it's used

Huffman coding is a stage in ZIP and GZIP (after LZ77 finds repeats), in JPEG (after the lossy transform), and in MPEG audio and video. It's optimal among codes that assign a whole number of bits per symbol; arithmetic and range coding do slightly better by breaking that restriction. The heap-driven greedy construction is the same shape as [Prim's algorithm](/citadel/algorithms/MinimumSpanningTree) for spanning trees, and the correctness argument matches [activity selection](/citadel/algorithms/ActivitySelection) and [fractional knapsack](/citadel/algorithms/FractionalKnapsack).
