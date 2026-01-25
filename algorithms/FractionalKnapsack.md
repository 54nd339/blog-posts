---
title: The Fractional Knapsack Problem - A Greedy Path to Maximum Value
description: Got a knapsack with limited capacity and a bunch of items, each with a value and weight? We explore the Fractional Knapsack problem and how a greedy algorithm, prioritizing value-to-weight ratio, finds the optimal solution.
date: 2023-02-15
draft: false
slug: /pensieve/algorithms/fractional-knapsack
tags:
  - DSA
  - Greedy
---

Hey everyone, and welcome back to the blog! Imagine you're at a grand buffet (perhaps one of the lavish Sunday brunches here in KIIT ), and you have a plate (your knapsack) with a limited capacity. There are all sorts of delicious dishes (items), each with a certain "tastiness" value and taking up a certain amount of space on your plate (weight). The best part? You can take fractions of dishes! Your goal is to fill your plate to maximize the total tastiness without overfilling it. This is the essence of the **Fractional Knapsack Problem**.

It's a classic combinatorial optimization problem where the aim is to fill a container with fractional amounts of different materials (or items) chosen to maximize the total value, subject to a weight constraint. Unlike its stricter cousin, the 0/1 Knapsack problem (where you must take an item whole or not at all), here we have the flexibility to take parts of items. Let's see how a simple greedy approach can solve this efficiently.

## The Problem: What (and How Much) to Pack?
Let's formally define the Fractional Knapsack Problem:

You are given:

* A set of $n$ items.
* For each item $i$:
  * It has a **weight** $w_i$.
  * It has a **value** $v_i$.
* A knapsack with a maximum **weight capacity** $W$.

The goal is to determine the *fraction* $x_i$ (where $0 \le x_i \le 1$) of each item $i$ to include in the knapsack such that:

1. The **total weight** of the selected fractions of items, $\sum (x_i \cdot w_i)$, is less than or equal to $W$.
2. The **total value** of the selected fractions of items, $\sum (x_i \cdot v_i)$, is as large as possible.

## The Greedy Strategy: Prioritize by Value Density

The beauty of the Fractional Knapsack problem is that it can be solved optimally using a greedy algorithm. The core idea is to prioritize items that give you the most "bang for your buck" – or, more formally, the highest value per unit of weight.

Here's the strategy:

1. **Calculate Value-to-Weight Ratios:** For each item, calculate its value-to-weight ratio ($v_i / w_i$). This represents how much value you get per unit of weight for that item.
2. **Sort by Ratio:** Sort all items in decreasing (descending) order based on these calculated ratios.
3. **Fill the Knapsack Greedily:** Iterate through the sorted items. For each item:
   * If the entire item can fit into the remaining capacity of the knapsack, take all of it. Add its full value to your total and decrease the knapsack's remaining capacity by the item's full weight.
   * If only a fraction of the item can fit, take as much as possible to fill the remaining capacity. Add the value of that fraction to your total, and the knapsack is now full.
   * Once the knapsack is full, you stop.

This approach ensures that you're always picking the "densest" value items first.

### Pseudocode for the Greedy Algorithm

Let's represent this logic with pseudocode:

```pseudocode
// items: A list of item objects, where each item has 'value' and 'weight' attributes.
// capacity: The maximum weight capacity of the knapsack.

function FractionalKnapsack(items, capacity):
    n = length(items)

    // 1. Calculate the value-to-weight ratio for each item
    //    (Let's assume this is added as a 'ratio' attribute to each item object)
    for i = 0 to n-1: // Using 0-based indexing for items array
        if items[i].weight > 0: // Avoid division by zero if an item has no weight
            items[i].ratio = items[i].value / items[i].weight
        else:
            items[i].ratio = infinity // Or handle as a special case (e.g., prioritize if value > 0)

    // 2. Sort items in decreasing order of their value-to-weight ratio
    sort items in decreasing order of items[i].ratio

    totalValue = 0
    currentWeight = 0

    // 3. Iterate through the sorted items and fill the knapsack
    for i = 0 to n-1:
        // If the knapsack is already full, we can stop
        if capacity == 0:
            break // Knapsack is full

        // Determine how much of the current item we can take
        // It's either the full weight of the item or the remaining capacity, whichever is smaller.
        amount_to_take = min(items[i].weight, capacity)
     
        // Add the value of the taken portion to the total value
        // (amount_to_take * items[i].ratio) is equivalent to (amount_to_take / items[i].weight) * items[i].value
        totalValue = totalValue + amount_to_take * items[i].ratio
     
        // Decrease the item's "remaining" weight (conceptually, for this iteration)
        // items[i].weight = items[i].weight - amount_to_take // Not strictly needed if we use amount_to_take
     
        // Decrease the knapsack's remaining capacity
        capacity = capacity - amount_to_take
         
    return totalValue
```

