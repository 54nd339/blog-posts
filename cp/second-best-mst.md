---
title: Second-Best Minimum Spanning Tree - Swapping a Single Edge
description: The second-smallest spanning tree differs from the MST by exactly one edge swap, so try adding each non-tree edge and removing the heaviest edge on the cycle it closes, found via max-on-path queries.
date: 2024-09-20
draft: false
slug: /cp/second-best-mst
tags:
  - Competitive Programming
  - Graphs
  - Minimum Spanning Tree
---

The second-best spanning tree — the one with the smallest total weight strictly greater than the [MST](/citadel/algorithms/MinimumSpanningTree)'s (or, in some phrasings, $\le$, allowing ties) — differs from the MST by **exactly one edge**. So: build the MST, then for each non-tree edge $e$, adding it creates a cycle; removing the heaviest *other* edge on that cycle gives a spanning tree. The best such swap is the answer.

## The problem

Given a connected weighted graph, find a spanning tree $T'$ minimising $w(T')$ subject to $T' \ne T_{\text{MST}}$ (or $w(T') > w(T_{\text{MST}})$, if the problem wants a *strictly* heavier tree).

Example: a triangle with edge weights $1, 2, 3$. MST = $\{1, 2\}$, weight $3$. Second-best swaps in the $3$-edge and drops the $2$: $\{1, 3\}$, weight $4$.

## The idea

Any spanning tree $T'$ can be obtained from the MST $T$ by a sequence of "add one non-tree edge, remove one tree edge on the resulting cycle" swaps, each of which does not decrease the weight. The **minimum-weight** such swap changes exactly one edge. So:

For each non-tree edge $e = (u, v, w_e)$: adding $e$ to $T$ closes a cycle consisting of $e$ plus the tree path $u \leadsto v$. Removing the **heaviest edge $f$ on that path** gives a valid spanning tree of weight $w(T) - w_f + w_e$. Since $T$ is an MST, $w_f \le w_e$, so this is $\ge w(T)$.

- For a **strictly** heavier tree, only consider swaps with $w_f < w_e$ (if $w_f = w_e$ the swap gives the same weight — a different tree of equal weight; include or exclude per the problem).
- The answer is $\min_e \big(w(T) - w_f(e) + w_e\big)$.

The per-edge query is "**maximum edge weight on the tree path between $u$ and $v$**". With $m$ non-tree edges, you want this fast.

## Computing max-on-path

- **Binary lifting** (same table as [LCA](/citadel/cp/lowest-common-ancestor)): store `up[v][k]` = the $2^k$-th ancestor and `maxw[v][k]` = the max edge weight on that jump. Path max is the LCA query, combining `maxw` along the way. $O(n \log n)$ build, $O(\log n)$ per query.
- For "strictly heavier", also track the **second-largest distinct** edge weight on the path, so that when $w_f = w_e$ you can fall back to the next one.

```python
def second_best_mst(n, edges):
    # edges: list of (w, u, v). Returns (mst_weight, second_best_weight_or_None)
    order = sorted(range(len(edges)), key=lambda i: edges[i][0])
    par = list(range(n))
    def find(x):
        while par[x] != x:
            par[x] = par[par[x]]; x = par[x]
        return x

    tree = [[] for _ in range(n)]
    in_mst = [False] * len(edges)
    mst_w = 0
    for i in order:
        w, u, v = edges[i]
        if find(u) != find(v):
            par[find(u)] = find(v)
            in_mst[i] = True
            mst_w += w
            tree[u].append((v, w))
            tree[v].append((u, w))

    from collections import deque

    def heaviest_edge_below(u, v, limit):
        # max weight edge on the tree path u..v that is strictly < limit; None if none
        prev = {u: (None, None)}
        dq = deque([u])
        while dq:
            x = dq.popleft()
            if x == v:
                break
            for to, w in tree[x]:
                if to not in prev:
                    prev[to] = (x, w)
                    dq.append(to)
        best = None
        cur = v
        while prev[cur][0] is not None:
            p, w = prev[cur]
            if w < limit and (best is None or w > best):
                best = w
            cur = p
        return best

    ans = None
    for i, (w, u, v) in enumerate(edges):
        if in_mst[i]:
            continue
        f = heaviest_edge_below(u, v, w)          # swap e in, that edge out
        if f is not None:
            cand = mst_w - f + w
            ans = cand if ans is None else min(ans, cand)
    return mst_w, ans
```

The BFS path walk is $O(n)$ per non-tree edge, so this is $O(mn)$ — fine up to a few thousand vertices. To reach $O((n + m)\log n)$, replace `heaviest_edge_below` with a **binary-lifting** query: root the MST, store `up[v][k]` (the $2^k$-th ancestor), `mx1[v][k]` (largest edge weight on that jump) and `mx2[v][k]` (largest *distinct* weight strictly below `mx1`). The [LCA](/citadel/cp/lowest-common-ancestor)-style walk then returns the heaviest path edge $< w_e$ in $O(\log n)$, using `mx2` when the top weight equals $w_e$.

## Complexity

- **Time:** $O(m \log m)$ for the MST, $O(n \log n)$ to build the lifting tables, $O(m \log n)$ for the per-edge queries. Total $O((n + m)\log n)$.
- **Space:** $O(n \log n)$.

## Common pitfalls

- **"Strictly heavier" vs "different tree".** If ties are allowed, use the plain path max ($\le w_e$); for strictly greater weight, you need the heaviest path edge **$< w_e$**, hence the second-largest-distinct table.
- **No valid swap.** If every non-tree edge's cycle has all edges equal to it, there is no strictly-heavier spanning tree; report accordingly.
- **Path max, not path sum.** You remove *one* edge — the maximum on the cycle path — not the whole path.
- **Multigraph / repeated weights.** Track distinct weights for the strict version; two parallel MST-weight edges give a same-weight second tree.
- **Disconnected input.** No spanning tree at all; handle before running.

## The keystone

The second-best spanning tree is one edge swap away from the MST: for each non-tree edge $e$, adding it forms a cycle, and removing the heaviest tree edge on that cycle (strictly lighter than $e$, for a strictly heavier tree) yields a candidate. Max-on-path via binary lifting makes all $m$ candidates cost $O(m \log n)$.
