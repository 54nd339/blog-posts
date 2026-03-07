---
title: Low-Level Design - How to Attack the Machine-Coding Round
description: The LLD interview gives you 60-90 minutes to turn a vague problem into clean, extensible classes. A repeatable method - clarify scope, list use cases, find the nouns and verbs, draw the interfaces, then apply the three or four patterns that actually come up - beats improvising every time.
date: 2026-03-07
draft: false
slug: /system-design/object-oriented-design
tags:
  - System Design
  - Low-Level Design
  - OOD
---

The low-level design round hands you a small system — a parking lot, an elevator, a card game — and asks for working code or a class diagram in about an hour. It's not testing whether you can build a parking lot. It's testing whether, given an underspecified problem, you produce a model that's clean, that encapsulates its rules, and that survives the interviewer saying "now add electric-vehicle charging" without a rewrite.

Improvising this under time pressure goes badly. What works is a fixed sequence: pin down scope, enumerate the operations, extract the domain model, define the seams where behaviour will vary, and reach for a small set of patterns you've seen before. This post is that sequence.

## Step 1: clarify and scope

Spend the first five minutes narrowing the problem. Ask:

- **What are the core operations?** (Park a car / retrieve it / pay. Not: build a loyalty programme.)
- **What's explicitly out of scope?** Say it out loud and get agreement — "I'll assume one physical site, not a chain" — so you're not judged for omitting something the interviewer didn't want.
- **What are the dimensions of variation?** Multiple vehicle types? Multiple pricing schemes? Those become your extension points.
- **Scale?** Usually "single process, in-memory, no database" — confirm it. LLD is about object structure, not distribution.

Write two or three concrete scenarios on paper: "a car enters, gets a ticket, parks in spot L2-14, leaves after 3h20m, pays ₹120." These are your acceptance tests and they keep the design honest.

## Step 2: actors and use cases

List who interacts with the system and what each can do. For a parking lot: a *driver* (enter, park, exit, pay), an *admin* (add/remove spots, set rates), maybe a *display* (show free counts). Each verb is a method on some object; the set of verbs is your public API. If a verb doesn't map cleanly to an actor's need, question it.

## Step 3: the domain model — nouns and verbs

Underline the nouns in the problem statement and your scenarios: *parking lot, floor, spot, vehicle, ticket, payment, rate*. Most become classes or enums. Then:

- **Entities vs value objects.** A `Ticket` has identity and a lifecycle (issued → paid → closed) — an entity. A `Money` or a `TimeRange` is defined entirely by its values and is immutable — a value object. Value objects prevent a lot of bugs (you can't accidentally mutate a shared `Money`).
- **Relationships.** `has-a` (a `Floor` has `Spot`s) is composition — model it as a field. `is-a` (an `ElectricCar` is a `Vehicle`) is inheritance — but prefer composition unless the subtype genuinely substitutes for the base everywhere (Liskov).
- **Where do the rules live?** "A motorcycle fits in any spot; a truck needs a large spot" belongs *on the spot or the vehicle*, not in a giant `if` in the parking-lot class. Push behaviour to the object that owns the data.

Sketch the class diagram: boxes for classes, lines for relationships, and — this is the important part — **interfaces first**. Name the things that will vary as interfaces before you write any concrete class.

## Step 4: the patterns that actually come up

You don't need the full Gang of Four. In LLD rounds, four or five patterns cover almost everything:

- **Strategy** — a family of interchangeable algorithms behind one interface. Pricing (`PricingStrategy`: flat / hourly / tiered), spot assignment (`nearest` / `by-type`), dispatch (elevator `SCAN` / `nearest-car`). Whenever the problem has "the rule for X could be different," that's a Strategy.
- **State** — an object whose behaviour changes with its mode, where each mode handles the same events differently. A vending machine (`Idle`, `HasMoney`, `Dispensing`), an ATM, an order. Replaces a sprawling `switch (currentState)`.
- **Factory** — centralize object creation when the concrete type depends on input. `VehicleFactory.create("truck")`, `TicketFactory`.
- **Observer** — one object's change notifies many. A `Spot` freeing up updates every `Display`; a button press notifies the elevator controller.
- **Singleton** — one instance for the whole process (the `ParkingLot` itself, a config registry). Use sparingly — it's global state — and make it thread-safe if it matters.
- **Decorator / Composite / Command / Builder** show up occasionally (add-ons on a coffee order, a tree of UI components, undoable operations, a complex constructor).

Naming the pattern isn't the goal; using the right seam is. If pricing is a Strategy, adding "weekend rates" is a new class, not an edit to existing code — the **open-closed principle** in practice.

## Step 5: SOLID, concretely

- **Single responsibility** — `ParkingLot` coordinates; it doesn't also compute fees and format receipts. Split those out.
- **Open-closed** — new behaviour via new classes (a new `PricingStrategy`), not by editing a `switch`.
- **Liskov** — a subtype must work anywhere its base does. If `ElectricCar` throws from a method `Car` supports, the hierarchy is wrong.
- **Interface segregation** — many small interfaces over one fat one. A `Chargeable` interface only for EV spots, not a `chargeEV()` on every `Spot`.
- **Dependency inversion** — `ParkingLot` depends on `PricingStrategy` (the interface), and the concrete strategy is injected. This is what makes it unit-testable.

## Step 6: concurrency

Ask whether it's needed — many LLD problems are single-threaded. If it is (two cars racing for the last spot, two threads on a shared cache):

- Identify the **shared mutable state** and the smallest critical section around it.
- Prefer a lock per resource (per floor, per spot bucket) over one global lock.
- Prefer **immutability** (value objects, copy-on-write) where you can — no lock needed.
- Make the thread-safe singleton actually thread-safe (holder idiom, or an eager instance).
- Mention `ConcurrentHashMap` / atomic counters rather than hand-rolled locking where the language offers them.

## Step 7: deliver

- A sensible package layout (`model/`, `service/`, `strategy/`).
- A `main` or a demo method that walks your scenarios from step 1.
- Unit tests for the **rules** — fee at the hour boundary, last-spot contention, invalid transitions. Not getters.
- Then iterate: the interviewer will add a requirement, and a good design absorbs it as a new class or a new strategy.

## The one idea to keep

The LLD round rewards a method, not raw modelling speed: clarify scope and name the dimensions of variation first, extract classes from the nouns and push each rule onto the object that owns its data, then turn every "this could be different" into an interface — usually a Strategy or a State. SOLID is the check that the seams are in the right place: adding the feature the interviewer asks for next should be a new class, not an edit to an old one.
