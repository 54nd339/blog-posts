---
title: The Activity Selection Problem - A Greedy Approach to Maximizing Your Schedule
description: Ever tried to fit as many non-overlapping activities into your day as possible? We explore the Activity Selection Problem, a classic combinatorial optimization challenge, and how a simple greedy algorithm can find an optimal solution.
date: 2023-02-11
draft: false
slug: /pensieve/algorithms/activity-selection
tags:
  - DSA
  - Greedy
---

Hey everyone, and welcome back to the blog! We all juggle multiple tasks and activities, trying to make the most of our time. Imagine you have a list of potential activities, each with a specific start and finish time, and you want to pick the maximum number of activities you can do without any of them overlapping. This is the essence of the **Activity Selection Problem**, a classic puzzle in the world of algorithms and combinatorial optimization.

It might sound complex, but thankfully, there's an elegant and efficient **greedy algorithm** that can help us find an optimal solution. Today, let's break down this problem, understand the greedy strategy, and even look at a proof of why this simple approach works so well!

## The Problem: Picking the Most Activities ️

Let's formally state the problem:

You are given $n$ activities, and each activity $i$ is represented by a **start time $s_i$** and a **finish time $f_i$**. Two activities, $i$ and $j$, are considered **non-conflicting** (or compatible) if their time intervals don't overlap. This means either activity $i$ starts after or at the same time activity $j$ finishes ($s_i \geq f_j$), OR activity $j$ starts after or at the same time activity $i$ finishes ($s_j \geq f_i$).

The goal of the activity selection problem is to find the largest possible set of mutually non-conflicting activities. We're looking for a solution set $S$ such that no other valid solution set $S'$ exists where $|S'| > |S|$.

## The Greedy Strategy: Pick Early, Finish Early
The core idea behind the greedy approach to solving the activity selection problem is surprisingly simple:

1.  **Sort the activities by their finish times** in increasing order.
2.  Select the **first activity** from this sorted list (the one that finishes earliest) and add it to your solution set.
3.  Then, iterate through the remaining sorted activities. For each activity, if its **start time is greater than or equal to the finish time of the last activity you selected**, then this new activity is compatible. Add it to your solution set and update your "last selected activity."
4.  Repeat until all activities have been considered.

This strategy makes a locally optimal choice at each step (picking the compatible activity that finishes earliest) with the hope of finding a global optimum.

### Pseudocode for the Greedy Algorithm

Here's how this looks in pseudocode, directly reflecting the logic you provided:

```pseudocode
Greedy-Iterative-Activity-Selector(A, s, f):
    // A: Array of activities (e.g., activity objects or indices)
    // s: Array of start times corresponding to activities in A
    // f: Array of finish times corresponding to activities in A

    // 1. Sort activities in A by their finish times (stored in f)
    Sort A by finish times stored in f

    // 2. Initialize the set of selected activities S with the first activity
    S = {A[1]} // Assuming 1-based indexing for A, s, f

    // 3. k keeps track of the index of the most recently added activity to S
    k = 1
   
    n = A.length // Total number of activities
   
    // 4. Iterate through the remaining activities
    for i = 2 to n:
        // If the start time of the current activity A[i] is greater than or
        // equal to the finish time of the last selected activity A[k]...
        if s[i] >= f[k]:
            // ...then activity A[i] is compatible. Add it to our solution set S.
            S = S U {A[i]}
            // Update k to be the index of this newly added activity.
            k = i
   
    // 5. Return the set of selected activities
    return S
```

### Explanation of the Steps:
1. `A` is an array representing the activities. `s` and `f` are arrays containing the start and finish times respectively. (The pseudocode uses 1-based indexing, common in some algorithm textbooks).
2. **Crucial First Step:** Sort the activities based on their finish times in ascending order. This sorting typically takes O(n log n) time.
3. Initialize a set `S` to store our selected non-conflicting activities. We greedily pick the very first activity in the sorted list (the one with the earliest finish time overall) and add it to S.
4. Initialize a variable `k` to keep track of the index of the last activity added to `S`. Initially, this is the first activity.
5. Iterate through the rest of the sorted activities (from the second activity up to the last).
6. For the current activity `A[i]`, check if its start time `s[i]` is greater than or equal to the finish time `f[k]` of the last activity `A[k]` that was added to our solution set `S`.
    * If this condition `s[i] ≥ f[k]` is true, it means activity `A[i]` does not conflict with the previously selected activity `A[k]` (and by extension, all activities already in S, because we always pick the earliest finishing compatible activity). So, `A[i]` can be added to `S`.
