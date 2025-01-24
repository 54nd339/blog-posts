---
title: Testing Strategy - The Pyramid, Doubles, and What to Automate
description: A test suite is a budget - the question is where to spend it. Many fast unit tests, fewer integration tests, a handful of end-to-end tests; the right kind of test double for each seam; and coverage as a signal, not a target. Plus the advanced techniques worth knowing.
date: 2025-01-24
draft: false
slug: /interview/testing-strategy
tags:
  - System Design
  - Interview Prep
  - Software Engineering
---

Every test you write costs time to author, time to run, and time to maintain when the code changes. A good suite spends that budget where it catches the most bugs per unit of cost — which turns out to mean *many small tests and few large ones*. Get the ratio wrong and you either have a slow, flaky suite nobody trusts, or a fast one that misses everything integration-shaped.

This is the shape of a suite that stays useful: the pyramid, the vocabulary of test doubles, and the techniques past plain example-based tests.

## The pyramid

Picture the suite as layers, widest at the bottom:

- **Unit tests** (the base, most numerous) — one class or function, no I/O, milliseconds each. They pin down logic: the fee calculation at the hour boundary, the parser on malformed input, the state machine's illegal transitions. Thousands of them run in seconds.
- **Integration tests** (the middle) — a few real components together: the repository against a real database, the HTTP handler against a real router, the queue consumer against a real broker. Slower (seconds), fewer, and they catch the bugs unit tests can't — wrong SQL, serialization mismatches, transaction boundaries.
- **End-to-end tests** (the tip, fewest) — the whole system through its real entry point (a browser, the public API). They're slow (minutes), flaky (timing, environment), and expensive to maintain, so you keep a small number covering the critical user journeys — sign up, check out, the money path.

The **anti-pattern** is the "ice-cream cone": lots of slow E2E tests, few unit tests, because "E2E tests are more realistic." They are, but a suite that takes 40 minutes and fails randomly gets ignored, and when one fails it doesn't tell you *where* the bug is. Push coverage down to the fastest layer that can catch each class of bug.

Some teams prefer the "testing trophy" framing — a fat integration layer, because for a web service most real bugs live at the seams between units, not inside them. The principle is the same: spend where the bugs are, and keep the slow layer thin.

## Test doubles

A **test double** stands in for a real dependency. The taxonomy (Gerard Meszaros / Martin Fowler):

- **Dummy** — passed to satisfy a signature, never used. `new Order(dummyLogger, ...)`.
- **Stub** — returns canned answers. `when(rates.get("USD")).thenReturn(1.0)`. Use it to *set up* a scenario.
- **Spy** — a stub that also records how it was called, so you can assert afterward. `verify(email).send(any())`.
- **Mock** — pre-programmed with expectations; the test fails if the calls don't match. Verifies *interactions*.
- **Fake** — a real, working, lightweight implementation: an in-memory repository, an embedded SQLite instead of Postgres, a fake clock. Behaves correctly, just not production-grade.

**Classical vs mockist.** The classical style uses real objects and fakes wherever practical, and only doubles what's slow or non-deterministic (the network, the clock, randomness). The mockist style doubles every collaborator and asserts on the interactions. Mockist tests are more isolated but couple tightly to *how* the code is structured — a refactor that changes which methods call which breaks tests even though behaviour didn't change. Default to classical; mock the boundaries (I/O, time, external services), not your own internal collaborators.

## TDD and BDD

- **TDD** (red–green–refactor) — write a failing test, make it pass with the least code, then refactor with the test as a safety net. Its real payoff isn't the tests; it's the *design pressure* — code that's hard to test is usually code with too many responsibilities or hidden dependencies, and TDD surfaces that immediately.
- **BDD** — express tests as behaviour scenarios (`Given a cart with 2 items, When I apply code SAVE10, Then the total drops by 10%`), often in Gherkin. Useful when non-engineers need to read or contribute to the specs; overhead when they don't.
- **Characterization tests** — for legacy code with no tests, write tests that capture *current* behaviour (even bugs) before you change anything, so you notice what your change affects.

## Beyond example-based tests

- **Property-based testing** (QuickCheck, Hypothesis, jqwik) — instead of asserting specific input→output pairs, assert a property that holds for *all* inputs, and let the framework generate hundreds of random cases and **shrink** any failure to a minimal reproducer. "Encoding then decoding returns the original", "the sorted list is a permutation of the input and is ordered." Finds edge cases you'd never think to write.
- **Fuzzing** — feed a function malformed and random input to find crashes and memory-safety bugs; see [fuzzing](/citadel/pentesting/fuzzing). Especially for parsers and anything handling untrusted bytes.
- **Mutation testing** (PIT, Stryker) — deliberately introduce bugs (flip a `<` to `<=`, delete a line) and check whether any test fails. A "surviving mutant" is a line your tests don't actually verify. It measures test *quality* in a way line coverage can't.
- **Snapshot / approval testing** — serialize a complex output (rendered HTML, a big JSON response) and diff it against a stored "approved" version. Cheap to write, but a magnet for rubber-stamping — review snapshot changes carefully.
- **Contract testing** (Pact) — for services, the consumer writes down what it expects from a provider's API as a contract; the provider's CI verifies it still satisfies every consumer's contract. Catches breaking API changes without running full E2E against every dependency.

## What to measure

- **Coverage** (line, branch) is a *floor*, not a target — 100% line coverage with no assertions catches nothing, and chasing the last 10% often means testing trivial getters. Watch for *drops*; don't worship the number. **Mutation score** is the better quality signal.
- **Flakiness** — quarantine flaky tests immediately (a flaky test in CI trains people to re-run until green, which hides real failures) and fix or delete them.
- **Suite duration** — keep the fast layer under a minute or two; developers won't run a suite they have to wait for. Parallelize; split slow tests into a separate stage.

## The one idea to keep

A test suite is a budget spent across a pyramid: many fast unit tests for logic, fewer integration tests for the seams (where most web-service bugs actually are), a handful of end-to-end tests for the critical journeys. Double only the boundaries — I/O, time, randomness — and prefer fakes to mocks so a refactor doesn't break tests that still describe correct behaviour. Treat line coverage as a floor and reach for property-based tests, fuzzing, and mutation testing when you need to know the tests are actually pulling their weight.
