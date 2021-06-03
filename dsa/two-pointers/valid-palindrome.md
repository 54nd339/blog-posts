---
title: Valid Palindrome - Two Pointers Without a Cleaned Copy
description: Checking whether a string reads the same both ways after dropping non-alphanumeric characters — build a filtered copy, or walk two pointers in from the ends and skip the junk in place.
date: 2021-06-03
draft: false
slug: /dsa/valid-palindrome
tags:
  - Competitive Programming
  - Two Pointers
  - Strings
---

The easy version cleans the string and compares it to its reverse. The version worth internalising walks one pointer in from each end, stepping over punctuation and spaces, and never allocates a second string.

## Description

Given a string `s`, return `true` if it is a palindrome once you consider only alphanumeric characters and ignore case.

**Example**

```
Input:  s = "A man, a plan, a canal: Panama"
Output: true          # "amanaplanacanalpanama"

Input:  s = "race a car"
Output: false
```

**Constraints**

- $1 \le \text{s.length} \le 2 \times 10^5$
- `s` consists of printable ASCII characters.

## Prerequisites

- [Two pointers](/citadel/dsa/two-pointers) converging from the ends.
- `str.isalnum()` / `str.lower()` and the fact that `isalnum` is Unicode-aware.

## Approach 1: Reverse String

### Intuition

Strip the string down to lowercase alphanumerics, then a palindrome is exactly a string equal to its own reverse.

### Algorithm

1. Build a new string (or list) of the lowercased alphanumeric characters of `s`.
2. Compare it with its reverse.
3. Return whether they are equal.

```python
def isPalindrome(s: str) -> bool:
    cleaned = [c.lower() for c in s if c.isalnum()]
    return cleaned == cleaned[::-1]
```

### Complexity

- **Time:** $O(n)$ — one filtering pass plus a reversal.
- **Space:** $O(n)$ — the cleaned sequence and its reversed copy.

## Approach 2: Two Pointers

### Intuition

Compare the string from both ends inward. Non-alphanumeric characters are simply skipped where they sit, so no filtered copy is needed.

### Algorithm

1. Set `lo = 0`, `hi = len(s) - 1`.
2. While `lo < hi`:
   - Advance `lo` while `s[lo]` is not alphanumeric.
   - Retreat `hi` while `s[hi]` is not alphanumeric.
   - If `s[lo].lower() != s[hi].lower()`, return `false`.
   - Move both pointers inward.
3. Return `true`.

```python
def isPalindrome(s: str) -> bool:
    lo, hi = 0, len(s) - 1
    while lo < hi:
        while lo < hi and not s[lo].isalnum():
            lo += 1
        while lo < hi and not s[hi].isalnum():
            hi -= 1
        if s[lo].lower() != s[hi].lower():
            return False
        lo += 1
        hi -= 1
    return True
```

### Complexity

- **Time:** $O(n)$ — each pointer only moves inward, so together they cross the string once.
- **Space:** $O(1)$ — no allocation.

## Common Pitfalls

- **Dropping the `lo < hi` guard inside the inner loops.** On a string of all punctuation (`",,,"`), `lo` runs off the end searching for an alphanumeric and `s[lo]` throws `IndexError`.
- **Forgetting `.lower()`** — `"Aa"` is a valid palindrome only after case folding.
- **Assuming Approach 1 is ASCII-only.** `str.isalnum()` keeps accented letters and other scripts; usually fine, but a surprise if the spec means "ASCII letters and digits".

## The keystone

Converging pointers turn "compare position `i` with position `n-1-i`" into one inward sweep, and skipping unwanted characters *in place* removes the filtered copy — $O(n)$ time, $O(1)$ space. The same shape solves [Two Sum II](/citadel/dsa/two-sum-ii), [3Sum](/citadel/dsa/3sum), and [Container With Most Water](/citadel/dsa/container-with-most-water).
