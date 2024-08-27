---
title: Finding the Faces of a Planar Graph - Walking Angular Order
description: Recovering the faces of a planar graph embedded in the plane by sorting each vertex's edges by angle and repeatedly walking "next edge clockwise" around the boundary of each face.
date: 2024-08-27
draft: false
slug: /cp/planar-graph-faces
tags:
  - Competitive Programming
  - Geometry
  - Graphs
---

Given a graph drawn in the plane — vertices at coordinates, edges as straight segments that only meet at vertices — its **faces** are the connected regions the drawing cuts the plane into (including the one unbounded outer face). You can list them all in $O(E \log E)$ by treating each undirected edge as two directed half-edges and repeatedly following "turn as far clockwise as possible" until you return to the start.

## The problem

Input: a planar embedding — vertices with $(x, y)$ coordinates and an adjacency list. Output: each face as the cyclic sequence of vertices (or half-edges) on its boundary. By **Euler's formula** $V - E + F = 1 + C$ (with $C$ connected components), so $F$ is known in advance as a sanity check.

## The idea

Split each undirected edge $\{u, v\}$ into two **directed half-edges** $u \to v$ and $v \to u$. Every half-edge belongs to exactly one face — the one on its left as you walk along it. To trace a face:

1. Start at an unused half-edge $u \to v$.
2. At $v$, among all edges leaving $v$, take the one that is the **next clockwise** from the reverse edge $v \to u$ — i.e. sort $v$'s outgoing edges by polar angle, find $v \to u$, step to the previous entry (wrapping).
3. That gives the next half-edge $v \to w$. Continue until you return to the starting half-edge.

Mark half-edges used as you go. Repeat from any unused half-edge. Each half-edge is visited once ⇒ $O(E)$ after an $O(E \log E)$ angular sort per vertex.

The face with the **most negative signed area** (or, equivalently, negative area under CCW convention) is the outer face.

## Algorithm

```python
import math

def planar_faces(coords, adj):
    # coords[v] = (x, y);  adj[v] = list of neighbours
    # sort each vertex's neighbours by angle
    order = {}
    for v, nbrs in enumerate(adj):
        s = sorted(nbrs, key=lambda u: math.atan2(coords[u][1] - coords[v][1],
                                                  coords[u][0] - coords[v][0]))
        order[v] = {u: i for i, u in enumerate(s)}
        adj[v] = s

    used = set()
    faces = []
    for u in range(len(adj)):
        for v in adj[u]:
            if (u, v) in used:
                continue
            face = []
            a, b = u, v
            while (a, b) not in used:
                used.add((a, b))
                face.append(a)
                # at b, take the edge just clockwise from (b -> a)
                i = order[b][a]
                nxt = adj[b][(i - 1) % len(adj[b])]
                a, b = b, nxt
            faces.append(face)
    return faces

def signed_area(coords, face):
    s = 0
    for i in range(len(face)):
        x1, y1 = coords[face[i]]
        x2, y2 = coords[face[(i + 1) % len(face)]]
        s += x1 * y2 - x2 * y1
    return s / 2
```

The outer face is the one with `signed_area` negative (traversed clockwise) — or the one with the largest $|{\text{area}}|$ if the embedding is a simple subdivision.

## Uses

- **Point location by face**, together with a [sweep-line locator](/citadel/cp/point-location).
- **Counting regions** a set of segments/lines cuts the plane into — build the arrangement graph (intersection points become vertices), then count faces via Euler.
- **Doubly-connected edge list (DCEL)** construction — this "next clockwise" walk is exactly how a DCEL's face records are populated.
- **Planar dual graph** — one dual vertex per face, dual edges across each primal edge; needed for planar max-flow / min-cut tricks.
- **Detecting whether an embedding is a valid planar subdivision** — face count must match Euler.

## Common pitfalls

- **Direction of the turn.** "Next clockwise from the reverse edge" walks the face on the **left** of each half-edge. Taking the counter-clockwise neighbour instead traces the same faces with opposite orientation — pick one and be consistent, or your inner faces and outer face swap sign.
- **`atan2` ties.** Overlapping edges (two neighbours in the same direction) mean the embedding is degenerate; the input should not have them.
- **Self-loops and multi-edges.** Handle each half-edge instance separately; a multi-edge contributes two half-edges per parallel copy, and a self-loop is its own tiny face.
- **Isolated vertices / bridges.** A bridge (edge whose removal disconnects) is traversed once in each direction *within the same face* — the face walk visits it twice. That is correct; do not dedupe vertices in a face.
- **Euler check.** $V - E + F = 1 + C$. If your $F$ is off by one, you probably merged or split the outer face.

## The keystone

Every directed half-edge bounds one face on its left; tracing a face means "walk the edge, then at the far vertex turn as far clockwise as possible" until you loop. Sort each vertex's edges by angle once, and all faces come out in $O(E \log E)$, with the negative-signed-area face being the outer one.
