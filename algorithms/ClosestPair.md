---
title: Finding Needles in Haystacks - Solving the Closest Pair of Points with Divide and Conquer
description: Given a scatter of points, how do you efficiently find the two closest to each other? We explore the Closest Pair of Points problem and an elegant O(n log n) solution using the Divide and Conquer algorithm.
date: 2023-02-04
draft: false
slug: /pensieve/algorithms/closest-pair-of-points
tags:
  - DSA
  - Divide and Conquer
---

Hey everyone, and welcome back to the blog! Imagine you have a map dotted with various locations – say, all the popular street food stalls in KIIT – and you want to find out which two stalls are closest to each other. If you only have a few stalls, you could just measure the distance between every pair. But what if you have thousands, or even millions? That brute-force approach quickly becomes unmanageable!

This is the essence of the **Closest Pair of Points problem**, a fundamental challenge in computational geometry. It asks: given a set of points in a plane, find the two points that have the smallest Euclidean distance between them. Today, we'll explore how a clever algorithmic technique, **Divide and Conquer**, can solve this problem much more efficiently than brute force.

## The Problem: Finding the Closest Duo
Formally, given a set of $n$ points $P = \{p_1, p_2, \ldots, p_n\}$ in a 2D plane, where each point $p_i = (x_i, y_i)$, we want to find two distinct points $p_a$ and $p_b$ in $P$ such that the Euclidean distance between them is minimized.

The Euclidean distance $d$ between two points $p_1 = (x_1, y_1)$ and $p_2 = (x_2, y_2)$ is given by the familiar formula:

$$
d(p_1, p_2) = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}
$$

A naive approach would be to calculate the distance between every possible pair of points, which involves $\binom{n}{2} = \frac{n(n-1)}{2}$ pairs, leading to an O(n²) time complexity. For large `n`, this is too slow.

## The Divide and Conquer Strategy
The Divide and Conquer paradigm typically involves three steps:

1. **Divide:** Break the problem into smaller, independent subproblems.
2. **Conquer:** Solve the subproblems recursively. If the subproblems are small enough, solve them directly.
3. **Combine:** Combine the solutions of the subproblems to get the solution for the original problem.

Here’s how we apply this to the Closest Pair of Points problem:

1. **Sort for Efficiency:** First, pre-sort all the points in the set $P$ according to their x-coordinates. Let this sorted array be $P_x$. Also, create another array $P_y$ which is the same set of points sorted by their y-coordinates. This initial sorting takes O(n log n) time.
2. **Divide:** If the number of points $|P_x|$ is small (e.g., 3 or less), solve the problem by brute force (calculate all pairwise distances). Otherwise, find a vertical line $L$ (defined by $x = x_{mid}$) that divides the set of points $P_x$ into two roughly equal-sized subsets:
   * $Q_x$: Points to the left of or on line $L$.
   * $R_x$: Points to the right of line $L$.
     (Corresponding $Q_y$ and $R_y$ arrays sorted by y-coordinate can also be derived or passed down).
