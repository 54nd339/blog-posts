---
title: Functional Programming Languages
description: A family of languages that treats a program as an expression to evaluate, not a sequence of state changes. Pure functions, immutable data, and first-class functions are the core; type inference, laziness, algebraic data types, and the actor model are what different members add. The ideas have leaked into every mainstream language.
date: 2023-02-06
draft: false
slug: /coding-languages/functional-languages
tags:
  - Programming Languages
  - Type Systems
---

Most languages you've used are **imperative**: a program is a list of commands that mutate state — assign this variable, push to that list, increment a counter. Functional languages start from a different place. A program is an **expression**, and running it means *evaluating* that expression. There are no statements, ideally no mutation, and functions are values you pass around like integers.

That sounds academic, and functional languages were academic for decades. But the ideas — immutability, pure functions, `map`/`filter`/`reduce`, pattern matching, `Option` instead of `null` — are now in Java, C#, Python, JavaScript, Rust, Swift, and Kotlin. Learning where they came from makes them make sense.

## The core ideas

**Pure functions.** A function is pure if its output depends only on its inputs and it has no side effects — no I/O, no mutating a global, no writing to a field. `f(x)` returns the same thing every time, and calling it changes nothing.

The payoff of purity is **referential transparency**: an expression can be replaced by its value without changing the program's meaning. That makes code easy to reason about (read one function, understand it fully), easy to test (no setup, no mocks — just inputs and outputs), easy to parallelise (pure functions can't race), and amenable to aggressive compiler optimisation (memoise, reorder, eliminate).

**Immutability.** Data structures aren't modified; operations return new versions. `list.append(x)` gives you a *new* list; the old one is unchanged and any other reference to it is safe. This sounds wasteful but **persistent data structures** share most of their internal structure between versions (a tree where an "update" copies only the path from root to the changed node — O(log n), not O(n)), so it's cheap in practice. Immutability is why functional code has so few aliasing bugs.

**First-class and higher-order functions.** Functions are values: store them, pass them, return them. A **higher-order function** takes or returns a function — `map`, `filter`, `reduce`/`fold`, `compose`. Loops become these combinators; a `for` loop that transforms a list becomes `map`, one that accumulates becomes `fold`.

**Recursion over iteration.** No mutable loop counter, so iteration is recursion. **Tail-call optimisation** — when the recursive call is the last thing a function does, the compiler reuses the stack frame — makes it as efficient as a loop and stack-safe.

## Type systems

Statically typed functional languages (Haskell, OCaml, F#, the ML family) get a lot of mileage from types:

- **Hindley–Milner inference** — the compiler figures out every type without annotations, and still catches type errors. You write `let add x y = x + y` and it knows `add : int -> int -> int`.
- **Algebraic data types** — types built by combining others: **product** types (a record: "a name *and* an age") and **sum** types (a tagged union: "a `Circle` *or* a `Rectangle` *or* a `Triangle`"). `Option`/`Maybe` (`Some x` or `None`) is a sum type, and it's why these languages don't have `null` — "might be absent" is in the type, and the compiler forces you to handle the `None` case.
- **Pattern matching** — destructure a value by its shape, with **exhaustiveness checking**: add a new variant to a sum type and the compiler flags every `match` that doesn't handle it. This turns "I forgot a case" from a runtime bug into a compile error.
- **Parametric polymorphism** (generics) and **type classes / traits** — `Ord a => a -> a -> Bool` says "for any type that has an ordering." Rust's traits and Swift's protocols are this idea.

## Laziness

**Haskell** evaluates expressions only when their value is actually needed. This lets you define infinite structures (`primes = ...` an infinite list) and consume a finite prefix, and it can skip computing values a branch never uses. The cost is that reasoning about *when* things evaluate (and about space usage — unevaluated thunks can pile up) is harder, so Haskell also has strictness annotations for when you want eager evaluation. Most functional languages (OCaml, F#, Scala, Clojure) are eager by default with opt-in laziness.

## Effects: doing I/O in a pure world

If functions are pure, how do you print or read a file? Different answers:

- **Haskell** — I/O actions are *values* of type `IO a`; you compose them and hand the final `IO` action to the runtime to execute. The `IO` type marks anything impure, so a pure function's signature guarantees it does no I/O. **Monads** are the general pattern for sequencing such effectful computations (and for `Maybe`, for lists, for parsers…) — `do` notation is sugar for it.
- **OCaml / F# / Scala / Clojure** — pragmatic: effects are allowed, purity is a strong convention, and the standard library is mostly pure.
- **Algebraic effects** (a newer approach, in OCaml 5, Koka) — declare the effects a function performs and handle them separately, like typed, resumable exceptions.

## The languages

- **Haskell** — pure, lazy, powerful type system. The research vehicle; used in finance, compilers, and anywhere correctness pays.
- **OCaml** — eager, pragmatic, fast compiler. Powers Jane Street's trading systems, the Rust compiler's early versions, and the `Flow`/`Infer` tools at Meta.
- **F#** — OCaml-family on .NET; functional-first but full interop with C# libraries.
- **Scala** — functional + object-oriented on the JVM; big-data (Spark is written in it) and backend services.
- **Clojure** — a Lisp on the JVM: dynamically typed, immutable-by-default data structures, macros, a strong story for concurrency (software transactional memory, `atom`s).
- **Erlang / Elixir** — the **actor model**: lightweight isolated processes communicating only by messages, "let it crash" supervision trees, hot code reloading, and the BEAM VM built for massive concurrency and uptime. WhatsApp, Discord's real-time backend, and telecom switches run on it.

## The leak into the mainstream

- **Lambdas and streams** — Java 8's `stream().map().filter().collect()`, C#'s LINQ, Python comprehensions.
- **Immutability** — `record` types in Java/C#, `const`/`readonly`, persistent collections (Immutable.js, Kotlin's `List`).
- **`Option`/`Result` over `null` and exceptions** — Rust's `Option<T>` and `Result<T, E>`, Swift's optionals, Kotlin's nullable types with compiler-enforced handling.
- **Pattern matching** — added to Java, Python (`match`), C#, with exhaustiveness where the language can.
- **Pure-ish core, effects at the edges** — the "functional core, imperative shell" architecture pattern.

## The one idea to keep

Functional programming treats computation as evaluating expressions built from pure functions over immutable data, which buys referential transparency — you can replace any expression with its value, making code easy to test, parallelise, and reason about locally. Static functional languages layer on Hindley–Milner inference, algebraic data types, and exhaustive pattern matching so "forgot a case" and "might be null" become compile errors. Even if you never write Haskell, these ideas are why modern languages have `map`, `Option`, `record`, and `match`.
