---
title: Segment Trees - Efficiently Conquering Range Queries and Updates
description: Unlock the power of Segment Trees! We dive into this versatile tree data structure, perfect for handling range queries (like sum, min, max) and updates on an array in logarithmic time. Explore its properties, implementations, and applications.
date: 2023-01-31
draft: false
slug: /pensieve/tree/segment-tree
tags:
  - DSA
  - Trees
---

Hey everyone, and welcome back to the blog! When you're dealing with arrays and need to frequently ask questions about a specific range of elements—like "What's the sum of elements from index 3 to 7?" or "What's the minimum value in this part of the array?"—doing it naively by iterating every time can be slow, especially if the array is large or you have many queries. This is where specialized data structures like the **Segment Tree** come to the rescue!

Segment Trees are powerful tree-based data structures used to store information about intervals or segments. They are particularly adept at handling range query problems and can also support efficient updates to array elements. Whether you're into competitive programming or building systems that need to quickly analyze array segments, understanding Segment Trees is a valuable skill.

## What's a Segment Tree All About? Introduction & Properties

A **Segment Tree** is essentially a **full binary tree** where each node represents an interval (or segment) of an array.

* The **root** of the segment tree represents the entire array (e.g., interval `[0, n-1]`).
* Each **internal node** represents the union of its children's intervals. For instance, if a node represents `[L, R]`, its left child might represent `[L, mid]` and its right child `[mid+1, R]`. The value stored in the internal node is typically an aggregate (like sum, min, max) of the values in the interval it represents.
* The **leaf nodes** of the segment tree represent the individual elements of the original array.

This structure allows for efficient querying and updating. Here are some key properties:

* **Full Binary Tree Structure:** Each internal node has exactly two children (though for implementation convenience, especially array-based, this might be implicit).
* **Logarithmic Height:** The height of a segment tree built on an array of `n` elements is O(log n). This is key to its efficiency.
* **Node Count:** A segment tree will have roughly `2 * n - 1` nodes in its pointer-based implementation, or might use an array of size up to `4n` in some array-based implementations to simplify indexing.
* **Purpose:** Primarily designed to store information about array intervals and perform range queries and point updates in logarithmic time.

---
## Building a Segment Tree: Implementation Approaches

Segment trees can be implemented in a couple of ways:

### 1. Array-based Implementation

This is often more efficient in practice due to better cache locality and no overhead for pointers.

* **Representation:** The tree is stored in an array. If the root is at index `i` (often `0` or `1` depending on convention):
  * Its left child is at index `2*i + 1` (if 0-indexed) or `2*i` (if 1-indexed).
  * Its right child is at index `2*i + 2` (if 0-indexed) or `2*i + 1` (if 1-indexed).
  * The parent of a node at index `j` is at index `(j-1)/2` (if 0-indexed) or `j/2` (if 1-indexed).
* **Size:** For an array of `n` elements, the segment tree array typically needs a size of around `2 * n` (for a "compact" bottom-up build) or up to `4 * n` for a more general recursive build to accommodate all nodes without complex index calculations.

Here's a conceptual pseudocode for an array-based implementation focusing on range sums (using a common competitive programming style for bottom-up construction):

