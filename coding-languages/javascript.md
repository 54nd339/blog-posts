---
title: JavaScript and TypeScript - The Event Loop and the Ecosystem
description: JavaScript runs one thread and never blocks it - the event loop, call stack, callback queue, and microtask queue explain how. Then TypeScript's static types, the frontend frameworks (React, Angular, Vue, Svelte, Solid), Node.js on the server, and the tooling around all of it.
date: 2020-11-02
draft: false
slug: /coding-languages/javascript
tags:
  - Programming Languages
  - JavaScript
  - TypeScript
  - Web
---

JavaScript started in 1995 as a way to add small interactions to web pages. It now runs frontends, backends, mobile apps, and desktop apps. The language has one quirk that explains most of its behaviour — it runs your code on a single thread and still manages not to freeze — and a large ecosystem built on top. This post covers the core execution model, then TypeScript, frameworks, Node.js, and tooling.

## The execution model

JavaScript was created by Brendan Eich at Netscape and is standardised as ECMAScript. It is often called interpreted, but modern engines — V8 (Chrome, Node.js), SpiderMonkey (Firefox), JavaScriptCore (Safari) — [JIT-compile](/citadel/coding-languages/coding-langs) hot code to native machine code at runtime, so "interpreted" is only loosely true.

### One thread, an event loop

JavaScript runs all your application code on a **single main thread**. One operation at a time. So how does it handle a network request, a timer, or a click without locking up the page while it waits? By handing those off and getting on with other work. The pieces:

1. **Call stack.** Function calls push onto the stack; synchronous code runs here, top frame first.
2. **Web APIs / Node APIs.** When the code hits an async operation — `setTimeout`, a `fetch` request, a DOM event listener — JavaScript does not wait. It hands the operation to the browser's Web APIs (or Node's C++ layer), which run it off the main thread.
3. **Callback queue (task / macrotask queue).** When such an operation finishes — the timer elapses, the response arrives — its callback is placed in the callback queue.
4. **Microtask queue.** Promise callbacks (`.then`, `.catch`, `.finally`, and the continuations of `async`/`await`) go into a separate, higher-priority queue when the promise settles.
5. **The event loop.** A continuous process: whenever the **call stack is empty**, it drains the *entire* microtask queue onto the stack, one task at a time; only then does it take *one* task from the callback queue. Repeat forever.

That ordering — all microtasks before the next macrotask — is why a `Promise.resolve().then(...)` runs before a `setTimeout(..., 0)` scheduled just before it.

### Other core characteristics

- **Dynamic typing.** Variables have no declared type; a variable can hold a number then a string. Flexible, and the source of a class of runtime bugs that TypeScript exists to catch.
- **Prototype-based objects.** Objects inherit directly from other objects (their prototype). The ES6 `class` keyword is syntax over this, not a separate mechanism.
- **First-class functions.** Functions are values: assign them, pass them as arguments (`map`, `filter`, `reduce`), return them. This is what makes functional patterns natural.
- **Automatic [garbage collection](/citadel/interview/garbage-collection).** Unreachable memory is reclaimed in the background; no manual free.
- **Two homes.** In the browser it manipulates the [DOM](/citadel/coding-languages/html), handles events, and makes `fetch` calls. On the server, via **Node.js**, it uses the same V8 engine with an event-driven, non-blocking I/O model.

## TypeScript

As a JavaScript codebase grows, dynamic typing gets expensive to manage. TypeScript, from Microsoft, is a **statically typed superset** of JavaScript: every valid JavaScript program is a valid TypeScript program, and TypeScript adds optional type annotations, interfaces, generics, enums, tuples, and union types.

It is not executed directly. The compiler (`tsc`) **transpiles** it to plain JavaScript, which then runs anywhere JavaScript does.

- **Pros:** many errors caught at compile time, code that is easier to read and refactor, better editor autocompletion and navigation, and it scales to large teams and codebases.
- **Cons:** an added build step, a type system to learn, and occasionally more verbose code (though inference helps).

It is the primary language for Angular and increasingly the default for React, Vue, and Node projects.

## Frontend frameworks

The rise of single-page applications drove a wave of component-based frameworks. They give structure, encourage reusable components, and manage updates to the page efficiently.

- **React** (Meta) — a library for building UIs from reusable components, using a **virtual DOM** to compute minimal real-DOM updates. UI is written in **JSX**. React is only the view layer; routing (React Router) and state management (Redux, Zustand, Context) come from elsewhere — flexible, but more decisions.
- **Angular** (Google) — a complete, opinionated framework for large applications. Written in and requiring TypeScript. Ships component architecture, dependency injection, routing, forms, and an HTTP client out of the box. Steeper learning curve; can feel heavy for small projects.
- **Vue** — a progressive framework aiming for approachability. Virtual DOM, component architecture, an HTML-based template syntax, reactive data binding, and official routing (Vue Router) and state (Pinia). Gentle learning curve, excellent docs.
- **Svelte** — a compiler, not a runtime library. It compiles components at build time into imperative vanilla JavaScript that updates the DOM directly — no virtual DOM shipped to the browser. Exceptional performance and tiny bundles; smaller ecosystem.
- **SolidJS** — declarative like React (JSX), but with fine-grained reactivity and no virtual DOM. Compiles templates to targeted DOM operations. Benchmarks near the top; newer, smaller ecosystem.

**Meta-frameworks** add server-side rendering and static generation on top: **Next.js** (React) and **Nuxt** (Vue) — SSR, static generation, file-system routing, API routes, image optimisation.

**Historically significant:** **jQuery** vastly simplified DOM traversal, events, and AJAX in the 2000s ("write less, do more"). Modern native APIs and component frameworks have displaced it in new work, but a large amount of legacy code still runs on it.

## Backend: Node.js and Express

- **Node.js** — a JavaScript runtime on V8 with an event-driven, non-blocking I/O model. Efficient for I/O-bound workloads: REST APIs, real-time servers, microservices, CLI tools. Its module ecosystem, npm, is the largest software registry in the world.
- **Express.js** — a minimal, unopinionated Node web framework: routing, middleware, template-engine integration. The de facto standard, and the "E" in the MEAN and MERN stacks.

## Graphics

- **Three.js** — a cross-browser library over WebGL for hardware-accelerated 3D in the browser. Abstracts WebGL into scenes, cameras, lights, materials, meshes, and model loaders. Used for product configurators, games, data visualisation, and web AR/VR.

## Tooling

- **Package managers:** **npm** (the default, and the registry) and **Yarn** (an alternative, largely compatible with `package.json`).
- **Bundlers and build tools:** **Webpack** — a configurable static module bundler with code splitting, lazy loading, and tree shaking. **Vite** — serves native ES modules during development for near-instant startup and hot module replacement, and uses Rollup for optimised production builds.
- **CSS frameworks** used alongside JS frameworks: **Bootstrap** (pre-styled components and a responsive grid) and **Tailwind CSS** (low-level utility classes composed in markup). See [CSS](/citadel/coding-languages/css).

## Key takeaways

- JavaScript runs one thread and stays responsive by offloading async work and processing its callbacks through the **event loop**: empty call stack → drain all microtasks → take one macrotask → repeat.
- **TypeScript** adds static types on top, transpiling back to JavaScript; worth the build step on anything large.
- Frontend work centres on component frameworks — **React, Angular, Vue**, with **Svelte** and **Solid** trading ecosystem size for performance — and **Next.js / Nuxt** for rendering strategy.
- **Node.js with Express** is the standard JavaScript backend.
- **npm/Yarn**, **Webpack/Vite**, and **Bootstrap/Tailwind** are the supporting cast. The right choice is always context-dependent.
