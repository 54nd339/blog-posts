---
title: Mobile App Development - Swift, Kotlin, Flutter, and React Native
description: Build a mobile app natively - Swift for iOS, Kotlin for Android - for best performance and platform fit, or cross-platform with Flutter or React Native to share one codebase. The languages and their features, how each cross-platform framework renders, and the architecture, state, persistence, and CI/CD that surround any real app.
date: 2020-12-10
draft: false
slug: /coding-languages/swift-kotlin
tags:
  - Programming Languages
  - Mobile Development
  - Swift
  - Kotlin
---

Building a mobile app starts with one decision: **native** — a separate app per platform in that platform's language — or **cross-platform** — one codebase for both. Native gets you the best performance and the tightest fit with each OS; cross-platform gets you faster delivery and less duplicated code. This post covers Swift and Kotlin for native work, Flutter and React Native for cross-platform, and the parts that are the same either way.

## Native iOS: Swift

Swift was introduced by Apple in 2014 to replace Objective-C for iOS, macOS, watchOS, and tvOS. It is designed to be **safe**, **fast** (often near C++), and **expressive**.

- **Static typing with inference** — type safety at compile time, but concise to write.
- **Safety features** — optionals make the presence or absence of a value explicit, cutting `nil`-dereference bugs; automatic reference counting (ARC) manages memory; structured error handling.
- **Protocol-oriented programming** — design around protocols (interfaces) rather than deep class hierarchies.
- **Modern syntax** — closures, generics, tuples, pattern matching.
- **Compiles to native machine code** optimised for Apple hardware, and interoperates with existing Objective-C.

Environment: **Xcode**. UI frameworks: **UIKit** (older, imperative) and **SwiftUI** (newer, declarative, across all Apple platforms with less code).

**Pros:** best performance, immediate access to new OS features, UX that matches iOS conventions, strong tooling. **Cons:** Apple-only — a separate Android app is still needed; a learning curve coming from other ecosystems.

## Native Android: Kotlin

Kotlin, by JetBrains, was endorsed by Google as a first-class Android language in 2017. It is a modern, more concise, safer alternative to [Java](/citadel/coding-languages/java) for Android — **concise**, **safe**, **interoperable**, **pragmatic**.

- **Static typing with strong inference** — safety with little verbosity.
- **Null safety** — the type system separates nullable from non-nullable types, catching `NullPointerException` at compile time.
- **Coroutines** — lightweight asynchronous code written in a sequential style.
- **Concise syntax** — data classes, extension functions, sealed classes, lambdas.
- **100% Java interoperability** — Kotlin calls Java and vice versa, so the whole Java library ecosystem is available.

Environment: **Android Studio** (built on IntelliJ IDEA). UI: the **Android UI toolkit** (XML layouts plus code) and **Jetpack Compose** (modern, declarative, in Kotlin — like SwiftUI).

**Pros:** concise, expressive, safe; excellent tooling; full Java interop; compiles to bytecode for the Android Runtime. **Cons:** Android-only; a small runtime adds slightly to APK size (usually negligible).

## Cross-platform: Flutter with Dart

- **Dart** — a client-optimised language from Google. Ahead-of-time compilation to native ARM/x64 for release builds; just-in-time compilation during development for stateful hot reload. See [compilation models](/citadel/coding-languages/coding-langs).
- **Flutter** — a UI toolkit for building natively compiled apps for mobile, web, desktop, and embedded from one codebase. Rather than use the platform's own widgets, Flutter draws every pixel itself with its **Skia** rendering engine (the same one behind Chrome and Android), which gives pixel-identical UI across platforms.
- **Features:** a large set of customisable widgets, a reactive framework, stateful hot reload, performance close to native.
- **Pros:** UI consistency, fast iteration, good performance from AOT compilation and direct rendering. **Cons:** Dart's third-party ecosystem is smaller than JavaScript's; app size is larger because the engine ships with the app; niche native features need platform channels.

## Cross-platform: React Native with JavaScript

- **Core idea** — build native apps with [JavaScript](/citadel/coding-languages/javascript) and React (Meta). Web developers who know React can build mobile apps.
- **How it renders** — components are written in React; React Native bridges the JavaScript thread (your logic) to the native UI threads, translating components into real native views (`<View>` becomes `UIView` on iOS, an Android `View` on Android). Not a web view.
- **Features:** React's declarative component model, the large JavaScript/React package ecosystem, Fast Refresh for quick iteration, the ability to drop to native modules when needed.
- **Pros:** reuse of JavaScript/React skills, substantial code sharing across platforms, huge ecosystem. **Cons:** the JavaScript bridge can be a bottleneck for heavy animation or computation; managing native modules across two platforms is fiddly; version upgrades can be painful; a truly native feel sometimes needs platform-specific tweaks.

## What every app needs regardless

- **Architectural patterns** — MVC, MVP, and MVVM adapted for mobile; **VIPER** (View, Interactor, Presenter, Entity, Router), popular on iOS for strict separation and testability; **RIBs** (Router, Interactor, Builder), from Uber, for business-logic isolation across both platforms. See [OOP](/citadel/interview/oop).
- **State management** — Redux or MobX (React Native); Provider, Riverpod, or BLoC (Flutter); ViewModel with StateFlow/SharedFlow (Kotlin).
- **Networking** — API calls over REST, GraphQL, or [gRPC](/citadel/interview/grpc), with JSON or Protobuf parsing.
- **Local persistence** — SQLite, Realm, Core Data (iOS), Room (Android), and key-value stores (UserDefaults / SharedPreferences).
- **Push notifications** — APNS (Apple) and FCM (Firebase).
- **CI/CD** — Fastlane, Bitrise, Codemagic, App Center, GitHub Actions, for automated builds and store deployment.
- **Testing** — unit tests, plus UI/integration tests with Espresso and UI Automator (Android) or XCUITest (iOS).
- **Store deployment** — packaging, signing, certificate management, and each store's review process.

## Choosing

Choose **native** (Swift, Kotlin) when performance is critical, you need new platform APIs the day they ship, you want UI that exactly matches the platform, or you have separate specialist teams. Choose **cross-platform** (Flutter, React Native) when time-to-market on both platforms matters, you want maximum code reuse, your team's strength is Dart or JavaScript/React, and the app does not depend on the newest native features.

## Key takeaways

- **Native**: Swift on iOS and Kotlin on Android — best performance and platform fit, two codebases.
- **Cross-platform**: Flutter renders its own pixels with Skia for identical UI; React Native bridges JavaScript to real native views. One codebase, some native nuance traded for speed of delivery.
- The language and framework are only part of it — architecture (MVVM, VIPER, RIBs), state management, persistence, networking, push, and CI/CD are shared work on every app.
