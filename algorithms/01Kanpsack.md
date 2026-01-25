---
title: The 0/1 Knapsack Problem - A Classic Guide to Maximizing Value with Limited Capacity
description: Dive into the 0/1 Knapsack problem, a classic combinatorial puzzle solved with dynamic programming. Explore its problem statement, iterative and recursive solutions (with memoization), and understand its time and space complexity.
date: 2023-02-16
draft: false
slug: /pensieve/algorithms/01-knapsack
tags:
  - DSA
  - DP
---

Hey everyone, and welcome back to the blog! Imagine you're a hiker preparing for a trip, but you have a knapsack with a limited weight capacity. You're presented with a variety of items, each with its own weight and value (how useful or precious it is to you). Your challenge? To choose which items to pack to maximize the total value you carry, without exceeding your knapsack's weight limit. And here's the catch: for each item, you can either take it (1) or leave it (0) – you can't take fractions of items. This is the essence of the **0/1 Knapsack Problem**.

This classic problem in combinatorial optimization isn't just for hikers; it arises in many real-world resource allocation scenarios. Whether you're deciding which projects to invest in with a limited budget, or which features to include in a software release with limited development time, the underlying logic is similar. Today, let's unpack this problem and explore how Dynamic Programming offers an elegant way to solve it.

## The Problem: What to Pack?
Let's formally define the 0/1 Knapsack Problem:

You are given a set of $n$ items. For each item $i$:
* It has a **weight** $w_i$.
* It has a **value** $v_i$.

You are also given a knapsack with a maximum **weight capacity** $W$.

The goal is to determine which items to include in the knapsack such that:
1.  The **total weight** of the selected items is less than or equal to $W$.
2.  The **total value** of the selected items is as large as possible.
Importantly, for each item, you can either include it entirely in the knapsack or not include it at all—you cannot take a fraction of an item (hence "0/1").

This problem often arises in resource allocation where the decision-maker has to decide which items to include in a limited-capacity "knapsack" to maximize the total value of the items included.

## Solving with Dynamic Programming: The Iterative (Bottom-Up) Approach

The 0/1 Knapsack problem has an optimal substructure and overlapping subproblems, making it a perfect candidate for **Dynamic Programming (DP)**. The iterative approach builds up a solution from smaller subproblems.

We use a 2D array, let's call it `dp[i][w]`, to store the maximum value that can be obtained using the first `i` items with a maximum allowed weight of `w`.

### Pseudocode: Iterative DP

```pseudocode
function knapsack_01_iterative(weights, values, W_capacity):
    n = length(weights) // Number of items

    // dp[i][w] will store the maximum value that can be achieved
    // using the first 'i' items with a weight limit of 'w'.
    // Dimensions: (n+1) rows for items (0 to n), (W_capacity+1) columns for weights (0 to W_capacity)
    dp = array of size (n+1) x (W_capacity+1)

    // Build table dp[][] in bottom-up manner
    for i = 0 to n: // Iterate through items (or number of items considered)
        for w = 0 to W_capacity: // Iterate through possible knapsack capacities
            if i == 0 or w == 0:
                // If there are no items (i=0) or no capacity (w=0),
                // the maximum value is 0.
                dp[i][w] = 0
            else if weights[i-1] <= w:
                // If the weight of the current item (weights[i-1] because items are 0-indexed)
                // is less than or equal to the current capacity 'w':
                // We have two choices:
                // 1. Include the current item:
                //    Value = values[i-1] + dp[i-1][w - weights[i-1]]
                //    (value of current item + max value from remaining capacity with previous items)
                // 2. Exclude the current item:
                //    Value = dp[i-1][w]
                //    (max value without including the current item, using previous items)
                dp[i][w] = max(values[i-1] + dp[i-1][w - weights[i-1]],  // Option 1: Include item i
                               dp[i-1][w])                             // Option 2: Exclude item i
            else:
                // If the weight of the current item is more than the current capacity 'w',
                // we cannot include it. So, the max value is the same as the max value
                // without this item (using previous i-1 items).
                dp[i][w] = dp[i-1][w]
   
    // The final answer is stored in dp[n][W_capacity]
    return dp[n][W_capacity]
```

### Explanation of the Iterative Approach:
1. We create a 2D table `dp` of size `(n+1) x (W+1)`, where `n` is the number of items and `W` is the knapsack capacity. `dp[i][w]` will store the maximum value achievable using a subset of the first `i` items with a total weight limit of `w`.
2. Base Cases:
    * `dp[0][w] = 0 for all w:` If there are no items, the value is 0, regardless of capacity.
    * `dp[i][0] = 0 for all i:` If the knapsack has no capacity, no items can be included, so the value is 0. These are covered by the `if i == 0 or w == 0: dp[i][w] = 0` condition.
3. Filling the DP Table: We iterate through each item i (from 1 to n) and for each possible weight capacity w (from 1 to W). For each cell dp[i][w]:
    * If the current item `i` (whose weight is `weights[i-1]` and value is `values[i-1]`) is heavier than the current capacity `w`: We cannot include this item. So, the maximum value `dp[i][w]` is the same as the maximum value we could get with the previous i-1 items at the same capacity w (i.e., dp[i-1][w]).
    * If the current item `i` can fit into the knapsack (`weights[i-1] <= w`): We have two choices:
        * Exclude item `i`: The maximum value is what we could achieve with the previous `i-1` items and the same capacity `w` (i.e., `dp[i-1][w]`).
        * Include item `i`: The value of this item (`values[i-1]`) is added. The remaining capacity becomes `w - weights[i-1]`. We then find the maximum value achievable with the previous `i-1` items for this remaining capacity (i.e., `values[i-1] + dp[i-1][w - weights[i-1]]`). We take the maximum of these two choices for `dp[i][w]`.