```pseudocode
class SegmentTree_ArrayBased:
    tree // The array to store the segment tree nodes
    n    // The size of the original input array

    // Constructor to build the tree
    // nums: the input array
    constructor buildTree(nums):
        n = length(nums)
        tree = new int[2 * n] // For compact bottom-up, size 2n is enough

        // Initialize leaf nodes with input array values
        for i = 0 to n-1:
            tree[n + i] = nums[i]
 
        // Build internal nodes by summing children, from bottom up
        // Starts from the parent of the last leaf node
        for i = n-1 downto 1: // Iterate from (n-1) down to 1 (root)
            tree[i] = tree[2 * i] + tree[2 * i + 1] // Sum of left and right child

    // Update the value at a specific original array index
    // index: original array index (0-based)
    // value: new value to set
    function updateTree(original_index, value):
        pos = original_index + n // Position of the leaf node in the tree array
        tree[pos] = value
 
        // Propagate updates upwards to parent nodes
        while pos > 1:
            // Parent is at pos/2. The other child is pos^1 (XOR 1 flips last bit, finds sibling)
            // For parent tree[pos/2], its children are tree[pos] and tree[pos^1]
            tree[pos / 2] = tree[pos] + tree[pos ^ 1]
            pos = pos / 2 // Move to parent

    // Query the sum of elements in a given range [left, right) (0-based, exclusive right)
    // left: start index of range in original array
    // right: end index of range in original array (exclusive)
    function queryTree(left, right): // Queries range [left, right-1]
        // Adjust to 1-based indexing in the tree array and map to leaf positions
        left = left + n
        right = right + n // right is exclusive, so it points one past the actual segment
 
        sum_val = 0
        while left < right:
            // If left is a right child (odd index), include its value and move to parent's right
            if left % 2 == 1: // or (left & 1)
                sum_val = sum_val + tree[left]
                left = left + 1
     
            // If right is a right child (odd index), it means its left sibling is part of the range
            // So include the node *before* right (which is right-1), and move to parent's left
            if right % 2 == 1: // or (right & 1)
                right = right - 1
                sum_val = sum_val + tree[right]
     
            // Move to parent level
            left = left / 2
            right = right / 2
        return sum_val
```

(Note: The array-based pseudocode above is a common iterative style used in competitive programming for sum queries. Different operations like min/max or lazy propagation would alter the logic, especially for updates and queries.)

### 2. Tree-based (Pointer-based) Implementation

This is often more intuitive to understand as it explicitly uses nodes and pointers.

* **Representation:** Each node in the segment tree is an object or struct containing:
  * The range `[start, end]` it represents.
  * The aggregated value for that range (e.g., sum, min, max).
  * Pointers to its `left` and `right` children.

```pseudocode
class SegmentTreeNode:
    start // Start index of the interval
    end   // End index of the interval
    sum   // Aggregated sum for this interval (can be min, max, etc.)
    left  // Pointer to the left child node
    right // Pointer to the right child node

class SegmentTree_PointerBased:
    root  // The root node of the segment tree

    // Public method to build the tree
    constructor buildTree(nums):
        if nums is null or length(nums) == 0:
            root = null
            return
        root = buildTreeHelper(nums, 0, length(nums)-1)

    // Recursive helper to build the tree
    function buildTreeHelper(nums, start_idx, end_idx):
        if start_idx > end_idx: // Base case for invalid range
            return null

        node = new SegmentTreeNode()
        node.start = start_idx
        node.end = end_idx

        if start_idx == end_idx: // Leaf node
            node.sum = nums[start_idx]
        else: // Internal node
            mid_idx = start_idx + (end_idx - start_idx) / 2
            node.left = buildTreeHelper(nums, start_idx, mid_idx)
            node.right = buildTreeHelper(nums, mid_idx + 1, end_idx)
            // Aggregate value from children
            node.sum = (node.left != null ? node.left.sum : 0) +
                       (node.right != null ? node.right.sum : 0)
        return node

    // Public method to update a value
    function updateTree(original_array_index, value):
        updateTreeHelper(root, original_array_index, value)

    // Recursive helper to update the tree
    function updateTreeHelper(node, idx, val):
        if node is null:
            return

        if node.start == node.end: // Leaf node found
            if node.start == idx: // Ensure it's the correct leaf
                node.sum = val
            return

        mid_idx = node.start + (node.end - node.start) / 2
        if idx <= mid_idx: // Update is in the left subtree
            updateTreeHelper(node.left, idx, val)
        else: // Update is in the right subtree
            updateTreeHelper(node.right, idx, val)
 
        // Recalculate sum after child update
        node.sum = (node.left != null ? node.left.sum : 0) +
                   (node.right != null ? node.right.sum : 0)

    // Public method for range query
    function queryTree(query_left_idx, query_right_idx):
        return queryTreeHelper(root, query_left_idx, query_right_idx)

    // Recursive helper for range query
    function queryTreeHelper(node, query_left, query_right):
        if node is null or query_left > node.end or query_right < node.start:
            return 0 // Query range is outside this node's range, or node is null

        // If this node's range is completely within the query range
        if query_left <= node.start and query_right >= node.end:
            return node.sum

        // If this node's range partially overlaps or contains the query range
        mid_idx = node.start + (node.end - node.start) / 2
        left_sum = queryTreeHelper(node.left, query_left, query_right)
        right_sum = queryTreeHelper(node.right, query_left, query_right)
 
        return left_sum + right_sum
```

