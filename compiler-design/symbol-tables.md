---
title: Symbol Tables - How a Compiler Tracks Names
description: The data structure every compiler phase reads and writes - what a symbol-table entry holds, how block structure and scope are handled with a stack of tables or scope indicators, why hash tables are the usual implementation, and how a scope tree mirrors nested blocks.
date: 2022-11-28
draft: false
slug: /compiler-design/symbol-tables
tags:
  - Compilers
  - Static Analysis
---

When a [compiler](/citadel/compiler-design/cd) sees `count` used in an expression, it needs to know: is `count` declared? what type? which `count`, if there are several in nested blocks? where does it live in memory? The **symbol table** is where those answers are stored. Nearly every phase touches it — the scanner creates entries, the semantic analyser fills in types, the code generator reads offsets — so its performance matters.

This post covers what an entry holds, how the table copes with nested scopes, and why it is almost always a hash table.

## What an entry holds

For each identifier, a symbol-table entry typically records:

- **Name** — the identifier string.
- **Type** — `int`, `float`, `char*`, `MyClass`, `array of int`, `function`.
- **Scope** — the region where the name is visible. This is what disambiguates two variables both called `x`.
- **Memory location / offset** — once decided, the address or offset from a base pointer.
- **For functions** — parameter count and types, return type, entry address.
- **For arrays** — number of dimensions, size of each, element type.
- **For classes/structs** — pointers to field and method information.
- **Other attributes** — `const`, `static`, visibility, label information, whatever the language needs.

| Name | Type | Scope | Offset | Notes |
| --- | --- | --- | --- | --- |
| `myVar` | `int` | global | 4 | |
| `calculate` | `func(int,int)->int` | global | — | params `val1:int`, `val2:int` |
| `counter` | `int` | `calculate` | 8 | local |
| `isActive` | `bool` | `class_X` | 12 | member |

## Scope and block structure

Most languages are **block-structured**: a `{ }` block, function body, or `if` branch can declare names visible only inside it. A name in an inner scope **shadows** a same-named name outside. The table has to get this right on every lookup.

Two organisations:

1. **One table per scope.** On entering a scope the compiler creates a fresh table with a pointer to its enclosing scope's table. A lookup searches the current table, then the parent, then the grandparent, up to global. On exiting a scope, its table is discarded.
2. **One global table with scope tags.** Every entry carries a scope level or ID. A lookup searches for the name, preferring entries from the current scope, then the nearest enclosing one.

The first is usually implemented as a **stack of tables**: push a table on scope entry, pop it on exit, and start every lookup from the top. This maps `enter_scope` / `exit_scope` directly onto `push` / `pop`.

```text
[global]                     <- bottom of stack
  [calculate]                <- pushed on entering the function
    [block inside calculate] <- top: lookups start here
```

## Hashing

`insert` fires on every declaration and `lookup` on every use, so both must be fast. **Hash tables** are the standard choice: run the name through a **hash function** to get an array index, and store the entry there.

Two names can hash to the same slot — a **collision** — handled by:

- **Chaining** — each slot holds a linked list of all entries that hashed there. The common approach.
- **Open addressing** — on collision, probe for the next free slot by a fixed sequence.

Average `insert` and `lookup` are $O(1)$ with a good hash function and a reasonable load factor; the worst case degrades to $O(n)$ when many keys collide. In a block-structured language, each scope can have its own hash table (the stack-of-tables design) or one table can hold every scope with entries chained to reflect nesting. The [hashing](/citadel/data-structures/data-structures) and [trie](/citadel/data-structures/trie) trade-offs apply here as anywhere.

## A scope tree

Where a hash table gives fast lookup, a **scope tree** makes the nesting explicit. Each node is a scope; the root is global; a node's children are the scopes nested directly inside it; each node points to its own symbol table (often a hash table) for the names declared right there.

```text
        global (G)
        /        \
  calculate (FA)   class_X (CB)
       |
   block1 (B1)
```

Resolving a non-local name is a walk up this tree: miss in the current node's table, try the parent's, and so on. The tree also tends to mirror the [abstract syntax tree](/citadel/compiler-design/semantic-analysis), where function definitions and block statements already introduce new scopes.

## The operations, and when they fire

- **`insert(name, info)`** — on a declaration. The scanner spots the new name; the semantic analyser supplies type and attributes; the entry goes into the *current* scope's table. Re-declaration in the same scope is caught here.
- **`lookup(name)`** — on a use. Search the current scope, then enclosing scopes outward. Found: return the attributes. Not found anywhere: "undeclared identifier".
- **`enter_scope()` / `exit_scope()`** — on `{` and `}` (or function entry/exit). Push or pop a table; on exit, the scope's names become unreachable from outside.

## The one idea to keep

The symbol table is the compiler's shared memory: it is how a late phase uses a fact an early phase discovered. Its two jobs — fast name lookup and correct scope resolution — pull toward different structures, so real compilers combine them: a hash table (or a stack of them) for speed, organised by a scope hierarchy for correctness.
