---
title: Semantic Analysis - Type Checking and Intermediate Code
description: The phase that checks what grammar cannot - intermediate representations (abstract syntax trees, three-address code), attributed grammars with synthesized and inherited attributes, syntax-directed translation, static versus dynamic type checking, and how source constructs become three-address code.
date: 2022-11-26
draft: false
slug: /compiler-design/semantic-analysis
tags:
  - Compilers
  - Static Analysis
  - Code Generation
---

"Colorless green ideas sleep furiously" is grammatically perfect and means nothing. A [parser](/citadel/compiler-design/syntax-analysis) accepts `int x = "hello" * 3;` for the same reason — the token structure is fine; the meaning is not. **Semantic analysis** is the phase that checks meaning: types line up, names are declared before use, operations are legal. It also starts turning the tree into something closer to machine code.

This post covers the intermediate forms the compiler builds, the mechanism it uses to compute facts over the tree, and the central check — type checking.

## Intermediate representations

Before deep semantic work, the compiler converts the parse tree into an **intermediate representation** (IR): lower-level than source, higher-level than any one machine's assembly. The IR is the seam between the [front end and back end](/citadel/compiler-design/cd) — it decouples the source language from the target, and most optimization runs on it because it is simpler than source and machine-neutral.

### Abstract syntax trees

An **AST** is a parse tree with the parsing scaffolding removed. Operators become interior nodes, operands their children. For `a = b * 5 + c`:

```text
      =
     / \
    a   +
       / \
      *   c
     / \
    b   5
```

It says directly what the parse tree said indirectly: multiply `b` by `5`, add `c`, assign to `a`.

### Polish and reverse Polish notation

Linear, parenthesis-free forms. **Prefix** (Polish) puts the operator first: `a + b * c` becomes `+ a * b c`. **Postfix** (reverse Polish) puts it last: `a b c * +`. Postfix is convenient for stack-based evaluation — push operands, and each operator pops its arguments.

### Three-address code

**Three-address code** (TAC) is a sequence of instructions, each with one operator and at most three operands (`x = y op z`, `x = op y`, or `x = y`), using compiler-generated temporaries. For `a = b * 5 + c`:

```text
t1 = b * 5
t2 = t1 + c
a  = t2
```

TAC is favoured for optimization: it is simple, close to machine instructions, and every intermediate value has a name to attach analysis to.

## Attributed grammars and syntax-directed translation

The compiler computes semantic facts by attaching them to the grammar. In an **attributed grammar**, each grammar symbol carries **attributes** (a type, an address, a piece of generated code), and each production carries **semantic rules** that compute them.

- **Synthesized attributes** flow *up* the tree — a node's value comes from its children. The type of `E -> E1 + E2` is synthesized from the types of `E1` and `E2`: both `int` gives `E.type = int`.
- **Inherited attributes** flow *down* or *sideways* — from parent or siblings. In `int x, y, z;`, the type `int` is inherited by each of `x`, `y`, `z` from the declaration.

**Syntax-directed translation** (SDT) is the process of running these rules as the parser recognises structure. Two styles:

- **Syntax-directed definitions** state the rules without fixing an evaluation order:
  ```text
  E -> E1 + E2   { E.type = (E1.type == int && E2.type == int) ? int : error }
  ```
- **Translation schemes** embed action code at specific points in the production, fixing the order — used for emitting IR:
  ```text
  E -> E1 + E2   { E.addr = newTemp(); emit(E.addr, "=", E1.addr, "+", E2.addr); }
  ```

## Type checking

The central semantic check: operations get operands of compatible types. It catches "multiply a string by an array" before it becomes undefined runtime behaviour.

**Static** type checking runs at compile time (C, Java, Rust) and catches type errors early. **Dynamic** type checking runs at runtime (Python, JavaScript) and is more flexible but surfaces errors later, on the path that triggers them.

Each language has a **type system**: rules assigning a type to every construct, defining how compound types are built, when two types are equivalent or compatible, and where **coercion** — an implicit conversion like `int` to `float` in `5 + 2.0` — is inserted. The type checker walks the tree with synthesized and inherited attributes, computes each expression's type, checks it against the rules, and adds coercion nodes where the language permits.

## Translating constructs to TAC

Guided by translation schemes, semantic analysis lowers each source construct:

**Assignment** `x = y + z`:
```text
t1 = y + z
x  = t1
```

**Nested arithmetic** `a * (b + c)`:
```text
t1 = b + c
t2 = a * t1
```

**Control flow** `if (x < y) S1 else S2`:
```text
        if x < y goto L1
        <code for S2>
        goto L2
  L1:   <code for S1>
  L2:
```

**Array access** `arr[i] = val` (4-byte elements):
```text
t1 = i * 4
t2 = &arr
t3 = t2 + t1
*t3 = val
```

**Function call** `f(p1, p2)`:
```text
param p1
param p2
call f, 2
```

## The one idea to keep

Semantic analysis is two jobs sharing one walk of the tree: verify the program makes sense, and lower it toward machine code. Both are driven by the same mechanism — attributes computed by rules attached to grammar productions — so the parser's structure directly organises the meaning-checking and the translation. The three-address code it emits is what the [optimizer](/citadel/compiler-design/code-optimisation) and [code generator](/citadel/compiler-design/code-genration) take from here.