## Explanation of the Steps:

1. The algorithm starts by calculating the ratio of value to weight ($v_i / w_i$) for each item. This ratio represents the "value density" of each item.
2. Then, it sorts the items in decreasing order based on this calculated ratio. The items offering the most value per unit of weight come first.
3. The algorithm iterates through this sorted list of items. For each item, it tries to add as much of it as possible to the knapsack:
   * If the entire current item can fit within the remaining `capacity` of the knapsack (i.e., `items[i].weight <= capacity`), the whole item is taken. Its full value is added to `totalValue`, and `capacity` is reduced by `items[i].weight`.
   * If the entire current item cannot fit (i.e., `items[i].weight > capacity`), then only a fraction of the item is taken – just enough to fill the remaining `capacity`. The value added is `capacity * items[i].ratio` (which is `capacity * (items[i].value / items[i].weight)`). After this, the `capacity` becomes 0, and the knapsack is full.
4. The loop continues until either all items are considered or the knapsack's `capacity` becomes 0. The accumulated `totalValue` is then returned.

## Why Does This Greedy Strategy Work? Proof of Correctness

The correctness of the fractional knapsack algorithm, which relies on picking items with the highest value-to-weight ratio first, can be proven using an "exchange argument" or by showing it satisfies the greedy choice property and optimal substructure.

1. **Greedy Choice Property:**
   Consider any optimal solution. If this solution doesn't pick as much as possible of the item with the currently highest value-to-weight ratio (let's call this item $x$), we can show that we can always create an equally good or better solution by swapping out some amount of a lower-ratio item (say $y$) that is in the optimal solution with an equivalent weight of item $x$. Since item $x$ has a higher value density than item $y$, this swap would either maintain or increase the total value without exceeding the capacity. This implies that there's always an optimal solution that makes the greedy choice (takes as much as possible of the highest ratio item).
2. **Optimal Substructure:**
   After making the greedy choice (taking some amount of the highest ratio item), the problem reduces to a smaller fractional knapsack problem: fill the remaining capacity with the remaining items (or the remaining fraction of the current item). If our greedy choice is part of an optimal solution, then the way we fill the rest of the knapsack must also be optimal for that subproblem.

These properties ensure that the fractional knapsack algorithm, by always picking the "best" item locally (highest value-to-weight ratio), leads to a globally optimal solution. The algorithm selects items in decreasing order of their value-to-weight ratio, ensuring the most valuable items (per unit weight) are selected first. It then adds the maximum possible amount of each item, ensuring the knapsack is filled with the most valuable combination.

## Complexity Analysis

* **Time Complexity:** The most time-consuming step in the fractional knapsack algorithm is sorting the items by their value-to-weight ratio. If there are $n$ items, sorting takes O(n log n) time using an efficient sorting algorithm (like Merge Sort or Quick Sort). The subsequent loop to fill the knapsack iterates through the items once, taking O(n) time.
  Therefore, the overall time complexity is dominated by the sort: **O(n log n)**.
* **Space Complexity:** If we can modify the input items array to include the ratio or sort it in place, the additional space required is minimal, primarily for storing a few variables. Thus, the auxiliary space complexity can be considered **O(1)**. If we create a new list of items with ratios or a new list for sorting, it would be O(n).

## Applications

The fractional knapsack problem is a fundamental greedy algorithm problem with applications in various domains where:

* Resources are divisible.
* You want to maximize value given a constraint. Examples include:
* **Resource Allocation:** Allocating a divisible resource (like bandwidth, time, or raw materials) to various tasks or products to maximize profit or utility.
* **Financial Portfolio Optimization (Simplified):** Deciding how much of different divisible assets to include in a portfolio to maximize return for a given risk or budget.
* **Cutting Stock Problems:** Optimizing how to cut raw materials (like logs or fabric) into smaller pieces of varying values to maximize the total value, where fractional cuts are allowed.

## Key Takeaways

* The **Fractional Knapsack Problem** allows taking fractions of items to maximize total value within a weight capacity.
* It can be optimally solved using a **greedy algorithm**:
  1. Calculate the value-to-weight ratio for each item.
  2. Sort items by this ratio in descending order.
  3. Iterate through sorted items, taking as much of each item as possible (or a fraction to fill capacity) until the knapsack is full.
* The time complexity is **O(n log n)** due to sorting, and space complexity is typically O(1) or O(n) depending on implementation details.

This problem beautifully illustrates the power of greedy algorithms when the greedy choice property and optimal substructure hold!