4. The value in `dp[n][W]` will be the maximum value that can be obtained using all n items with the full knapsack capacity `W`.

## Solving with Dynamic Programming: The Recursive Approach with Memoization (Top-Down)

Alternatively, we can solve the 0/1 Knapsack problem using recursion with memoization, which is often more intuitive to derive from the problem's recursive nature.

### Pseudocode: Recursive DP with Memoization

```pseudocode
// Main function to initialize memoization table and call the helper
function knapsack_01_recursive(weights, values, W_capacity):
    n = length(weights)
    // memo[i][w] will store the result for knapsack_01_memoization(weights, values, w, i, memo)
    // Initialize with a value indicating "not yet computed", e.g., -1
    memo = array of size (n+1) x (W_capacity+1) filled with -1
   
    return knapsack_01_memoization_helper(weights, values, W_capacity, n, memo)

// Recursive helper function with memoization
function knapsack_01_memoization_helper(weights, values, W, i, memo):
    // Base Case: No items left (i=0) or no capacity left (W=0)
    if i == 0 or W == 0:
        return 0
   
    // If this subproblem has already been solved, return the stored result
    if memo[i][W] != -1:
        return memo[i][W]
   
    // If the weight of the i-th item (weights[i-1] due to 0-indexing of arrays
    // vs 1-to-n thinking for 'i' items) is greater than the remaining capacity W,
    // then this item cannot be included.
    if weights[i-1] > W:
        memo[i][W] = knapsack_01_memoization_helper(weights, values, W, i-1, memo)
    else:
        // Otherwise, consider two cases:
        // 1. Include the i-th item:
        //    value = values[i-1] + result from (i-1) items and capacity (W - weights[i-1])
        // 2. Exclude the i-th item:
        //    value = result from (i-1) items and capacity W
        include_item_value = values[i-1] + knapsack_01_memoization_helper(weights, values, W - weights[i-1], i-1, memo)
        exclude_item_value = knapsack_01_memoization_helper(weights, values, W, i-1, memo)
       
        memo[i][W] = max(include_item_value, exclude_item_value)
       
    return memo[i][W]
```

### Explanation of the Recursive Approach:
1. The main function initializes a memoization table memo (e.g., with -1s) to store results of subproblems.
2. The recursive helper function `knapsack_01_memoization_helper(weights, values, W, i, memo)` calculates the maximum value considering items from `1` to `i` with capacity `W`.
3. **Base Cases:** If there are no items left (`i == 0`) or no knapsack capacity (`W == 0`), the maximum value is 0.
4. **Memoization Check:** Before computing, it checks if `memo[i][W]` already contains a valid result. If so, it returns it immediately, avoiding redundant calculations.
5. Recursive Step:
    * If the current item `i` (index `i-1` in arrays) is too heavy (`weights[i-1] > W`), it cannot be included. So, the result is the same as solving for `i-1` items with capacity `W`.
    * Otherwise, we consider two options:
        * Include item `i`: The value is `values[i-1]` plus the result of the subproblem with `i-1` items and remaining capacity `W - weights[i-1]`.
        * Exclude item `i`: The value is the result of the subproblem with `i-1` items and the same capacity `W`. We choose the max of these two options.
6. The result is stored in memo[i][W] before being returned.

While the recursive approach with memoization might feel more intuitive as it directly mirrors the recursive definition of the problem, the iterative (bottom-up) approach is often slightly more efficient in practice due to avoiding recursion overhead. However, both achieve the same optimal solution.

## Complexity Analysis (for both DP approaches)

* **Time Complexity:** Both the iterative DP and the recursive DP with memoization solve each subproblem `(i, w)` only once. There are `n * W` possible subproblems (where `n` is the number of items and W is the maximum capacity). Each subproblem takes constant time to solve (a few comparisons and additions).
Therefore, the time complexity for both approaches is **O(n * W)**.

* **Space Complexity:**

    * **Iterative Approach:** Requires a 2D array `dp` of size `(n+1) x (W+1)`. So, the space complexity is **O(n * W)**. (It's possible to optimize space to O(W) for the iterative version if we only need the final value, not the path).
    * **Recursive Approach with Memoization:** Requires a 2D array `memo` of size `(n+1) x (W+1)`, plus the space for the recursion call stack. In the worst case, the recursion depth can be `n`. So, the space complexity is **O(n * W) + O(n) = O(n * W)**.

## Key Takeaways

* The 0/1 Knapsack Problem is a classic optimization problem where you aim to maximize the total value of items selected without exceeding a weight capacity, with the constraint that each item must be either fully included or not at all.
* Dynamic Programming is an effective technique to solve this problem, using either an iterative (bottom-up) approach or a recursive approach with memoization (top-down).
* Both DP approaches typically have a time complexity of O(n * W) and a space complexity of O(n * W).
* This problem structure appears in various resource allocation scenarios beyond just packing a physical knapsack.

Understanding the 0/1 Knapsack problem and its DP solution is a great way to get comfortable with dynamic programming concepts, which are invaluable for tackling a wide range of optimization challenges in software development.