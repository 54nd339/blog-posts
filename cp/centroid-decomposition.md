---
title: Centroid Decomposition - Divide and Conquer on Trees
description: Recursively removing the centroid of a tree to build a balanced decomposition of depth O(log n), then counting or aggregating over all paths by processing, at each centroid, only the paths that pass through it.
date: 2024-10-27
draft: false
slug: /cp/centroid-decomposition
tags:
  - Competitive Programming
  - Graphs
  - Trees
---

A **centroid** of a tree is a vertex whose removal leaves every remaining component with at most $n/2$ vertices. Every tree has one (or two). Remove it, recurse into each piece, and you get a recursion tree of depth $O(\log n)$ — the centroid decomposition. It is the tree analogue of "pick the middle element": the workhorse for problems about *all paths* in a tree.

## The problem

Answer questions that range over every pair of vertices and the path between them:

- how many pairs are at distance exactly $k$ (or at most $k$)?
- the number of paths whose length is a multiple of $3$, whose edge colours form a palindrome, whose weight sum lies in $[a, b]$;
- closest marked vertex to each query vertex, under updates.

Brute force is $O(n^2)$. Centroid decomposition brings most of these to $O(n \log n)$ or $O(n \log^2 n)$.

## The idea: every path has a highest centroid

Build the decomposition: find the centroid $c$ of the whole tree, "remove" it, and recurse on each resulting component (each has $\le n/2$ vertices, so the recursion is $O(\log n)$ deep and touches $O(n \log n)$ vertex-visits in total).

The key observation: **any path $u \to v$ in the original tree passes through exactly one centroid that is "highest" in the decomposition** — the first centroid removed that lies on the path. So if, at each centroid $c$, you count only the paths that go *through $c$*, every path is counted exactly once, at its highest centroid.

**Counting paths through $c$.** Run a BFS from $c$ into the (not-yet-removed) component, recording the distance of every vertex, grouped by which neighbour of $c$ the branch started at. A path through $c$ joins a vertex in one branch to a vertex in another (or to $c$ itself). Maintain a running multiset of distances seen so far; for each new branch, first *query* it against the running multiset (pairs from different branches), then *merge* it in. Subtracting same-branch pairs afterward, or querying branch-by-branch as here, prevents counting a path that never actually reaches $c$.

## How it works: pairs at distance at most $K$

```python
from collections import deque
import bisect

def count_pairs_within(n, adj, K):
    removed = [False] * n
    size = [0] * n
    total = 0

    def compute_sizes(entry):
        order, stack, parent = [], [(entry, -1)], {entry: -1}
        while stack:
            u, p = stack.pop()
            order.append(u)
            for w in adj[u]:
                if w != p and not removed[w]:
                    parent[w] = u
                    stack.append((w, u))
        for u in reversed(order):
            size[u] = 1
            for w in adj[u]:
                if parent.get(w, -2) == u:
                    size[u] += size[w]
        return size[entry]

    def find_centroid(entry, total_size):
        u, p = entry, -1
        while True:
            for w in adj[u]:
                if w != p and not removed[w] and size[w] > total_size // 2 and size[w] < size[u]:
                    p, u = u, w
                    break
            else:
                return u

    def branch_distances(c, first):
        out, dq = [], deque([(first, c, 1)])
        while dq:
            u, p, d = dq.popleft()
            if d <= K:
                out.append(d)
            for w in adj[u]:
                if w != p and not removed[w]:
                    dq.append((w, u, d + 1))
        return out

    stack = [0]
    while stack:
        entry = stack.pop()
        c = find_centroid(entry, compute_sizes(entry))
        removed[c] = True

        seen = [0]                               # the centroid itself, distance 0
        for w in adj[c]:
            if removed[w]:
                continue
            branch = branch_distances(c, w)
            for d in branch:                     # pair with everything already seen
                total += bisect.bisect_right(seen, K - d)
            for d in branch:
                bisect.insort(seen, d)

        for w in adj[c]:
            if not removed[w]:
                stack.append(w)
    return total
```

For "distance exactly $k$" swap the sorted list for a frequency array indexed $0 \dots k$ and add `freq[k - d]`. For weighted paths use a Fenwick tree keyed by distance.

## The centroid tree

Recording, for each centroid, the centroid that removed *its* component gives the **centroid tree**: a rooted tree of depth $O(\log n)$ where the path between two original vertices always contains their centroid-tree LCA. This supports "nearest marked vertex" style queries with updates: store at each centroid the best distance among marked vertices in its component; a query walks its $O(\log n)$ centroid-tree ancestors.

## Complexity

- **Build + one linear-work pass per level:** $O(n \log n)$ vertex visits; with a sort or Fenwick per centroid, $O(n \log^2 n)$.
- **Space:** $O(n)$ (plus $O(n)$ for the centroid tree if you keep it).

## Common pitfalls

- **Recomputing sizes over removed vertices.** After removing a centroid, `compute_sizes` and every BFS must skip `removed[]` vertices, and sizes are relative to the *current* component, not the original tree.
- **Counting same-branch pairs.** A pair of vertices in the same branch of $c$ has a path that does not pass through $c$ — it will be counted at a deeper centroid. Query each branch against the accumulated set *before* merging it, or subtract each branch's internal pairs.
- **Forgetting the centroid itself.** Paths from $c$ to any vertex are valid; seed the running multiset with distance $0$.
- **Wrong centroid.** The centroid must make *all* components $\le \text{total}/2$. Stopping at "the first vertex with a small subtree" without checking the parent side gives an unbalanced split and $O(n^2)$ blowup.
- **Recursion depth.** The first `compute_sizes` runs on the whole tree — a path graph is depth $n$. Use the iterative traversals shown here.

## The keystone

Recursively delete the centroid — the vertex that splits the tree into halves — to get an $O(\log n)$-deep decomposition. Every path has a unique highest centroid, so processing at each centroid only the paths through it counts every path once, turning all-pairs path problems into $O(n \log n)$ BFS-and-merge work.
