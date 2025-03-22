---
title: Frontend Performance - Payload, Loading, and Rendering
description: "Three levers make a web page fast: how many bytes it ships, when it loads them, and how much it renders. The techniques under each, from Brotli and tree shaking to code splitting, resource hints, and list virtualization."
date: 2025-03-22
draft: false
slug: /interview/fe-performance
tags:
  - System Design
  - Interview Prep
  - Web Performance
---

Amazon found every 100 ms of added latency cost about 1% of sales; Google saw half a second of extra load time drop traffic by 20%. Page speed also feeds Core Web Vitals, which Google uses as a ranking signal. The work of making a page fast splits into three levers: the size of what it downloads, the order and timing of those downloads, and the amount of rendering it does. This walks each.

The infrastructure side — where these assets are served from and cached — is the [website performance](/citadel/interview/website-performance) post; here the focus is the frontend build and browser behaviour.

## Payload: ship fewer bytes

- **Compression.** Servers and CDNs compress text assets — HTML, CSS, JS, JSON — before sending; the browser inflates them. Brotli beats gzip on ratio and is the current default. Configured once, applies to everything.
- **Tree shaking.** With ES module syntax (`import`/`export`), bundlers (Webpack, Rollup, Vite) trace which exports are actually reached and drop the rest from the bundle — dead-code elimination across your dependencies.
- **Minification.** Strip whitespace, comments, and newlines, and shorten local identifiers, without changing behaviour. Build tools do this automatically as a final pass.

## Loading: fetch the right things at the right time

- **Code splitting.** Break one large bundle into chunks so the browser downloads only what the initial view needs, and pulls the rest on navigation or interaction. The single biggest lever on Time to Interactive.
- **Dynamic `import()`.** The language-level way to load a module on demand — `import('./chart.js')` returns a promise resolving to the module. Use it to defer route components, below-the-fold features, and rarely used code paths.
- **Script attributes.** `defer` downloads a script without blocking HTML parsing and runs it after parsing, in document order. `async` also does not block parsing but runs as soon as it arrives, order not guaranteed.
- **Resource hints** (`<link rel="...">`):
  - `preload` — fetch a resource the current page needs soon (a critical font, a late-discovered stylesheet) at high priority, without executing it yet. Set `as` so the browser prioritises and applies policy correctly.
  - `prefetch` — fetch, at idle and low priority, something a *future* navigation will likely need.
  - `dns-prefetch` — resolve a third-party domain's DNS ahead of time.
  - `preconnect` — go further: DNS plus TCP plus TLS to an origin in advance, so the first real request to it is fast.
- **HTTP/2 and HTTP/3 stream priority.** The client marks the relative priority of resource streams on one connection so the server sends the important bytes first. Covered in [HTTP](/citadel/interview/http).

## Rendering: draw less

- **Conditional rendering.** Only mount the components currently needed. Frameworks lean on this directly (`v-if`, `*ngIf`, `{#if}`), keeping unused subtrees out of the render entirely.
- **List virtualization (windowing).** For a list of thousands of rows, render only the ones in the viewport plus a small buffer, swapping items in and out of the DOM as the user scrolls. Turns an O(n) layout into an O(viewport) one.
- **Lazy loading.** Defer offscreen images and iframes until they near the viewport. `loading="lazy"` on `<img>` and `<iframe>` is the built-in way; the Intersection Observer API gives finer control for custom cases.

## What holds it up

These techniques assume support underneath: a [CDN](/citadel/interview/cdn) serving the optimised assets from the edge; modern image formats (WebP, AVIF) for smaller pictures at equal quality; correct HTTP [caching](/citadel/interview/caching) headers (`Cache-Control`, `ETag`) so repeat visits skip downloads; build tools (Vite, Webpack) automating minification, tree shaking, and splitting; and server-side rendering or static generation (Next.js, Nuxt) so the browser gets meaningful HTML before the JavaScript arrives.

## Measure the metric you care about, not the technique

Every item here trades effort for milliseconds against a specific metric — payload work moves Largest Contentful Paint, splitting and hints move Time to Interactive, rendering work moves interaction latency. Pick the one Core Web Vital that is failing, apply the levers that move it, measure again, and stop when it passes. A page can be exhaustively "optimised" and still feel slow if the effort went somewhere the user never waits.
