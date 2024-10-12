---
title: Lowest Common Ancestor - Binary Lifting, Euler Tour, and Offline
description: Finding the deepest node that is an ancestor of both u and v — binary lifting for O(log n) online queries, Euler tour plus sparse-table RMQ for O(1), and Tarjan's offline union-find pass.
date: 2024-10-12
draft: false
slug: /cp/lowest-common-ancestor
tags:
  - Competitive Programming
  - Graphs
  - Trees
---

The lowest common ancestor of $u$ and $v$ in a rooted tree is the deepest node that is an ancestor of both. It is the primitive behind tree-path queries: $\text{dist}(u, v) = \text{depth}[u] + \text{depth}[v] - 2\,\text{depth}[\text{lca}(u, v)]$, path sums via prefix sums to the root, and [heavy-light decomposition](/citadel/cp/heavy-light-decomposition). Three standard methods trade preprocessing for query time.

## The problem

Preprocess a rooted tree of $n$ nodes so that many `lca(u, v)` queries are fast. Also common: `k`-th ancestor of $v$, distance between $u$ and $v$, "is $u$ on the path from $a$ to $b$".

## Method 1: Binary lifting (O(n log n) build, O(log n) query)

Store `up[v][k]` = the $2^k$-th ancestor of $v$, filled by `up[v][k] = up[ up[v][k-1] ][k-1]`. To find `lca(u, v)`:

1. Lift the deeper of $u, v$ up to the other's depth (using the bits of the depth difference).
2. If they coincide, that is the LCA.
3. Otherwise, from the highest power down, jump both up by $2^k$ whenever their $2^k$-th ancestors *differ*. They end up as children of the LCA; return `up[u][0]`.

```python
import sys

def build_lca(n, adj, root=0):
    sys.setrecursionlimit(1 << 25)
    LOG = max(1, (n).bit_length())
    up = [[root] * n for _ in range(LOG)]
    depth = [0] * n
    order = []
    stack = [(root, root)]
    seen = [False] * n
    while stack:
        v, p = stack.pop()
        if seen[v]:
            continue
        seen[v] = True
        up[0][v] = p
        for to in adj[v]:
            if not seen[to]:
                depth[to] = depth[v] + 1
                stack.append((to, v))
    for k in range(1, LOG):
        for v in range(n):
            up[k][v] = up[k - 1][up[k - 1][v]]
    return up, depth, LOG

def lca(u, v, up, depth, LOG):
    if depth[u] < depth[v]:
        u, v = v, u
    d = depth[u] - depth[v]
    for k in range(LOG):
        if d >> k & 1:
            u = up[k][u]
    if u == v:
        return u
    for k in range(LOG - 1, -1, -1):
        if up[k][u] != up[k][v]:
            u, v = up[k][u], up[k][v]
    return up[0][u]

def kth_ancestor(v, k, up, LOG):
    for i in range(LOG):
        if k >> i & 1:
            v = up[i][v]
            if v is None:
                return None
    return v
```

Binary lifting is the default: simple, handles `k`-th ancestor for free, and $O(\log n)$ is fast enough almost always.

## Method 2: Euler tour + RMQ (O(n log n) build, O(1) query)

Record the **Euler tour**: the sequence of nodes visited by a DFS, appending $v$ every time you enter it *and* every time you return to it from a child. This array has length $2n - 1$; store `first[v]` = the first occurrence index of $v$, and `euler_depth[i]` = depth of the node at tour position $i$.

Then $\text{lca}(u, v)$ = the node of **minimum depth** in the tour segment between `first[u]` and `first[v]` — a [range minimum query](/citadel/cp/sparse-table). A sparse table over `euler_depth` gives $O(1)$ per LCA after $O(n \log n)$ preprocessing.

```python
def build_euler_lca(n, adj, root=0):
    euler = []
    depth_at = []
    first = [0] * n
    # iterative DFS producing the "enter, and re-enter after each child" tour
    stack = [(root, 0, iter(adj[root]))]
    d = {root: 0}
    seen = {root}
    euler.append(root); depth_at.append(0); first[root] = 0
    while stack:
        v, dv, it = stack[-1]
        advanced = False
        for to in it:
            if to not in seen:
                seen.add(to); d[to] = dv + 1
                first[to] = len(euler)
                euler.append(to); depth_at.append(dv + 1)
                stack.append((to, dv + 1, iter(adj[to])))
                advanced = True
                break
        if not advanced:
            stack.pop()
            if stack:
                euler.append(stack[-1][0])
                depth_at.append(stack[-1][1])
    # sparse table on depth_at storing argmin index
    return euler, depth_at, first    # then build an RMQ that returns the min-depth index
```

The $O(1)$ query wins when the number of queries is enormous ($\gg n \log n$); otherwise binary lifting's simplicity usually wins.

## Method 3: Tarjan's offline (O(n α(n)) total)

When **all queries are known up front**: one DFS with [union-find](/citadel/cp/disjoint-set-union). After fully processing a node $u$'s subtree, union $u$ into its parent's set and set that set's "ancestor" representative to the parent. When you finish node $u$, for every query $(u, w)$ with $w$ already finished, the answer is `find(w)`'s ancestor label. Near-linear total.

## Other methods

- **Farach-Colton and Bender:** the $\pm 1$ RMQ on the Euler tour has consecutive depths differing by exactly $1$, which allows an $O(n)$-build, $O(1)$-query RMQ (block decomposition + precomputed small-block tables). Theoretically optimal; rarely worth the code.
- **[RMQ ⟷ LCA equivalence](/citadel/cp/sparse-table):** general RMQ reduces to LCA on a Cartesian tree, and LCA reduces to $\pm 1$ RMQ — the two problems are interchangeable.

## Complexity

| method | build | query | notes |
| --- | --- | --- | --- |
| binary lifting | $O(n \log n)$ | $O(\log n)$ | + k-th ancestor; simplest |
| Euler + sparse RMQ | $O(n \log n)$ | $O(1)$ | best for very many queries |
| Tarjan offline | $O((n + q)\,\alpha(n))$ | — | all queries up front |
| Farach-Colton-Bender | $O(n)$ | $O(1)$ | heavy to implement |

## Common pitfalls

- **Root's ancestor.** Set `up[k][root] = root` (a self-loop) so lifting past the root is harmless; a `None` needs guarding everywhere.
- **Depth-align before the pairwise lift.** Method 1 must first bring both nodes to the same depth; skipping that gives wrong answers when depths differ.
- **Euler tour length.** It is $2n - 1$ (enter once per node, re-enter after each of its children). Off-by-one here breaks the RMQ range.
- **Recursion depth.** Iterative DFS for $n \gtrsim 10^5$ in Python.
- **Forest (multiple roots).** Add a virtual super-root, or handle "different trees ⇒ no LCA" explicitly.

## The keystone

`lca(u, v)`: lift the deeper node to equal depth, then jump both up in decreasing powers of two while their ancestors differ. That is binary lifting, $O(\log n)$ per query after an $O(n \log n)$ table. For $O(1)$ queries, reduce LCA to a range-minimum query on the Euler tour; if all queries are known offline, Tarjan's union-find pass is near-linear.