7. If `A[i]` is added to `S`, update k to be i, as `A[i]` is now the last activity selected.

## Why Does This Greedy Approach Work? Proof of Correctness

It might seem too simple, but this greedy strategy indeed yields an optimal solution. Your provided text outlines a clear proof using the "greedy choice stays ahead" argument and optimal substructure:

1. **Greedy Choice Property:**
    Let $S_{\text{activities}} = {1,2,\ldots,n}$ be the set of activities ordered by finish time. Assume that $A \subseteq S_{\text{activities}}$ is an optimal solution, also ordered by finish time; and that the index of the first activity in $A$ is $k \neq 1$, i.e., this optimal solution does not start with the greedy choice (activity 1).

    We can show that $B = (A \setminus {k}) \cup {1}$, which begins with the greedy choice (activity 1), is another optimal solution. Since $f_1 \leq f_k$ (activity 1 finishes no later than activity $k$), and the activities in $A$ are disjoint by definition, the activities in $B$ are also disjoint. Since $B$ has the same number of activities as $A$, that is, $|A| = |B|$, $B$ is also optimal.

    This demonstrates that there is always an optimal solution that includes the first activity selected by the greedy algorithm (the one with the earliest finish time).

2. **Optimal Substructure:**
    Once the greedy choice ($a_1$) is made, the problem reduces to finding an optimal solution for the subproblem consisting of activities that start after $a_1$ finishes.

    If $A$ is an optimal solution to the original problem $S_{activities}$ containing the greedy choice $a_1$, then $A' = A \setminus {a_1}$ is an optimal solution to the activity-selection subproblem $S' = {i \in S_{activities} : s_i \geq f_1}$.

    As you pointed out, if this were not the case, and we could pick a solution $B'$ to $S'$ with more activities than $A'$, then adding $a_1$ to $B'$ would yield a feasible solution $B$ to $S_{activities}$ with more activities than $A$, which contradicts the optimality of $A$.

    By repeatedly making the greedy choice and then solving the remaining subproblem, we construct an overall optimal solution.

## Complexity Analysis

* **Time Complexity:** The dominant part of the algorithm is sorting the activities by their finish times, which takes O(n log n) time, where $n$ is the number of activities. The subsequent for loop iterates through the activities once, which takes O(n) time. Thus, the total time complexity is O(n log n) + O(n) = O(n log n).
* **Space Complexity:** The algorithm uses a set S to store the selected activities, which can contain at most $n$ activities in the worst case. If we consider the space for storing the input arrays (or if they are sorted in-place and the output is a list of indices), the auxiliary space for S is O(n). Thus, the space complexity is typically considered O(n).

## Applications of Activity Selection

The activity selection problem, while a classic algorithmic puzzle, has conceptual parallels in various real-world scheduling scenarios:

* **Meeting Scheduling:** Optimizing the number of meetings that can be held in a single conference room given their start and end times.
* **Resource Allocation:** Assigning a limited resource (like a machine or a person) to the maximum number of compatible tasks.
* **CPU Task Scheduling:** Selecting a set of non-conflicting tasks for execution on a single processor, where each task has a defined start and finish requirement.
* **Interval Scheduling:** A more general class of problems where you want to select a maximum number of compatible intervals.

While real-world scenarios often introduce more complex constraints (like activity priorities, different resource requirements, etc.), the core greedy idea of prioritizing activities that finish earliest (thus freeing up the resource sooner for subsequent compatible activities) is a powerful and often effective heuristic.

## Key Takeaways

* **The Activity Selection Problem** aims to find the maximum number of non-conflicting activities from a set, given their start and finish times.
* A simple yet elegant **greedy algorithm** solves this problem optimally. The strategy involves sorting activities by their finish times and then iteratively selecting the next compatible activity that has the earliest finish time.
* The correctness of this greedy approach is supported by the "greedy choice property" and the "optimal substructure" property of the problem.
* The time complexity is dominated by the initial sort, resulting in O(n log n), with O(n) space complexity.

It's a beautiful example of how a seemingly straightforward greedy choice can lead to a globally optimal solution for certain types of optimization problems!