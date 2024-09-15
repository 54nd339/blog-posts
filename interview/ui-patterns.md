---
title: UI Architecture Patterns - MVC, MVP, MVVM, VIPER
description: All four split UI code into data, display, and the logic between them, so the logic can be tested without a screen. What each pattern's intermediary is, how it wires the components, and the boilerplate-versus-testability trade.
date: 2024-09-15
draft: false
slug: /interview/ui-patterns
tags:
  - System Design
  - Interview Prep
  - Architecture
---

Without structure, UI code becomes a knot — display, input handling, and business logic tangled in one class, impossible to test and dangerous to change. **UI architecture patterns** untangle it by applying **separation of concerns**: split the code into components with single responsibilities. All of them share two:

- **Model** — the data and business logic, the source of truth.
- **View** — what the user sees and interacts with.

They differ in the *intermediary* between those two and how it is wired.

![Five panels wiring the same components differently. MVC: the view notifies a controller, the controller updates the model, the view gets data from the model. MVP: the view and presenter talk through an interface both ways; the presenter updates and gets data from the model. MVVM: the view is bound to a view-model that notifies it of changes; the view-model updates and is notified by the model. MVVM-C: MVVM plus a coordinator controlling navigation. VIPER: the view binds to a presenter, which drives an interactor that manages entities, with a router handling navigation.](../images/ui-patterns.png "Each pattern's intermediary — controller, presenter, view-model — and how tightly the view is coupled to the model. Source: ByteByteGo.")

## MVC — the classic

**Model, View, Controller.** ~50 years old, the ancestor of the others. The **controller** sits between: it receives user input from the view, updates or queries the model, and picks a view to render. The model may notify views directly of changes.

- **Pros** — a first separation of concerns; several views can share one model.
- **Cons** — the controller swells into a "massive view controller" / god object in complex UIs; the view often still knows about the model, so coupling is looser than ideal; the controller's ties to the UI framework hurt testability.
- Web frameworks (Rails, Django, Spring MVC), desktop apps.

## MVP — enhancing testability

**Model, View, Presenter.** The **view goes passive** — it only displays what it is told and forwards input events — and all UI logic moves to the **presenter**, which talks to the view through an interface. One presenter per view.

- **Pros** — the presenter has no direct UI-framework dependency, so it unit-tests easily; the view is dumb.
- **Cons** — interface contracts between view and presenter add boilerplate; the presenter can still grow large.
- Android, Windows Forms.

## MVVM — data binding

**Model, View, ViewModel.** The **view-model** exposes the model's data as observable properties and commands, and knows nothing about the specific view. **Data binding** keeps the view and view-model in sync automatically: the view updates when a bound property changes, and forwards input by invoking commands.

- **Pros** — the view-model has no view reference and tests cleanly; binding removes most glue code; good for state-heavy UIs.
- **Cons** — binding can feel like magic and is hard to debug; overkill for simple screens; a large view-model still gets unwieldy.
- WPF, Angular, Vue, Android Jetpack.

## VIPER — maximum separation

**View, Interactor, Presenter, Entity, Router** — iOS, granular. The **interactor** holds one use case's business logic and manipulates **entities** (plain data); the **presenter** holds view logic and mediates; the **router** owns navigation between screens.

- **Pros** — every component has one job and tests in isolation; parallel work across a large team.
- **Cons** — heavy boilerplate and many files per screen; managing the five-way communication is itself complex; steep learning curve.
- Large, complex iOS apps.

**MVVM-C** is a lighter middle ground: MVVM plus a **coordinator** for navigation, VIPER's router idea without the rest.

## The pattern scales with the app, not the other way around

Every step from MVC to VIPER buys more testability and clearer boundaries by adding more components and more wiring. That is worth it for a large app with a big team and strict test requirements, and pure overhead for a settings screen. Match the pattern to the complexity you actually have. On the web, [Flux and Redux](/citadel/interview/programming-paradigms) take a different angle — one predictable, centralised state store — and the [software architecture](/citadel/interview/software-architecture) post places these in the broader separation-of-concerns family.
