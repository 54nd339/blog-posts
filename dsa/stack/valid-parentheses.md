---
title: Valid Parentheses - The Stack Is the Data Structure
description: Checking that brackets are balanced and correctly nested — repeatedly deleting matched pairs is quadratic, while a stack that matches each closer against the top is one linear pass.
date: 2021-07-07
draft: false
slug: /dsa/valid-parentheses
tags:
  - Competitive Programming
  - Stack
  - Strings
---

The most recently opened bracket is the first one that must close — last-in, first-out by definition. That is the whole reason a stack solves this.

## Description

Given a string `s` containing just the characters `()[]{}`, decide whether every bracket is closed by the correct type in the correct order.

**Example**

```
Input:  s = "()[]{}"
Output: true

Input:  s = "([)]"
Output: false

Input:  s = "([])"
Output: true
```

**Constraints**

- $1 \le \text{s.length} \le 10^4$
- `s` consists only of `()[]{}`.

## Prerequisites

- [Stack](/citadel/dsa/stack) semantics: push, pop, peek.
- A closing → opening bracket map.

## Approach 1: Brute Force

### Intuition

A valid string always contains an adjacent matched pair. Delete all such pairs, then repeat; the original is valid iff this reduces to empty.

### Algorithm

1. Repeat until the string stops changing: replace every `"()"`, `"[]"`, `"{}"` with `""`.
2. Return whether the result is empty.

```python
def isValid(s: str) -> bool:
    prev = None
    while prev != s:
        prev = s
        s = s.replace("()", "").replace("[]", "").replace("{}", "")
    return s == ""
```

### Complexity

- **Time:** $O(n^2)$ — each pass is $O(n)$ and may remove only one pair, so up to `n/2` passes.
- **Space:** $O(n)$ for the intermediate strings.

## Approach 2: Stack

### Intuition

Scan once. Push openers. On a closer, the stack top must be its matching opener; pop it or fail. At the end the stack must be empty.

### Algorithm

1. Map each closing bracket to its opening bracket. Create an empty stack.
2. For each character `c`:
   - If `c` is a closing bracket: if the stack is empty or its top does not equal `match[c]`, return `false`; otherwise pop.
   - Else push `c`.
3. Return whether the stack is empty.

```python
def isValid(s: str) -> bool:
    match = {")": "(", "]": "[", "}": "{"}
    stack = []
    for c in s:
        if c in match:
            if not stack or stack.pop() != match[c]:
                return False
        else:
            stack.append(c)
    return not stack
```

### Complexity

- **Time:** $O(n)$ — one pass, $O(1)$ per character.
- **Space:** $O(n)$ — the stack, e.g. on `"((((("`.

## Common Pitfalls

- **A closer with an empty stack.** `"())"` — the third character has nothing to match; the `not stack` check catches it.
- **Leftover openers.** `"([]"` — returning `True` unconditionally is wrong; the answer is `not stack`.
- **Keying the map by the opener.** Key it by the *closer* so the lookup on a closing bracket is direct.

## The keystone

When the rule is "the thing opened most recently is the thing resolved first", a stack models it exactly — matching, nesting, undo. This push-on-open, resolve-on-close pattern scales to [Generate Parentheses](/citadel/dsa/generate-parentheses) and expression evaluation in [Evaluate Reverse Polish Notation](/citadel/dsa/evaluate-reverse-polish-notation).
