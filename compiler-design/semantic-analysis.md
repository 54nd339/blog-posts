---
title: Beyond Syntax - How Compilers Understand Meaning with Semantic Analysis!
description: Dive into Semantic Analysis! Discover Abstract Syntax Trees, Three-Address Code, attributed grammars, syntax-directed translation, and the crucial role of type checking in making your code work.
date: 2023-05-14
draft: false
slug: /pensieve/compiler-design/semantic-analysis
tags:
  - Compiler Design
  - CS Basics
---

Hey Code Detectives! In our [last adventure into Syntax Analysis](/pensieve/compiler-design/syntax-analysis), we saw how the parser takes tokens and builds a parse tree, ensuring our code follows the grammatical rules of the language. But just because a sentence is grammatically correct doesn't mean it makes sense, right? "Colorless green ideas sleep furiously" is grammatically impeccable, but semantically... not so much!

This is where **Semantic Analysis** steps in. It's the phase where the compiler delves deeper into the code's *meaning*. It checks for semantic errors, gathers type information, and ensures that different parts of your program fit together logically. Think of it as the compiler's logic checker or meaning verifier, ensuring your instructions are sensible before trying to execute them.

Let's unravel the magic of how compilers understand what your code truly intends to do!

---
## Bridging the Gap: Intermediate Representations (IR)
Before diving deep into semantic checks, compilers often convert the parse tree (or a more abstract version of it) into an **Intermediate Representation (IR)**, also known as intermediate code. Why?
* **Decoupling**: It separates the source language-dependent frontend from the target machine-dependent backend.
* **Optimization**: Most optimizations are performed on IRs because they are simpler than source code and more machine-independent than target code.

Here are some popular forms of IR:

### 1. Abstract Syntax Trees (ASTs)
An **AST** is a condensed, hierarchical representation of the source code. It's similar to a parse tree but omits details related to parsing derivations and focuses on the essential structure. Operators become internal nodes, and their operands are their children.

For `a = b * 5 + c;`, the AST might look like this conceptually:
* An 'assign' node (=) is at the root.
    * Its left child is the identifier 'a'.
    * Its right child is a 'plus' node (+).
        * The 'plus' node's left child is a 'multiply' node (\*).
            * The 'multiply' node's left child is 'b'.
            * The 'multiply' node's right child is the number '5'.
        * The 'plus' node's right child is 'c'.

This structure clearly shows `b * 5` being done first, then added to `c`, and the result assigned to `a`.

### 2. Polish Notation (and Reverse Polish Notation - RPN)
These are linear, parenthesis-free notations for expressions.
* **Polish Notation (Prefix)**: Operator precedes operands.
    * `a + b` becomes `+ a b`
    * `a + b * c` becomes `+ a * b c` (assuming * has higher precedence)
* **Reverse Polish Notation (Postfix)**: Operator follows operands.
    * `a + b` becomes `a b +`
    * `a + b * c` becomes `a b c * +`
    RPN is particularly useful for stack-based evaluation.

### 3. Three-Address Code (TAC)
**TAC** is a sequence of instructions, each having at most one operator on the right side and at most three "addresses" (names, constants, or compiler-generated temporaries).
General form: `x = y op z` or `x = op y` or `x = y`.
For `a = b * 5 + c;`:
t1 = 5          // If 5 needs to be loaded or is a different type
t2 = b * t1     // Or t2 = b * 5
t3 = t2 + c
a = t3

TAC is favored because it's simple, close to machine instructions, and makes optimization straightforward.

---
## Adding Meaning: Attributed Grammars & Syntax-Directed Translation (SDT)

How does the compiler actually perform semantic checks and build IRs? Through **Syntax-Directed Translation (SDT)**. SDT uses the grammatical structure (syntax) to guide the semantic analysis and translation process.

### Attributed Grammars
An **attributed grammar** is a Context-Free Grammar (CFG) where each grammar symbol (terminal or non-terminal) is associated with a set of **attributes**. Each production rule in the grammar can have **semantic rules** associated with it to compute the values of these attributes.