## Core Operations on a Segment Tree

Regardless of implementation, Segment Trees efficiently support these main operations:

### 1. Build

* **Goal:** Construct the segment tree from a given input array.
* **Process:** This typically involves recursively dividing the array into segments and calculating the aggregate value for each node. For array-based bottom-up, it involves initializing leaves and then computing parent values.
* **Time Complexity:** O(n), where `n` is the number of elements in the array, because each element is processed a constant number of times during the tree construction.
* **Space Complexity:** O(n) to store the tree itself (which has about `2n` to `4n` nodes).

### 2. Update (Point Update)

* **Goal:** Modify the value of an element at a specific index in the original array and reflect this change in the segment tree.
* **Process:** Update the corresponding leaf node. Then, propagate this change upwards by recalculating the aggregate values of all affected parent nodes up to the root.
* **Time Complexity:** O(log n), because only the nodes on the path from the updated leaf to the root need to be changed, and the height of the tree is O(log n).
* **Space Complexity:** O(1) for iterative array-based, O(log n) for recursive pointer-based due to call stack.

### 3. Query (Range Query)

* **Goal:** Retrieve an aggregate value (e.g., sum, minimum, maximum) over a specified range `[L, R]` of the original array.
* **Process:** This involves traversing the segment tree. If a node's segment is completely within the query range, its pre-computed aggregate value is used. If a node's segment partially overlaps, the query is recursively made on its children. If a node's segment is completely outside the query range, it's ignored.
* **Time Complexity:** O(log n), because a range query in a segment tree typically decomposes the query range into at most O(log n) disjoint segments represented by nodes in the tree.
* **Space Complexity:** O(1) for iterative array-based, O(log n) for recursive pointer-based.

## Applications of Segment Trees

Segment trees are incredibly versatile and are used in a wide range of applications where range queries and point updates are frequent:

* **Range Sum Queries (RSQ):** Finding the sum of elements in a given range of an array (as shown in our examples).
* **Range Minimum/Maximum Queries (RMQ):** Finding the minimum or maximum element in a given range.
* **Range Frequency Queries:** Finding the number of elements in a given range that satisfy a certain condition.
* **Maximum Subarray Sum Problem:** Segment trees can be adapted to solve variations of this classic problem.
* **Computational Geometry:** Problems involving geometric intervals or segments.
* **Dynamic Programming Optimizations:** Certain DP problems can be sped up using segment trees.
* **Data Analytics and Time-Series Analysis:** Quickly aggregating data over time windows or other ranges.

## Key Takeaways

* Segment Trees are powerful tree data structures designed for efficient range queries and point updates on an array.
* They achieve O(log n) time complexity for query and update operations, and O(n) for building the tree.
* The core idea is to pre-compute and store aggregate information for intervals, allowing range queries to be answered quickly by combining information from a small number of tree nodes.
* They can be implemented using arrays (often more performant) or with explicit tree nodes and pointers (often more intuitive).
* Segment trees are a staple in competitive programming and are useful in various real-world applications requiring efficient range-based data processing.

While a bit more complex to implement than simple arrays, the performance gains offered by Segment Trees for range-based problems are often well worth the effort!
