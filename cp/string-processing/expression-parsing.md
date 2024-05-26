---
title: Expression Parsing - Shunting Yard and Operator Precedence
description: Turning an infix arithmetic string into a value with two stacks — one for numbers, one for operators — applying an operator when a lower-or-equal precedence one arrives, plus unary minus and right-associativity.
date: 2024-05-26
draft: false
slug: /cp/expression-parsing
tags:
  - Competitive Programming
  - String Processing
  - Parsing
---

Given a string like `"3 + 4 * 2 - ( 1 - 5 ) ^ 2"`, compute its value respecting precedence and parentheses. Dijkstra's shunting-yard algorithm does it in one left-to-right pass with two stacks: numbers, and pending operators. It is the standard contest answer for calculator problems and the first step of any small interpreter.

## The problem

Evaluate (or convert to postfix / a tree) an infix expression with:

- binary operators of various precedence and associativity (`+ - * / % ^`),
- parentheses,
- unary minus (and maybe unary plus),
- multi-digit numbers, possibly negatives via the unary case.

Example: `3 + 4 * 2` = $11$; `( 3 + 4 ) * 2` = $14$; `2 ^ 3 ^ 2` = $512$ (right-associative).

## The idea

Keep `nums` (a value stack) and `ops` (an operator stack). Scan tokens:

- **Number** → push to `nums`.
- **`(`** → push to `ops`.
- **`)`** → while the top of `ops` is not `(`, pop and apply; then pop the `(`.
- **Operator `o`** → while `ops` is non-empty, its top is not `(`, and the top has **higher precedence**, or **equal precedence and `o` is left-associative** → pop and apply. Then push `o`.
- **End of input** → pop and apply until `ops` is empty.

"Apply" pops the right operand, then the left, computes, and pushes the result. Unary minus is detected by context (a `-` at the start, after another operator, or after `(`) and handled either as a distinct high-precedence operator or by pushing a `0` first.

## How it works

`3 + 4 * 2 - 1`:

| token | action | `nums` | `ops` |
| --- | --- | --- | --- |
| 3 | push | [3] | [] |
| + | push | [3] | [+] |
| 4 | push | [3,4] | [+] |
| * | `*` > `+`? push | [3,4] | [+,*] |
| 2 | push | [3,4,2] | [+,*] |
| - | `*`≥`-` apply; `+`≥`-` apply; push | [11] | [-] |
| 1 | push | [11,1] | [-] |
| end | apply | [10] | [] |

Result $10$.

## Algorithm

```python
import re

PREC = {"+": 1, "-": 1, "*": 2, "/": 2, "%": 2, "^": 3, "u-": 4}
RIGHT = {"^", "u-"}

def apply(nums: list[int], op: str) -> None:
    if op == "u-":
        nums.append(-nums.pop())
        return
    b = nums.pop()
    a = nums.pop()
    if op == "+":   nums.append(a + b)
    elif op == "-": nums.append(a - b)
    elif op == "*": nums.append(a * b)
    elif op == "/": nums.append(a // b)     # or int(a / b) to truncate toward zero
    elif op == "%": nums.append(a % b)
    elif op == "^": nums.append(a ** b)

def evaluate(expr: str) -> int:
    tokens = re.findall(r"\d+|[()+\-*/%^]", expr)
    nums, ops = [], []
    prev = None                                  # previous token type: None / "num" / "op" / "("
    for tok in tokens:
        if tok.isdigit():
            nums.append(int(tok))
            prev = "num"
        elif tok == "(":
            ops.append(tok)
            prev = "("
        elif tok == ")":
            while ops[-1] != "(":
                apply(nums, ops.pop())
            ops.pop()
            prev = "num"
        else:
            if tok == "-" and prev in (None, "op", "("):
                tok = "u-"                        # unary minus
            while (ops and ops[-1] != "("
                   and (PREC[ops[-1]] > PREC[tok]
                        or (PREC[ops[-1]] == PREC[tok] and tok not in RIGHT))):
                apply(nums, ops.pop())
            ops.append(tok)
            prev = "op"
    while ops:
        apply(nums, ops.pop())
    return nums[0]
```

## Variants

- **To postfix (RPN):** instead of `apply`, append the popped operator to an output list. Evaluating RPN is then a single value-stack pass — see [evaluate reverse Polish notation](/citadel/dsa/evaluate-reverse-polish-notation).
- **To an expression tree:** `apply` pops two subtree roots, makes a new node, pushes it.
- **Recursive descent** is the alternative: one function per precedence level (`parse_expr` calls `parse_term` calls `parse_factor`), each looping over its operators. Cleaner for grammars with many levels or function calls; shunting-yard is terser for pure arithmetic.
- **Functions and commas** (`max(a, b)`): push function tokens like operators with special "pop until `(`" handling and an argument counter.

## Complexity

- **Time:** $O(n)$ — each token is pushed and popped at most once.
- **Space:** $O(n)$ for the two stacks.

## Common pitfalls

- **Unary vs binary minus.** Decide by the previous token: a `-` after nothing, an operator, or `(` is unary. Give `u-` high precedence and right-associativity.
- **Associativity in the pop condition.** Left-assoc pops on `>=` precedence; right-assoc (`^`) pops only on strictly `>`. Mixing them evaluates `2^3^2` as $64$ instead of $512$.
- **Integer division sign.** `a // b` in Python floors toward $-\infty$; if the problem wants truncation toward zero, use `int(a / b)` or `math.trunc`.
- **Tokenising.** A regex like `\d+|[()+\-*/%^]` skips whitespace for free; handle multi-character operators (`**`, `<=`) explicitly if the grammar has them.
- **Mismatched parentheses.** `ops[-1]` on an empty stack, or leftover `(` at the end — validate if the input is not guaranteed well-formed.

## The keystone

Shunting-yard evaluates infix in one pass with a value stack and an operator stack: push numbers, and before pushing an operator, drain every stacked operator that binds at least as tightly (strictly tighter, for right-associative ones). Parentheses are just push-and-drain markers. The same skeleton emits RPN or builds a parse tree.
