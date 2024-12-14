---
title: Programming Paradigms - Ten Ways to Structure Code
description: A paradigm is a philosophy of how computation is expressed, independent of language. Ten influential ones — imperative, declarative, OOP, functional, logic, reactive, concurrent, event-driven, generic, distributed — with what each optimizes for.
date: 2024-12-14
draft: false
slug: /interview/programming-paradigms
tags:
  - System Design
  - Interview Prep
  - Programming Paradigms
---

A **programming paradigm** is a way of thinking about computation — a set of concepts that shape how a program is structured and how state and control flow are expressed. It is not a language; most modern languages are multi-paradigm and let you mix them. Here are ten influential paradigms, each with its core idea, what it is good and bad at, and where it shows up.

## 1. Imperative

Code that **controls execution flow and state changes directly** — explicit statements telling the machine *how* to do a task, step by step, mutating variables as it goes. **Procedural programming** is imperative code organised into functions that call each other.

```java
int sum = 0;
for (int i = 0; i < numbers.length; i++) {
    sum += numbers[i];   // mutate state, step by step
}
```

Maps closely to hardware and is efficient; managing mutable state by hand gets tangled at scale. *C, Fortran, Pascal, assembly.*

## 2. Declarative

Describe **what result you want, not how to compute it** — the system works out the how. Concise and close to a specification for its domain; less flexible for general work, and performance rides on the engine's optimiser. *SQL, HTML, CSS, regex, Makefiles.* Functional and logic programming are its main sub-paradigms.

## 3. Object-oriented

Structure the program around **objects** — instances of classes bundling data with the methods that act on it — and their interactions. Four pillars: encapsulation, abstraction, inheritance, polymorphism, covered in [OOP and SOLID](/citadel/interview/oop). **Class-based** OOP (Java, C++) inherits by defining classes; **prototype-based** OOP (early JavaScript, Self) inherits by cloning objects. Good for modelling entities and reuse; prone to deep hierarchies and god objects if undisciplined. *Java, C++, Python, C#, Ruby.*

## 4. Functional

A sub-paradigm of declarative: computation as the **evaluation of mathematical functions**, emphasising **pure functions** (output depends only on input, no side effects), **immutability**, and functions as first-class values.

```java
int sumOfSquares = numbers.stream()
    .map(n -> n * n)
    .reduce(0, Integer::sum);
```

Easy to reason about and test, and safe for concurrency because there is no shared mutable state; steeper learning curve, and side effects (I/O) need deliberate handling. *Haskell, Clojure, Scala, F#; Streams in Java, comprehensions in Python.*

## 5. Logic

Also declarative. A program is a set of **facts and rules**; you pose a query and an inference engine deduces answers by unification and backtracking search.

```prolog
parent(john, pete).
parent(susan, john).
grandparent(GP, GC) :- parent(GP, P), parent(P, GC).
% ?- grandparent(susan, pete).  ->  true
```

Strong for knowledge representation, symbolic reasoning, and expert systems; poor for numeric or I/O-heavy work. *Prolog, Datalog.*

## 6. Reactive

Declarative programming built around **asynchronous data streams and the propagation of change**. Logic is expressed as operators that transform and combine observable streams, with subscribers reacting to new values and **backpressure** handling fast producers. Excellent for responsive UIs and event-heavy, I/O-bound services; debugging long operator chains is hard. *RxJava, Project Reactor, RxJS, Akka Streams.*

## 7. Concurrent

Language constructs for **multiple computations running (or appearing to run) at once** — threads, message passing, shared-resource management. The **actor model** is a sub-paradigm: independent actors, each with private state and a mailbox, communicating only by asynchronous messages. Uses multi-core hardware and keeps applications responsive; notoriously hard to get right, with race conditions, deadlocks, and starvation lurking. *Go (goroutines, channels), Erlang/Elixir (actors), `java.util.concurrent`.* See [process, thread, concurrency](/citadel/interview/process-thread).

## 8. Event-driven

Program flow is **determined by events** — clicks, key presses, messages, timers — dispatched to listeners by an event loop, typically asynchronous and callback-based. Highly responsive; well suited to GUIs and I/O-bound servers ([Node.js](/citadel/interview/http)). Unmanaged, it degenerates into callback nesting; the non-linear flow is harder to trace. Closely related to [event sourcing](/citadel/interview/event-sourcing) and [choreography](/citadel/interview/orchestration).

## 9. Generic

Write algorithms and data structures against **type parameters to be filled in later**, instantiated per concrete type without losing type safety.

```java
static <T> void printAll(List<T> list) {
    for (T item : list) System.out.print(item + " ");
}
```

High reuse and compile-time type checking; syntax and error messages can get arcane (C++ templates), and some implementations have limits (Java's type erasure). *C++ templates, generics in Java, C#, Rust, Swift.*

## 10. Distributed

Software spanning **multiple autonomous machines that coordinate by passing messages over a network**. It must confront concurrency, partial failure, network latency, and consistency across nodes. Buys scalability and fault tolerance; costs a large jump in complexity. Realised with [message queues](/citadel/interview/message-queue), [gRPC](/citadel/interview/grpc), Kubernetes, and a [microservices](/citadel/interview/cloud-native) architecture.

## Paradigms are lenses, and you switch them mid-project

The paradigms are not rival religions to pick one of. A typical service uses OOP for its overall structure, functional style for data transformations, event-driven patterns at the edges, generics for reusable containers, and has to reason about concurrency throughout. Knowing several means that when a problem is awkward in the paradigm you are in — deeply nested state changes, a rule engine, a stream of events — you recognise it and reach for the lens that fits.
