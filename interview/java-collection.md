---
title: Java Collections Framework - Interfaces and Implementations
description: The JCF separates what a collection does (the interface) from how it does it (the implementation). The Collection hierarchy of List, Set, and Queue, why Map sits outside it, and the cost table that decides which class to pick.
date: 2024-11-22
draft: false
slug: /interview/java-collection
tags:
  - System Design
  - Interview Prep
  - Java
---

The Java Collections Framework is built on one separation: an **interface** says *what* a collection can do — add, remove, iterate — and an **implementation** is a concrete class that decides *how*, with its own trade-offs in ordering, memory, and speed. You code against the interface and pick the implementation for the access pattern. This walks the hierarchy and the cost table that drives the choice.

## The Collection interface

`Collection` defines the basic operations and extends `Iterable`, which is what lets any collection be walked with a for-each loop or an explicit `Iterator`. Three subinterfaces specialise it.

**`List`** — an ordered sequence. Keeps insertion order, allows duplicates, and supports access by integer index.

**`Set`** — models a mathematical set: no duplicates. Order is not guaranteed in general, though specific implementations impose one.

**`Queue`** — holds elements awaiting processing, with insert/extract/inspect operations. Usually FIFO; `PriorityQueue` instead orders by natural ordering or a supplied `Comparator`.

## Why Map is separate

`Map` is part of the framework but is **not** a `Collection`. A `Collection` is a group of single elements; a `Map` is a set of key-value **pairs**, each key unique and mapping to one value. You can view its keys, values, or entries as collections, but the map itself is not a collection of single items the way a `List` or `Set` is.

## Choosing an implementation

The interface fixes the semantics; the implementation fixes the performance and the ordering.

| Class | Interface | Ordering | Typical costs |
| --- | --- | --- | --- |
| `ArrayList` | List | insertion | index/get O(1); add at end amortised O(1); insert/remove mid O(n) |
| `LinkedList` | List, Queue | insertion | get by index O(n); add/remove at ends O(1) |
| `HashSet` | Set | none | add / remove / contains O(1) average |
| `LinkedHashSet` | Set | insertion | O(1) average, with a linked list tracking order |
| `TreeSet` | Set (SortedSet) | sorted | add / remove / contains O(log n) |
| `HashMap` | Map | none | get / put O(1) average |
| `LinkedHashMap` | Map | insertion (or access) | O(1) average, ordered |
| `TreeMap` | Map (SortedMap) | sorted keys | get / put O(log n) |
| `PriorityQueue` | Queue | heap (by comparator) | peek O(1); offer / poll O(log n) |

The single highest-value habit: use a hash-based `Set` or `Map` for membership and lookup instead of scanning a `List`. Checking `contains` on a 10,000-element `ArrayList` is 10,000 comparisons; on a `HashSet` it is one hash and a bucket probe.

## Pick the interface first, then the class

Declare variables and parameters as `List`, `Set`, `Map` — the interface — so the implementation can change without touching callers. Then choose the class from the table: `ArrayList` and `HashMap` are the right default the large majority of the time; reach for `LinkedHashMap` when iteration order matters, `TreeMap` when you need the keys sorted or range queries, `ArrayDeque` or `PriorityQueue` when the access pattern is a queue. The wrong pick still compiles and still works — it just quietly turns an O(1) operation into an O(n) one.
