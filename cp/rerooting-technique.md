---
title: The Rerooting Technique - One DP That Answers Every Root
description: Computing a subtree dynamic program for all n choices of root in O(n) total by pushing a "rest of the tree" value down from each parent, with prefix and suffix aggregates to exclude one child at a time.
date: 2024-10-24
draft: false
slug: /cp/rerooting-technique
tags:
  - Competitive Programming
  - Graphs
  - Trees
---

A rooted tree DP computes some value for the root from its subtrees. Sometimes you need that value for **every** vertex as the root — sum of distances to all others, longest path starting here, number of colourings with this vertex fixed. Re-running the DP $n$ times is $O(n^2)$. The rerooting technique gets all $n$ answers in $O(n)$ by computing, for each vertex, one extra value: the contribution of everything *outside* its subtree.

## The problem

Root the tree anywhere and let $f(v)$ be a DP over $v$'s subtree that combines the children's values with an **associative, commutative** merge (a monoid): $f(v) = \big(\bigoplus_{c \in \text{child}(v)} g(f(c))\big)$, where $g$ lifts a child's value across the edge. Goal: for every $v$, the value of $f$ if the tree were rooted at $v$.

Concrete example — **sum of distances**. `down_cnt[v]` = size of $v$'s subtree, `down_dist[v]` = sum of distances from $v$ to its subtree. For the fixed root $r$, `ans[r] = down_dist[r]`. On a path graph $0\text{-}1\text{-}2\text{-}3$ rooted at $0$: `down_dist = [6, 3, 1, 0]`, so `ans[0] = 6`; the answers for all four roots are $6, 4, 4, 6$.

## The idea: down pass, then up pass

**Down pass** (post-order from the fixed root): compute `down[v]` from the children, exactly the ordinary subtree DP.

**Up pass** (pre-order): maintain `up[v]` = the DP value of the component that hangs off $v$ *through its parent* — everything not in $v$'s subtree, folded up to $v$. Then the answer rooted at $v$ is `combine(down[v], up[v])`.

To pass `up` from a vertex $u$ to a particular child $c$, $u$ combines:

- its own `up[u]` (the part above $u$), lifted across the $u$–$c$ edge, and
- `g(down[c'])` for every **other** child $c' \ne c$.

The "every other child" is the catch. If the merge has an inverse you can compute the full product and divide out $g(\text{down}[c])$. If it does not (max, min, gcd), precompute **prefix and suffix aggregates** over the child list so excluding index $i$ is `merge(prefix[i], suffix[i+1])` in $O(1)$. Either way each edge is processed once: $O(n)$.

## How it works: sum of distances

Rooting at $r$, moving the root to an adjacent child $c$ flips the $r$–$c$ edge. Every vertex in $c$'s subtree gets one closer ($down\_cnt[c]$ of them); every other vertex gets one farther ($n - down\_cnt[c]$ of them). So

$$\text{ans}[c] = \text{ans}[r] - down\_cnt[c] + (n - down\_cnt[c]).$$

That is the rerooting recurrence in closed form — no prefix/suffix needed because the update is arithmetic and reversible.

```python
from collections import deque

def sum_of_distances(n, adj):
    order, parent = [], [-1] * n
    seen = [False] * n
    dq = deque([0]); seen[0] = True
    while dq:
        u = dq.popleft(); order.append(u)
        for w in adj[u]:
            if not seen[w]:
                seen[w] = True; parent[w] = u; dq.append(w)

    cnt = [1] * n
    dist = [0] * n
    for u in reversed(order):                     # down pass
        for w in adj[u]:
            if w != parent[u]:
                cnt[u] += cnt[w]
                dist[u] += dist[w] + cnt[w]

    ans = [0] * n
    ans[0] = dist[0]
    for u in order:                               # up pass (reroot along each edge)
        for w in adj[u]:
            if w != parent[u]:
                ans[w] = ans[u] - cnt[w] + (n - cnt[w])
    return ans
```

## The general template (no inverse: prefix/suffix)

For a monoid without an inverse — say each vertex wants the **maximum distance to any other vertex** (its eccentricity) — exclude one child with prefix/suffix maxima:

```python
def eccentricity(n, adj):
    order, parent = [], [-1] * n
    seen = [False] * n
    dq = deque([0]); seen[0] = True
    while dq:
        u = dq.popleft(); order.append(u)
        for w in adj[u]:
            if not seen[w]:
                seen[w] = True; parent[w] = u; dq.append(w)
    child = [[w for w in adj[u] if w != parent[u]] for u in range(n)]

    down = [0] * n
    for u in reversed(order):
        down[u] = max((down[w] + 1 for w in child[u]), default=0)

    up = [0] * n
    ans = [0] * n
    for u in order:
        ch = child[u]
        vals = [down[w] + 1 for w in ch]         # contribution of each child, seen from u
        k = len(ch)
        pref = [0] * (k + 1)
        suf = [0] * (k + 1)
        for i in range(k):
            pref[i + 1] = max(pref[i], vals[i])
        for i in range(k - 1, -1, -1):
            suf[i] = max(suf[i + 1], vals[i])
        for i, w in enumerate(ch):
            best_sibling = max(pref[i], suf[i + 1])   # excludes child i
            up[w] = max(up[u] + 1, best_sibling + 1)
        ans[u] = max(down[u], up[u])
    return ans
```

## Complexity

- **Time:** $O(n)$ — two traversals; the prefix/suffix arrays cost $O(\deg u)$ per vertex, summing to $O(n)$.
- **Space:** $O(n)$.

## Common pitfalls

- **Non-associative or non-commutative merge.** The technique needs a monoid. If order matters, the "exclude one child" step is ill-defined.
- **Excluding the child via subtraction when there is no inverse.** `full - g(down[c])` only works for `+`, `xor`, counting mod $p$, etc. For `max`/`min`/`gcd`/`or` use prefix/suffix aggregates.
- **The identity element.** `prefix[0]` and `suffix[k]` must be the monoid identity ($0$ for sum, $-\infty$ or $0$ for a max of non-negative depths, $1$ for product). A wrong identity corrupts every leaf-adjacent exclusion.
- **Forgetting the edge lift $g$.** `up[u]` is the value *at $u$*; before handing it to child $c$ you must lift it across the edge (add $1$ to a depth, multiply by a transition weight, etc.).
- **Recursion depth.** For $n \sim 10^5$ a recursive down pass overflows Python's stack — use the explicit BFS order shown here and iterate it in reverse.

## The keystone

Do the subtree DP once to get `down[v]`, then a second pass carrying `up[v]` — the DP value of the tree minus $v$'s subtree, folded to $v$. The answer for root $v$ is `merge(down[v], up[v])`, and passing `up` to each child needs "all other children combined", which is a subtraction when the merge has an inverse and a prefix/suffix aggregate when it does not. Total cost $O(n)$ for all $n$ roots.
