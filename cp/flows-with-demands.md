---
title: Flows With Demands - Lower Bounds on Edges
description: Handling networks where each edge must carry at least a given amount of flow - the per-vertex excess transformation that turns lower bounds into a plain max-flow feasibility test, and how to then extract the minimal or maximal feasible s-t flow.
date: 2024-10-14
draft: false
slug: /cp/flows-with-demands
tags:
  - Competitive Programming
  - Graphs
  - Network Flow
---

Ordinary [max flow](/citadel/cp/maximum-flow) has capacities $0 \le f(e) \le c(e)$. Many modelling problems instead impose a **lower bound**: edge $e$ must carry at least $d(e)$ units. "Every shift needs at least two people", "each pipe must stay above a minimum pressure flow" — these are flows with demands. A single transformation reduces them to a feasibility max flow.

## The problem

Each edge $e = (u, v)$ has a lower bound $d(e)$ and an upper bound $c(e)$, with $0 \le d(e) \le c(e)$. Find a flow $f$ with $d(e) \le f(e) \le c(e)$ on every edge and conservation at every vertex (a **feasible circulation**), or with a designated source $s$ and sink $t$ (a **feasible $s$-$t$ flow**). Then optionally minimise or maximise the $s$-$t$ flow value.

Example: a triangle $0 \to 1 \to 2 \to 0$ with bounds $[1, 3]$, $[1, 3]$, $[1, 3]$. Sending $1$ unit around the cycle is a feasible circulation; $3$ around is the maximum.

## The idea: route the lower bounds for free, then patch the excess

Split every edge's flow as $f(e) = d(e) + f'(e)$ where $0 \le f'(e) \le c(e) - d(e)$. The $d(e)$ part is *mandatory* — imagine it is already flowing. That mandatory flow violates conservation: at each vertex $v$ it delivers $\text{in}_d(v) = \sum_{e \to v} d(e)$ and removes $\text{out}_d(v) = \sum_{v \to e} d(e)$.

Define the **excess** $\text{exc}(v) = \text{in}_d(v) - \text{out}_d(v)$. Build a new network on the same vertices with:

- edge $u \to v$ of capacity $c(e) - d(e)$ for every original edge (this carries $f'$);
- a **super-source** $S$ with an edge $S \to v$ of capacity $\text{exc}(v)$ whenever $\text{exc}(v) > 0$;
- a **super-sink** $T$ with an edge $v \to T$ of capacity $-\text{exc}(v)$ whenever $\text{exc}(v) < 0$.

A feasible circulation exists **iff the max flow $S \to T$ saturates every super-source edge** (equivalently, every super-sink edge — the two capacities sum to the same total). The real flow on each original edge is then $f'(e) + d(e)$.

## From circulation to a feasible s-t flow

If the problem has a real source $s$ and sink $t$ and no conservation requirement there, add one edge $t \to s$ with lower bound $0$ and capacity $\infty$. Now *every* vertex must conserve, so it is a pure circulation problem — solve it as above. The flow that ends up on the $t \to s$ edge is the $s$-$t$ flow value.

## Minimal and maximal feasible s-t flow

Both start from the circulation feasibility test; they differ in *when* the $t \to s$ edge is added.

- **Maximal:** add $t \to s$ with capacity $\infty$ **before** the feasibility max flow. Run max flow $S \to T$; if infeasible, stop. Record `val` = flow on $t \to s$, then delete the $t \to s$ edge entirely (both directions) and add `max_flow(s, t)` in the residual graph.
- **Minimal:** run max flow $S \to T$ **without** the $t \to s$ edge first. Then add $t \to s$ with capacity $\infty$ and continue the max flow $S \to T$. If the super-source edges are now all saturated it is feasible, and the flow on $t \to s$ is the minimal value.

## Algorithm

```python
INF = float("inf")

def _build(n, edges):
    # edges: (u, v, lo, hi). Returns a Dinic on n+2 nodes, S=n, T=n+1, and 'need'.
    S, T = n, n + 1
    din = Dinic(n + 2)
    excess = [0] * n
    for (u, v, lo, hi) in edges:
        din.add_edge(u, v, hi - lo)
        excess[v] += lo
        excess[u] -= lo
    need = 0
    for v in range(n):
        if excess[v] > 0:
            din.add_edge(S, v, excess[v])
            need += excess[v]
        elif excess[v] < 0:
            din.add_edge(v, T, -excess[v])
    return din, S, T, need

def feasible_circulation(n, edges):
    din, S, T, need = _build(n, edges)
    if din.max_flow(S, T) != need:
        return None
    flows = []
    for i, (u, v, lo, hi) in enumerate(edges):
        flows.append(lo + din.flow_on_edge(i))   # f = d(e) + f'(e)
    return flows

def max_feasible_st_flow(n, edges, s, t):
    din, S, T, need = _build(n, edges)
    ts = din.add_edge(t, s, INF)
    if din.max_flow(S, T) != need:
        return None
    val = din.flow_on(ts)
    din.remove_edge(ts)                           # both directions
    return val + din.max_flow(s, t)

def min_feasible_st_flow(n, edges, s, t):
    din, S, T, need = _build(n, edges)
    got = din.max_flow(S, T)
    ts = din.add_edge(t, s, INF)
    got += din.max_flow(S, T)                     # continue on the same residual graph
    if got != need:
        return None
    return din.flow_on(ts)
```

(`add_edge` returns a handle; `flow_on` reads the reverse-edge residual; `remove_edge` zeroes both the forward and reverse residuals; `flow_on_edge(i)` indexes the $i$-th original edge — thin wrappers over the [Dinic template](/citadel/cp/maximum-flow).)

## Complexity

One or two max-flow computations on a graph with $V + 2$ vertices and $E + V$ edges: $O(V^2 E)$ with Dinic, far less in practice.

## Common pitfalls

- **Excess sign.** $\text{exc}(v) = (\text{lower bounds into } v) - (\text{lower bounds out of } v)$. The rule is: **$S \to v$ with capacity $\text{exc}(v)$ when $\text{exc}(v) > 0$**, and $v \to T$ with capacity $-\text{exc}(v)$ when it is negative. Swapping the two directions makes every instance report infeasible.
- **Feasibility check.** "Feasible" is *max flow equals the total super-source capacity*, not "max flow is positive". A single unsaturated demand edge means no feasible flow exists.
- **Removing $t \to s$ for the maximal case.** Zero the reverse residual too. Leaving it lets the follow-up `max_flow(s, t)` cancel circulation flow through the $s \to t$ reverse arc and overcount.
- **Reusing the residual graph for the minimal case.** The second `max_flow(S, T)` must continue on the *same* Dinic object, not a fresh one — you are adding the $t \to s$ edge to an already-partly-saturated network.
- **Reconstructing the real flow.** It is $f'(e) + d(e)$, never just $f'(e)$. Forgetting the lower bound gives a flow that violates the lower bound everywhere it mattered.

## The keystone

Pre-pay every edge's lower bound as mandatory flow, measure the conservation it breaks as a per-vertex excess, and hire a super-source/super-sink to repair exactly that excess. A feasibility max flow that saturates all excess edges yields a feasible circulation; splicing in a $t \to s$ edge before or after that max flow gives the maximal or minimal feasible $s$-$t$ flow.
