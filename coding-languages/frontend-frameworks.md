---
title: Frontend Frameworks - Components, Reactivity, and Rendering Strategies
description: React, Vue, Svelte, and Angular all solve the same problem - keeping the DOM in sync with your data without writing manual update code. They differ in how they detect what changed (a virtual DOM diff, reactive proxies, or a compiler) and in where the HTML is first built - the browser, the server, or at build time.
date: 2023-01-30
draft: false
slug: /coding-languages/frontend-frameworks
tags:
  - Programming Languages
  - Web
---

Without a framework, updating a web page means finding DOM nodes and mutating them by hand: `document.getElementById("count").textContent = state.count`. For a whole app that's hundreds of these, scattered, easy to get out of sync with the actual data, and impossible to reason about.

Every frontend framework exists to remove that. You write a function of your state that describes what the UI *should* look like, and the framework makes the real DOM match — computing the minimal set of changes. The frameworks differ in three places: how a UI is broken into pieces (components — they mostly agree here), how the framework *detects* what changed when state updates, and *where* the initial HTML is produced. Those are the axes worth understanding.

## The component model

All the major frameworks converged on **components**: a reusable unit that takes **props** (inputs from its parent), holds local **state**, renders some markup, and can nest other components. Data flows **down** (parent passes props to child); events flow **up** (child calls a callback the parent gave it). One-way data flow makes it possible to trace where any piece of displayed data came from.

Components have a lifecycle — created, updated, destroyed — and a way to run **side effects** (fetch data, subscribe to something, set a timer) tied to that lifecycle, with cleanup on destroy. React calls this `useEffect`; Vue has `onMounted`/`watchEffect`; the shape is the same.

## How change detection works — the real difference

When state changes, the framework has to figure out which DOM nodes to update. Three strategies:

### Virtual DOM + diffing (React)

`render()` produces a lightweight tree of plain objects describing the desired DOM (the **virtual DOM**). On a state change, React re-runs `render()`, produces a new virtual tree, **diffs** it against the previous one, and applies only the differences to the real DOM. `key` props help it match list items across renders so it moves nodes instead of recreating them.

The upside: the mental model is simple — "UI is a function of state, just re-render." The downside: the diff runs even for changes that affect one text node, so large component trees need manual optimisation (`memo`, `useMemo`, `useCallback`) to skip re-rendering subtrees whose props didn't change.

### Fine-grained reactivity (Vue, SolidJS, Svelte 5 "runes")

Wrap state in **reactive** containers. Vue uses ES `Proxy` objects that intercept reads and writes; when a component renders, Vue records *which* reactive values it read (dependency tracking); when one of those values is written, Vue re-runs exactly the computations that depended on it. SolidJS takes this to the limit — a component function runs *once*, and only the specific DOM nodes bound to a changed signal update; there's no re-render and no virtual DOM.

The upside: updates are surgical by default, no manual memoisation. The downside: the reactivity has rules (destructuring a reactive object can lose reactivity; you must use the framework's primitives), which is its own learning curve.

### Compiler (Svelte)

Svelte has no runtime diffing engine. Its **compiler** analyses your component at build time and generates imperative JavaScript that updates precisely the right DOM nodes when a specific variable changes. `count += 1` compiles to code that updates exactly the text node showing `count`. The shipped bundle is small (no framework runtime) and updates are direct.

The upside: least code, smallest bundle, fast updates. The downside: the "magic" happens at compile time, so what you write and what runs diverge more than in the others.

### Angular

Historically used **zone.js** to monkey-patch async APIs and trigger a change-detection pass over the component tree after any event; you could opt components into `OnPush` to skip subtrees. Angular is now moving to **signals** (fine-grained reactivity, like Vue/Solid) to make this precise and drop the zone.

## State management

Local component state covers most needs. For state shared across distant components:

- **Context / provide-inject** — pass a value down the tree without threading props through every level. Fine for low-frequency data (theme, current user).
- **Stores** — a central state container: Redux (explicit actions and reducers, time-travel debugging, verbose), Zustand / Jotai / Pinia / NgRx (lighter). Use for genuinely global, frequently-updated app state.
- **Server state** — data fetched from an API isn't really "client state"; libraries like TanStack Query / SWR handle caching, revalidation, and loading/error states so you're not reinventing them per component.

The common mistake is putting everything in a global store. Most state is local; reach for a store when the alternative is prop-drilling through many layers or duplicating state.

## Rendering strategies — where the HTML comes from

- **CSR / SPA** — the server sends a near-empty HTML shell and a JS bundle; the browser runs the framework and builds the whole page client-side. Fast subsequent navigation, but a slow first paint (blank until JS loads and runs) and poor SEO without extra work.
- **SSR** — the server runs the framework to produce full HTML per request, sends it (fast first paint, good SEO), then the client **hydrates** — attaches event listeners and reactivity to the server-rendered markup. Hydration itself costs CPU and delays interactivity.
- **SSG** — render every page to static HTML at *build* time. Instant, cacheable on a CDN, ideal for content that doesn't change per request (docs, blogs, marketing). **ISR** regenerates individual pages on a schedule or on demand.
- **Streaming SSR** — send the HTML in chunks as it's ready, so the user sees the shell immediately and slower parts (a personalised widget) stream in. React Suspense enables this.
- **Islands / partial hydration** — render the page mostly as static HTML and hydrate only the interactive bits ("islands"), shipping far less JS. Astro's model; also Qwik's "resumability" (no hydration pass at all).
- **React Server Components** — components that run only on the server, never ship their code to the client, and can directly access the database; interactive "client components" are the exception, marked explicitly.

**Meta-frameworks** package these: Next.js (React), Nuxt (Vue), SvelteKit (Svelte), Remix (React), Astro (multi). They add routing, data loading, bundling, and a default rendering strategy so you're not assembling it yourself.

## The one idea to keep

Every frontend framework lets you describe the UI as a function of state and keeps the DOM in sync for you; they differ in *how they detect what changed*. React re-renders and diffs a virtual DOM (simple model, needs manual memoisation at scale); Vue and Solid track which reactive values each computation read and re-run only those (surgical by default, reactivity has rules); Svelte compiles the update code at build time (smallest bundle, most "magic"). Orthogonally, pick where the HTML is first built — client, server per request, or build time — based on whether the content is per-user and how much first-paint speed and SEO matter.
