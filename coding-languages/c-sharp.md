---
title: C# and .NET - The CLR, Bytecode, and the Platform
description: C# compiles to CIL bytecode that the Common Language Runtime JIT-compiles and runs - the same two-stage idea as Java's JVM. The language's core features, how .NET went from Windows-only Framework to cross-platform .NET 8+, what the CLR does, and the frameworks for web, data, and cross-platform UI.
date: 2020-11-13
draft: false
slug: /coding-languages/c-sharp
tags:
  - Programming Languages
  - C#
  - .NET
---

C# (C-sharp) and the .NET platform came out of Microsoft in 2000, designed by a team led by Anders Hejlsberg — who had previously built Turbo Pascal and Delphi. The aim was the power of C++ with the productivity of Visual Basic, in a modern object-oriented language on a managed runtime. The execution model is the same two-stage idea as [Java's](/citadel/coding-languages/java): compile to portable bytecode, then let a runtime JIT-compile and execute it.

## The C# language

- **Object-oriented**, supporting encapsulation, abstraction, inheritance, and polymorphism. See [OOP](/citadel/interview/oop).
- **Statically typed and type-safe** — types checked at compile time, with guarantees that prevent whole classes of type errors and memory-safety issues.
- **Component-oriented** — designed around building and composing software components.
- **C-family syntax** — familiar from C, C++, or Java.

Notable language features, added over successive versions:

- **Properties, events, delegates** — for encapsulated state and event-driven code.
- **LINQ (Language Integrated Query)** — SQL-like querying over collections, XML, and databases, written directly in C#.
- **`async`/`await`** — asynchronous code that reads sequentially.
- **Generics** — reusable type-safe code across types.
- **Pattern matching** — richer type checks and data extraction.
- **Nullable reference types** — reference types are non-nullable unless declared otherwise, cutting `NullReferenceException`.
- **Records** — concise immutable data types.

## The .NET platform

C# is tightly bound to .NET, which has been through three eras:

- **.NET Framework** — the original, Windows-only implementation. Still supported for existing applications; not the target for new work.
- **.NET Core** — a from-scratch redesign: open-source, cross-platform (Windows, macOS, Linux), modular, built for performance and cloud-native workloads.
- **.NET 5, 6, 7, 8...** — from .NET 5 onward, Framework and Core merged into a single ".NET" with one SDK and one Base Class Library. Even-numbered releases (6, 8) are Long-Term Support.

### The Common Language Runtime

The CLR is the execution environment for all .NET languages (C#, F#, VB.NET). The pipeline:

1. The C# compiler translates `.cs` source into **Common Intermediate Language (CIL)** bytecode — also called MSIL — stored in assemblies (`.dll` or `.exe`).
2. At runtime the CLR takes over:
   - **Class loader** — loads CIL assemblies into memory.
   - **Bytecode verifier** — checks the CIL for type safety and security.
   - **JIT compiler** — translates CIL to native machine code for the current OS and CPU at runtime, so hot code runs natively. This is how CIL gets both portability and speed ([the general pattern](/citadel/coding-languages/coding-langs)).
   - **[Garbage collector](/citadel/interview/garbage-collection)** — automatic memory management.
   - **Security engine and exception handling.**

The idea maps directly onto Java's: compile once to CIL, run anywhere there is a CLR.

### The Base Class Library

A large standard library of reusable types: collections, file I/O, networking, data access, cryptography, string handling, XML, and more.

## Frameworks on .NET

- **ASP.NET Core** — cross-platform, high-performance web applications and APIs. Development models: **MVC**, **Razor Pages** (page-focused UI), **Blazor** (interactive client-side UI in C# instead of JavaScript, running via WebAssembly or server-side over SignalR), and **Web APIs** for REST services. Built for testability with dependency injection.
- **Entity Framework Core** — a cross-platform object-relational mapper. Work with the database through .NET objects and LINQ, with migrations for schema changes and providers for SQL Server, PostgreSQL, MySQL, SQLite, Cosmos DB.
- **.NET MAUI** — one C# codebase and UI for native mobile (iOS, Android) and desktop (Windows, macOS). The successor to Xamarin.Forms.
- **WPF and WinForms** — Windows desktop UI. WPF uses XAML for rich interfaces; WinForms is the older, simpler, event-driven model, still used for existing and some new applications.
- **Azure** — Microsoft's cloud has first-party .NET support: App Service, Azure Functions (serverless), Azure SQL, Cosmos DB, AKS, Azure DevOps.

## Tooling

- **Visual Studio** — the flagship IDE for .NET: coding, debugging, testing, profiling, deployment.
- **Visual Studio Code** — lightweight and cross-platform; the C# extension adds IntelliSense and debugging.
- **NuGet** — the .NET package manager and gallery for libraries and tools.
- **.NET CLI** — cross-platform command-line tooling to create, build, test, run, and publish projects; the basis for automation.
- **CI/CD** — integrates with Azure DevOps, GitHub Actions, Jenkins, TeamCity.

## In production: Stack Overflow

[Stack Overflow](/citadel/system-design/stack-overflow) is a large, high-traffic site built on the .NET stack — C#, ASP.NET MVC, and SQL Server — running on a relatively small number of on-premise Windows servers. It is a standing demonstration that a well-optimised .NET application scales far on modest hardware, even as a monolith.

## Key takeaways

- C# compiles to **CIL bytecode**; the **CLR** verifies it, JIT-compiles the hot paths to native code, and garbage-collects — the same two-stage model as the JVM.
- .NET went from Windows-only **Framework** to cross-platform **.NET Core**, unified from **.NET 5** into one SDK and library for web, mobile, desktop, cloud, and gaming.
- **ASP.NET Core** (web), **EF Core** (data), and **.NET MAUI** (cross-platform UI), plus **Visual Studio**, the **.NET CLI**, and **NuGet**, make up the working platform.
