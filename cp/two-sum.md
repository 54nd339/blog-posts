---
title: Two Sum
description: Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.
date: 2022-03-16
draft: true
slug: /pensieve/cp/two-sum
tags:
  - Array
  - HashTable
---
## Problem Statement

Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.
You can return the answer in any order. come up with an algorithm that is less than O(n2) time complexity?

## Constraints:

- 2 <= nums.length <= 104
- -109 <= nums[i] <= 109
- -109 <= target <= 109
- Only one valid answer exists.

## Examples

  Example 1:

    Input: nums = [2,7,11,15], target = 9
    Output: [0,1]

  Example 2:

    Input: nums = [3,2,4], target = 6
    Output: [1,2]

  Example 3:

    Input: nums = [3,3], target = 6
    Output: [0,1]

## Solution

```cpp
class Solution {
public:
  vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> m;
    for (int i = 0; i < nums.size(); i++) {
      if (m.find(target - nums[i]) != m.end()) {
        return {m[target - nums[i]], i};
      }
      m[nums[i]] = i;
    }
    return {};
  }
};
```

## Explanation

We use a hash table to store the numbers we have seen so far. For each number, we check if there is another number that, if added, would sum to the target. If there is, we have found a solution and return immediately. If not, the number is added to the hash table and we continue iterating until the end of the array.
