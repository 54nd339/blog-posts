---
title: Java - The JVM, Bytecode, and the Ecosystem
description: Java compiles to portable bytecode that runs on any Java Virtual Machine - that is the whole "write once, run anywhere" idea. How the JVM loads, verifies, and JIT-compiles bytecode; the core language features; what changed from Java 8 to 21; and the ecosystem of Spring, Maven, Gradle, and application servers.
date: 2020-11-10
draft: false
slug: /coding-languages/java
tags:
  - Programming Languages
  - Java
  - JVM
---

Java was released in 1995 by a team led by James Gosling at Sun Microsystems (now Oracle), with one headline promise: **"write once, run anywhere" (WORA)**. Compile your code once, and it runs unchanged on any operating system and processor that has a Java Virtual Machine. Thirty years on it still runs a large share of enterprise backends, Android apps, and big-data infrastructure. This post covers how that portability works and what surrounds the language.

## Core characteristics

- **Object-oriented.** Built around encapsulation, abstraction, inheritance, and polymorphism. See [OOP](/citadel/interview/oop).
- **Statically typed.** Variable types are declared and checked at compile time, catching a class of errors early.
- **Platform-independent via the JVM.** Source compiles to bytecode, not native code; the JVM runs the bytecode.
- **Automatic memory management.** A [garbage collector](/citadel/interview/garbage-collection) reclaims unused objects, so there is no manual free and no dangling pointers.
- **JIT-compiled for speed.** Initially interpreted, then the hot paths are compiled to native code at runtime — enough to be competitive with compiled languages for long-running work.

## How Java runs: bytecode and the JVM

Two stages, which is what makes WORA possible ([the general pattern](/citadel/coding-languages/coding-langs)):

### 1. Compile to bytecode

`javac` translates `.java` source into **Java bytecode** — a compact, platform-independent instruction set — stored in `.class` files. Bytecode targets the JVM, not any real processor.

### 2. Execute on the JVM

The **Java Virtual Machine** is a software engine, built for each OS and CPU, that provides the runtime for bytecode. Running a class involves:

1. **Classloader** — loads `.class` files into memory, from disk or the network.
2. **Bytecode verifier** — checks the loaded bytecode against the JVM spec before it runs: no illegal memory access, no stack overflows, no type violations.
3. **Execution engine** — either **interprets** bytecode instruction by instruction, or, more usually, runs a **JIT compiler** that detects frequently executed "hot spots" and compiles them to native machine code at runtime, so later calls run at native speed.

Other JVM components:

- **Garbage collector** — reclaims memory from objects no longer reachable.
- **JNI (Java Native Interface)** — lets Java call, and be called by, code in C, C++, or assembly.
- **JRE vs JDK** — the JRE is the JVM plus core libraries, enough to *run* Java. The JDK is the JRE plus development tools (`javac`, debugger) needed to *build* it.

## Core language features

- **A large standard library**, organised into packages: `java.lang` (`Object`, `String`, `Thread`, `Math`), `java.util` (the [collections framework](/citadel/interview/java-collection), date/time, random), `java.io` (files and streams), `java.net` (sockets, URLs), `java.math` (`BigInteger`, `BigDecimal`), `java.time` (the modern date/time API, Java 8+).
- **Exception handling** — `try`/`catch`/`finally`/`throw`/`throws` for structured runtime-error handling.
- **Multithreading and concurrency** — the `Thread` class and `Runnable` interface at the low level; the `java.util.concurrent` package (Java 5+) for the Executor framework, thread-safe collections, locks, semaphores, and atomics.
- **A managed, sandboxed runtime** — the bytecode verifier and the runtime security model give Java its security reputation, though vulnerabilities still occur and need patching.

## What changed since Java 8

Java releases on a six-month cadence now; the notable additions:

- **Lambda expressions and functional interfaces** (8) — concise functional constructs.
- **Streams API** (8) — functional-style processing of collections.
- **`Optional`** (8) — a container that may or may not hold a value, to handle nulls without `NullPointerException`.
- **Modules** (Project Jigsaw, 9) — a way to modularise large applications and the JDK itself.
- **`var`** (10) — local-variable type inference, less boilerplate.
- **Records** (16) — concise immutable data-carrier classes.
- **Pattern matching for `instanceof`** (16+) — check and cast in one step.
- **Sealed classes** (17) — restrict which types may extend or implement a class or interface.
- **Virtual threads** (Project Loom, GA in 21) — lightweight user-mode threads that let a program run far more concurrent operations without the cost of OS threads.

## The ecosystem

- **Frameworks.** **Spring** and **Spring Boot** dominate enterprise Java: dependency injection, aspect-oriented programming, transaction management, web MVC, data access, and security, with Spring Boot removing most of the configuration. **Jakarta EE** (formerly Java EE) is a set of specifications — Servlets, JSP, EJB, JPA, JMS — for multi-tier server applications. Also **Quarkus** (Kubernetes-native), **Vert.x** (reactive), and **Play**.
- **Build tools.** **Maven** manages the build, dependencies, and packaging from a `pom.xml`. **Gradle** does the same with a Groovy or Kotlin DSL and more flexibility.
- **IDEs.** IntelliJ IDEA, Eclipse, Apache NetBeans.
- **Application servers.** Apache Tomcat and Jetty (servlet containers), WildFly, WebSphere, WebLogic (full Jakarta EE).

## Where Java is used

- **Enterprise backends** — banking, finance, insurance, retail, where robustness, scale, and security matter.
- **Web applications** — Spring Boot, Spring MVC, Jakarta EE.
- **Android** — the original primary language, still widely used, though Kotlin is now often preferred.
- **Big data** — Hadoop, Spark, Kafka, and Elasticsearch are written in Java or Scala (also JVM). See [the Hadoop ecosystem](/citadel/big-data/hadoop).
- **Embedded and IoT** — Java ME and specialised JVMs for constrained devices.

## Key takeaways

- Java compiles to portable **bytecode**; the **JVM** loads it, verifies it, and JIT-compiles the hot paths to native code — that pipeline is "write once, run anywhere".
- **Garbage collection**, a large standard library, structured exceptions, and built-in concurrency make it a solid choice for large systems.
- The language keeps evolving: lambdas and streams (8), modules (9), records and sealed classes (16–17), virtual threads (21).
- The ecosystem — **Spring/Spring Boot**, **Maven/Gradle**, mature IDEs, a huge existing codebase — is as much of the reason Java persists as the language itself.
