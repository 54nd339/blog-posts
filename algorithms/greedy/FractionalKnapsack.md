---
title: Fractional Knapsack - Greedy by Value-to-Weight Ratio
description: Fill a capacity-limited knapsack to maximize value when you may take fractions of items. Sorting by value per unit weight and filling greedily is optimal, which is exactly what fails for the all-or-nothing 0/1 version.
date: 2021-03-04
draft: false
slug: /algorithms/FractionalKnapsack
tags:
  - Algorithms
  - Greedy
---

Same setup as [0/1 knapsack](/citadel/algorithms/01Kanpsack) — items with weights $w_i$ and values $v_i$, a capacity $W$, maximize value — but now you may take *any fraction* $x_i \in [0, 1]$ of each item. That one change collapses an NP-complete problem into a sort.

## The rule

Take the most valuable stuff per kilogram first. Compute each item's value density $v_i / w_i$, sort descending, and walk the list: take each item whole while it fits, then take a fraction of the next one to top off the capacity exactly, and stop.

With items $(v, w) = (60, 10), (100, 20), (120, 30)$ and capacity 50, the densities are 6, 5, 4. Take item 1 whole (value 60, 40 capacity left), item 2 whole (value 100, 20 left), then $20/30$ of item 3 for $120 \cdot \tfrac{2}{3} = 80$. Total 240 — better than any whole-item selection.

## The code

```python
def fractional_knapsack(items, capacity):     # items: list of (value, weight)
    total = 0.0
    for value, weight in sorted(items, key=lambda it: it[0] / it[1], reverse=True):
        if capacity <= 0:
            break
        take = min(weight, capacity)
        total += value * (take / weight)      # value of the fraction taken
        capacity -= take
    return total


# ratios: 60/10=6, 100/20=5, 120/30=4  -> take items 1, 2 whole, 2/3 of item 3
assert fractional_knapsack([(60, 10), (100, 20), (120, 30)], 50) == 240.0
```

## Why it's optimal

An **exchange argument**. Suppose an optimal solution takes less than the maximum possible of the highest-ratio item $x$, and therefore takes some amount of a lower-ratio item $y$. Swap a unit of weight of $y$ for a unit of weight of $x$: the total weight is unchanged, and because $x$ has the higher value density, the total value goes up or stays the same. Repeating drives the solution to the greedy one without ever losing value — so greedy is optimal.

This is exactly the step that fails for **0/1 knapsack**. There you can't take "a unit of weight" of an item; it's all or nothing, so the swap isn't available and the highest-ratio item might not belong in the optimal set at all. That's why 0/1 needs a DP table and this one needs a sort.

## Cost

Sorting by ratio is **$O(n \log n)$**; the fill loop is $O(n)$. Space is $O(1)$ beyond the input if you sort in place, $O(n)$ otherwise. (A selection-based approach can even do it in $O(n)$ expected time, since you don't truly need the items fully sorted — only partitioned around the fill point.)

## The takeaway

Fractional knapsack earns its place next to 0/1 knapsack as the contrast case: divisibility is what makes the greedy exchange argument go through. When items are divisible and you're maximizing value under one linear constraint, sort by the ratio. When they aren't, reach for [dynamic programming](/citadel/algorithms/01Kanpsack). The exchange-argument style of proof is shared with [activity selection](/citadel/algorithms/ActivitySelection) and [Huffman coding](/citadel/algorithms/HuffmanCoding).
