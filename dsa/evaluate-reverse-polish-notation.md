---
title: Evaluate Reverse Polish Notation - Operators Act on the Last Two Results
description: Evaluating a postfix expression — a stack holds pending operands, and each operator pops two, combines them, and pushes the result, collapsing the whole expression in one linear pass.
date: 2021-07-11
draft: false
slug: /dsa/evaluate-reverse-polish-notation
tags:
  - Competitive Programming
  - Stack
  - Math
---

Postfix needs no parentheses because operator order encodes precedence: when you meet an operator, its two operands are the two most recently produced values. "Most recent two" is a stack.

## Description

Given tokens of an arithmetic expression in Reverse Polish (postfix) notation, evaluate it and return the integer result. Valid operators are `+`, `-`, `*`, `/`; division truncates toward zero.

**Example**

```
Input:  tokens = ["2", "1", "+", "3", "*"]
Output: 9          # ((2 + 1) * 3)

Input:  tokens = ["4", "13", "5", "/", "+"]
Output: 6          # (4 + (13 / 5))
```

**Constraints**

- $1 \le \text{tokens.length} \le 10^4$
- Each token is an operator or an integer in `[-200, 200]`.
- The expression is always valid; no division by zero; `/` truncates toward zero.

## Prerequisites

- [Stack](/citadel/dsa/stack) as an operand store.
- Integer division that truncates toward zero (`int(a / b)`, not `a // b`).

## Approach 1: Brute Force

### Intuition

Repeatedly find the leftmost operator, fold it and its two preceding numbers into one value, and restart.

### Algorithm

1. While more than one token remains, scan left to right for the first operator.
2. Replace it and the two numbers before it with the computed value.
3. Return the last token as an integer.

```python
def evalRPN(tokens: list[str]) -> int:
    tokens = tokens[:]
    ops = {"+", "-", "*", "/"}
    while len(tokens) > 1:
        for i, tok in enumerate(tokens):
            if tok in ops:
                a, b = int(tokens[i - 2]), int(tokens[i - 1])
                val = (a + b if tok == "+" else a - b if tok == "-"
                       else a * b if tok == "*" else int(a / b))
                tokens[i - 2:i + 1] = [str(val)]
                break
    return int(tokens[0])
```

### Complexity

- **Time:** $O(n^2)$ — each fold restarts the scan and shifts the list.
- **Space:** $O(n)$.

## Approach 2: Doubly Linked List

### Intuition

The quadratic cost is list-shifting. A doubly linked list lets you splice three nodes into one in $O(1)$.

### Algorithm

1. Build a doubly linked list of the tokens.
2. Walk it; at each operator node, take its two previous nodes as operands, compute, and replace all three with a single value node, fixing the links.
3. When one node remains, return its value.

```python
def evalRPN(tokens: list[str]) -> int:
    class Node:
        __slots__ = ("val", "prev", "next")
        def __init__(self, val):
            self.val, self.prev, self.next = val, None, None

    head = cur = Node(tokens[0])
    for t in tokens[1:]:
        node = Node(t)
        cur.next, node.prev = node, cur
        cur = node

    ops = {"+", "-", "*", "/"}
    node = head
    while node:
        if node.val in ops:
            b = int(node.prev.val)
            a = int(node.prev.prev.val)
            r = (a + b if node.val == "+" else a - b if node.val == "-"
                 else a * b if node.val == "*" else int(a / b))
            new = Node(str(r))
            before = node.prev.prev.prev
            after = node.next
            new.prev, new.next = before, after
            if before:
                before.next = new
            else:
                head = new
            if after:
                after.prev = new
            node = new
        node = node.next
    return int(head.val)
```

### Complexity

- **Time:** $O(n)$ — each node processed once, splices are $O(1)$.
- **Space:** $O(n)$ for the list.

## Approach 3: Recursion

### Intuition

The last token is the outermost operator. Consume tokens from the end: a number returns itself; an operator recursively evaluates its right operand, then its left.

### Algorithm

1. Set a pointer to the last index.
2. `helper()`: read and consume the token at the pointer. If a number, return it. If an operator, `right = helper()`, `left = helper()`, apply and return.

```python
def evalRPN(tokens: list[str]) -> int:
    pos = len(tokens) - 1

    def helper() -> int:
        nonlocal pos
        tok = tokens[pos]
        pos -= 1
        if tok not in ("+", "-", "*", "/"):
            return int(tok)
        right = helper()
        left = helper()
        if tok == "+":
            return left + right
        if tok == "-":
            return left - right
        if tok == "*":
            return left * right
        return int(left / right)

    return helper()
```

### Complexity

- **Time:** $O(n)$ — each token visited once.
- **Space:** $O(n)$ recursion depth.

## Approach 4: Stack

### Intuition

Scan left to right. Push numbers. On an operator, pop two (second pop = left operand, first pop = right), apply, push the result. The final stack holds one value.

### Algorithm

1. For each token: if it is a number, push `int(token)`.
2. If it is an operator, `b = pop()`, `a = pop()`, push the operator applied to `(a, b)`.
3. Return the single remaining value.

```python
def evalRPN(tokens: list[str]) -> int:
    stack: list[int] = []
    ops = {
        "+": lambda a, b: a + b,
        "-": lambda a, b: a - b,
        "*": lambda a, b: a * b,
        "/": lambda a, b: int(a / b),
    }
    for tok in tokens:
        if tok in ops:
            b = stack.pop()
            a = stack.pop()
            stack.append(ops[tok](a, b))
        else:
            stack.append(int(tok))
    return stack[0]
```

### Complexity

- **Time:** $O(n)$ — one pass.
- **Space:** $O(n)$ for the stack.

## Common Pitfalls

- **`//` instead of `int(a / b)`.** `//` floors: `-7 // 2 == -4`, but the problem wants `-3`. Truncation toward zero needs `int(a / b)`.
- **Operand order.** The second value popped is the *left* operand. `a = pop(); b = pop()` makes `a` the right operand and `a - b` computes right minus left.
- **Recursion reads right-first.** Going right-to-left, the first recursive call yields the right operand, the second the left.

## The keystone

A stack turns any postfix expression into a fold: push operands, let each operator reduce the top of the stack. Infix-to-postfix (shunting yard) plus this evaluator is how calculators and many runtimes handle arithmetic.