3. **Conquer:** Recursively call the closest pair algorithm on the left subset $Q_x$ (using $Q_y$) to find the closest pair $(p_1, q_1)$ with distance $d_L$. Similarly, recursively find the closest pair $(p_2, q_2)$ in the right subset $R_x$ (using $R_y$) with distance $d_R$.
4. **Combine - The Tricky Part:**
   * Let $d = \min(d_L, d_R)$. This $d$ is the minimum distance found *so far* where both points are in the same half.
   * Now, we must consider pairs where one point is in the left subset ($Q_x$) and the other is in the right subset ($R_x$). Such a "split pair" could potentially be closer than $d$.
   * We only need to check for split pairs whose distance could be less than $d$. This means we only need to consider points within a vertical "strip" of width $2d$ centered around the dividing line $L$ (i.e., points whose x-coordinates are within $x_{mid} - d$ and $x_{mid} + d$). Let's call this set of points $S_{strip}$.
   * Sort the points in $S_{strip}$ by their y-coordinates. (This is where having $P_y$ pre-sorted helps; $S_{strip}$ sorted by y can be constructed efficiently from $P_y$).
   * For each point $p$ in the sorted $S_{strip}$, we only need to check distances to other points in $S_{strip}$ that are "close" to $p$ in the y-dimension. A crucial geometric insight here is that for any point $p$ in the strip, we only need to check a small, constant number of subsequent points (at most 7, in fact, that fall within a $d \times 2d$ rectangle around $p$) in the y-sorted strip. This is because if two points are closer than $d$, their y-coordinates cannot differ by more than $d$.
   * Let $(p_3, q_3)$ be the closest pair found in this strip step, with distance $d_{strip}$.
   * The overall closest pair is the minimum of $(p_1, q_1)$, $(p_2, q_2)$, and $(p_3, q_3)$.

The time complexity of this algorithm is **O(n log n)**, a significant improvement over the O(n²) brute-force method. The recurrence relation is T(n) = 2T(n/2) + O(n), which solves to O(n log n).

## Pseudocode Insights

Let's look at the conceptual pseudocode you provided, which captures the essence of this algorithm.

```pseudocode
// P: A list of point objects, pre-sorted by x-coordinate
// Assumes points have .x and .y attributes
// distance(p1, p2) calculates Euclidean distance

function closestPair(P_x): // P_x is points sorted by x
    n = length(P_x)
    if n <= 3: // Base case: for small number of points, use brute force
        return bruteForceClosestPair(P_x)
    else:
        // Divide
        mid_index = n / 2
        Q_x = P_x[0 : mid_index]      // Left half
        R_x = P_x[mid_index : n]  // Right half

        // Conquer: Recursively find closest pairs in left and right halves
        (p1_L, q1_L) = closestPair(Q_x) // Closest pair in left
        dist_L = distance(p1_L, q1_L)

        (p1_R, q1_R) = closestPair(R_x) // Closest pair in right
        dist_R = distance(p1_R, q1_R)

        // Determine current minimum distance d from the two halves
        d = min(dist_L, dist_R)
     
        // Tentatively set the overall best pair based on d
        if dist_L < dist_R:
            min_pair_so_far = (p1_L, q1_L)
        else:
            min_pair_so_far = (p1_R, q1_R)

        // Combine: Find closest "split pair" (one point in left, one in right)
        // This involves points in a strip around the dividing line
        (p3_split, q3_split, dist_split) = closestSplitPair(P_x, P_y_sorted, d) // P_y_sorted is P sorted by y

        // Compare d with the distance of the closest split pair
        if dist_split < d:
            return (p3_split, q3_split) // The split pair is the overall closest
        else:
            return min_pair_so_far // The closest pair was entirely in one half

// P_y_sorted: All points from original P, sorted by y-coordinate
// d: current minimum distance found in either left or right halves
function closestSplitPair(P_x_original, P_y_original_sorted, d):
    mid_x_coordinate = P_x_original[length(P_x_original)/2].x // x-coordinate of the dividing line

    // Create a strip of points within distance 'd' of the dividing line
    S_strip_y_sorted = []
    for p in P_y_original_sorted:
        if abs(p.x - mid_x_coordinate) < d:
            S_strip_y_sorted.add(p)

    min_split_dist = d
    closest_strip_pair = null // Initialize

    // For each point in the y-sorted strip, check a few subsequent points
    for i = 0 to length(S_strip_y_sorted) - 2:
        // Only need to check points within d distance in y-dimension.
        // Due to geometric properties, we only need to check a small constant
        // number of points ahead (e.g., at most 7 other points in the d x 2d box).
        for j = i + 1 to min(i + 7, length(S_strip_y_sorted) - 1):
            p_i = S_strip_y_sorted[i]
            p_j = S_strip_y_sorted[j]
         
            // Optimization: if y-distance itself is already >= d, no need to check further for p_i
            if (p_j.y - p_i.y) >= d:
                break
         
            current_dist = distance(p_i, p_j)
            if current_dist < min_split_dist:
                min_split_dist = current_dist
                closest_strip_pair = (p_i, p_j)
 
    if closest_strip_pair is not null:
        return (closest_strip_pair[0], closest_strip_pair[1], min_split_dist)
    else:
        // Return a pair indicating no split pair was closer than d
        // This needs careful handling; perhaps return (null, null, d_infinity)
        // For simplicity here, assuming d is returned if no closer pair found
        return (null, null, d)

// Helper for base case
function bruteForceClosestPair(P_small_set):
    min_dist = infinity
    pair = (null, null)
    for i = 0 to length(P_small_set) - 1:
        for j = i + 1 to length(P_small_set) - 1:
            d = distance(P_small_set[i], P_small_set[j])
            if d < min_dist:
                min_dist = d
                pair = (P_small_set[i], P_small_set[j])
    return pair // Returns the pair of points
```

