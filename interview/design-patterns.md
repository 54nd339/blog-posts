---
title: Design Patterns - The Gang of Four Catalog
description: Named solutions to recurring object-oriented design problems, and a shared vocabulary for them. The 23 Gang of Four patterns in three groups, each with its intent and the situation it fits.
date: 2024-09-05
draft: false
slug: /interview/design-patterns
tags:
  - System Design
  - Interview Prep
  - Design Patterns
---

A **design pattern** is a named, reusable arrangement of classes and objects that solves a design problem which keeps recurring. It is not code you copy — it is a template you adapt, and, just as usefully, a word two engineers can say instead of describing the whole structure.

The canonical set is the 23 patterns from *Design Patterns: Elements of Reusable Object-Oriented Software* (1994) by Gamma, Helm, Johnson, and Vlissides — the **Gang of Four (GoF)**. They fall into three groups by what they organise: object creation, object composition, and object communication. This is a catalogue of all 23 with intent and fit; the standing caveat is at the end.

## Creational: how objects get made

These decouple a system from the specifics of what it instantiates and how.

| Pattern | Intent | Fits when |
| --- | --- | --- |
| **Singleton** | one instance, one global access point | a genuinely single resource — a config registry, a connection pool |
| **Factory Method** | an interface for creation; subclasses pick the concrete class | a class can't know in advance which type it must create |
| **Abstract Factory** | create families of related objects without naming concretes | the product set must stay internally consistent (a UI toolkit per OS) |
| **Builder** | separate step-by-step construction from the final representation | an object with many optional parts, or a complex assembly process |
| **Prototype** | create new objects by cloning a prototypical instance | copying is cheaper than constructing, or the concrete type is hidden |

## Structural: how objects compose

These assemble objects into larger structures that stay flexible.

| Pattern | Intent | Fits when |
| --- | --- | --- |
| **Adapter** | wrap one interface to look like another | integrating a component whose interface doesn't match |
| **Decorator** | add responsibilities to an object dynamically by wrapping it | you want optional features without a subclass per combination |
| **Facade** | one simple interface over a complex subsystem | callers need the common 20% of a large library |
| **Proxy** | a stand-in that controls access to another object | lazy creation, access control, caching, remote calls |
| **Composite** | treat individual objects and trees of them uniformly | part-whole hierarchies — UI trees, filesystems |
| **Bridge** | split an abstraction from its implementation so both vary freely | avoiding an M×N class explosion (shapes × rendering backends) |
| **Flyweight** | share common state across many small objects | huge counts of near-identical objects — glyphs, tiles |

## Behavioral: how objects communicate

These assign responsibility and define the patterns of interaction between objects.

| Pattern | Intent | Fits when |
| --- | --- | --- |
| **Observer** | notify all dependents when one object changes state | event systems, a view reacting to a model ([UI patterns](/citadel/interview/ui-patterns)) |
| **Strategy** | a family of interchangeable algorithms behind one interface | swapping sort orders, pricing rules, validation policies |
| **Command** | package a request as an object | undo/redo, queued or logged operations, deferred execution |
| **Chain of Responsibility** | pass a request along handlers until one takes it | middleware stacks — logging, auth, validation in sequence |
| **Iterator** | sequential access to a collection without exposing its internals | uniform traversal across list, set, map |
| **Mediator** | centralise how a set of objects interact | many components with tangled mutual references |
| **Memento** | capture and restore an object's state without breaking encapsulation | snapshots for undo, save points |
| **State** | change behaviour by switching an internal state object | an object that acts like a small state machine |
| **Template Method** | fix an algorithm's skeleton, let subclasses fill steps | several variants share a structure, differ in details |
| **Visitor** | add operations over an object structure without changing its classes | new operations across a stable class hierarchy (an AST) |
| **Interpreter** | represent a grammar and evaluate sentences in it | a small, stable domain language |

## What patterns buy, and the caveat

The payoff is real: a solution that many people have already hit the edges of, a shared vocabulary for design review, and code that a new reader recognises. Systems that need to change and extend are where they earn their keep.

The caveat is equally real, and it is why the GoF book itself warns against it: a pattern applied to a problem that does not have it adds indirection and vocabulary for nothing. Start from the problem, understand it fully, and reach for a pattern only when it genuinely fits — not the reverse. The [SOLID principles](/citadel/interview/oop) are the design instincts that tell you *when* a pattern is warranted; the patterns are the shapes you reach for once it is.