Attributes can be:
* **Synthesized Attributes**: The value of a synthesized attribute at a node in the parse tree is computed from the attribute values of its children. Information flows *up* the parse tree.
    * *Example*: The `type` of an expression `E -> E1 + E2` is synthesized from the types of `E1` and `E2`. If both are `int`, then `E.type` is `int`.
* **Inherited Attributes**: The value of an inherited attribute at a node is computed from the attribute values of its parent and/or siblings. Information flows *down* or *sideways* across the parse tree.
    * *Example*: In a variable declaration like `int x, y, z;`, the type `int` is inherited by `x`, `y`, and `z` from the declaration specifier.

### Syntax-Directed Translation (SDT)
**SDT** is the process of attaching semantic rules to grammar productions. As the parser recognizes syntactic structures, these rules are executed.
* **Syntax-Directed Definitions (SDDs)**: These are high-level specifications. They associate semantic rules with productions without specifying the order of evaluation.
    * Example (for type checking an expression):
        `Production: E -> E1 + E2`
        `Semantic Rule: E.type = (E1.type == int && E2.type == int) ? int : error;`
* **Translation Schemes**: These embed semantic actions (code fragments) directly within the grammar productions, explicitly defining the order of execution. Often used for generating IR.
    * Example (generating TAC for `E -> E1 + E2`):
        `E -> E1 + E2 { E.address = newTemp(); emit(E.address + " = " + E1.address + " + " + E2.address); }`
        (This is conceptual; `emit` generates a TAC instruction, `E.address` holds the temporary variable name storing the result of E).

---
## The Heart of Meaning: Type Checking ✔️

One of the most critical tasks of semantic analysis is **type checking**. It ensures that operations are performed on compatible types of data.
* **Why?** To catch errors like trying to multiply a string by an array. This prevents unexpected runtime behavior.

**Static vs. Dynamic Type Checking:**
* **Static Type Checking**: Performed at compile-time (e.g., Java, C++, C#). Catches errors early.
* **Dynamic Type Checking**: Performed at run-time (e.g., Python, JavaScript). More flexible, but errors appear later.

**Type Systems:**
Each language has a **type system** – a set of rules that assign a type property to various constructs. The type system defines basic types, how new types are constructed, rules for type equivalence, compatibility, inference, and **type coercion** (implicit type conversions, e.g., `int` to `float` in `5 + 2.0`).

The type checker uses attributed grammars/SDT to determine the type of each expression and verifies these against the language's type rules.

---
## From Code to IR: Translating Language Constructs ️

Semantic analysis, guided by SDT, is also responsible for translating source language constructs into an intermediate representation like Three-Address Code.

* **Assignments**: `x = y + z;`
    ```tac
    t1 = y + z
    x = t1
    ```

* **Arithmetic Expressions**: `a * (b + c)`
    ```tac
    t1 = b + c
    t2 = a * t1
    // Result is in t2
    ```

* **Boolean Expressions & Control Flow**: For `if (x < y) then S1 else S2`:
    A conceptual TAC sequence would involve:
    1.  Code to evaluate `x < y`.
    2.  A conditional jump: `if_false x < y goto L_false` (jump to code for S2 if condition is false).
    3.  `<TAC for S1>`
    4.  `goto L_next` (skip S2).
    5.  `L_false: <TAC for S2>`
    6.  `L_next: ...`

* **Array Accesses**: `arr[i] = val;` (assuming `arr` elements are 4 bytes)
    ```tac
    t1 = i * 4           // Calculate offset
    t2 = base_address(arr) // Get base address of arr
    t3 = t2 + t1         // Address of arr[i]
    *(t3) = val          // Store val at the computed address
    ```

* **Procedure/Function Calls**: `myFunc(p1, p2);`
    ```tac
    param p1
    param p2
    call myFunc, 2  // Call myFunc with 2 parameters
    ```

---
## The Essence of Understanding!

Semantic analysis is where the compiler truly starts to "understand" your program. It moves beyond just checking grammar to verifying that your code makes logical sense. By using intermediate representations, attributed grammars, syntax-directed translation, and rigorous type checking, this phase ensures that variable types are compatible, operations are valid, and control flows correctly.

This deep understanding is crucial not only for catching subtle bugs but also for paving the way for effective code optimization and, ultimately, for generating correct and efficient machine code.