The `closestPair` function is the main recursive driver. If the number of points is small (e.g., ≤ 3), it directly computes the closest pair using a brute-force check. Otherwise, it divides the set, recursively solves for the two halves, gets the minimum distance d found so far, and then calls `closestSplitPair`. The `closestSplitPair` function is the clever part: it considers only points within a $2d$-width strip around the dividing line, sorts them by y-coordinate, and then for each point, only checks distances to a small, constant number of its y-neighbors in the strip. This crucial optimization ensures the "combine" step takes only O(n) time.

## Why is the Combine Step O(n)? The Magic of the Strip

The critical insight for the O(n log n) complexity lies in the closestSplitPair step.

1. Creating the strip $S_{strip}$ by filtering points from $P_y$ (which is $P$ sorted by y-coordinate) takes O(n) time.
2. The inner loops in `closestSplitPair` look like they might be O(n²), but they are not. For any point $p$ in the sorted strip, we only need to check distances to other points in the strip that fall within a $d \times 2d$ rectangle below it (or above it, depending on iteration direction). It can be proven that there can be at most a small constant number of such points (around 7-8) for any $p$ if the minimum distance is $d$. Thus, the inner loop effectively runs a constant number of times for each point in the strip.
3. Since there are at most $n$ points in the strip, this part of the combine step also takes O(n) time.

## Correctness of the Algorithm

The correctness relies on the divide and conquer approach:

1. **Base Case:** If the number of points is very small (e.g., 1, 2, or 3), the brute-force method correctly finds the closest pair.
2. **Recursive Step & Combine Step:**
   * If the closest pair of points lies entirely in one of the subsets (left or right), the recursive calls will correctly find this pair.
   * If the closest pair of points involves one point from the left subset and one from the right subset (a "split pair"), then both points must lie within the $2d$-width strip around the dividing line (where $d$ is the minimum distance found within the left and right subsets). The `closestSplitPair` function is specifically designed to find such a pair if its distance is less than $d$. The proof that checking only a constant number of neighbors in the y-sorted strip is sufficient is a key geometric argument.

By correctly handling these cases, the algorithm ensures it finds the overall closest pair.

## Key Takeaways

* The **Closest Pair of Points problem** seeks to find two points in a set with the minimum Euclidean distance between them.
* A naive brute-force approach is O(n²), which is too slow for large datasets.
* A **Divide and Conquer algorithm** provides an efficient O(n log n) solution.
* The key steps involve:
  1. Pre-sorting points by x and y coordinates.
  2. Recursively finding the closest pairs in left and right halves.
  3. A clever "combine" step that efficiently checks for closer pairs in a "strip" around the dividing line, only considering a constant number of y-neighbors for each point in the strip.

This algorithm is a beautiful example of how the divide and conquer strategy, combined with geometric insights, can lead to significant performance improvements for computational geometry problems.
