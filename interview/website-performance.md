---
title: Website Performance - Core Web Vitals and How to Move Them
description: Performance is a set of user-centric metrics, not one number. What LCP, INP, and CLS measure, the supporting metrics behind them, and the full-stack changes — from image formats to TTFB — that improve each.
date: 2025-02-14
draft: false
slug: /interview/website-performance
tags:
  - System Design
  - Interview Prep
  - Web Performance
---

Website performance is measured, not felt — and it is measured as several metrics, each capturing a different part of the experience: how fast content appears, how fast the page reacts, whether it stays still while loading. It matters because it converts: Amazon found 100 ms of added latency cost about 1% of sales, Google saw half a second drop search traffic 20%, and Core Web Vitals are a search ranking signal.

This covers the metrics and the changes that move them. The frontend build side is the [frontend performance](/citadel/interview/fe-performance) post; this one spans the stack.

## Core Web Vitals

Google's three user-centric metrics, for loading, interactivity, and visual stability:

- **LCP (Largest Contentful Paint)** — when the largest element in the viewport (a hero image, a headline block) finishes rendering. Perceived load speed. Good: **≤ 2.5 s**.
- **INP (Interaction to Next Paint)** — the latency from a user interaction to the next visual update, across *all* interactions on the page. Replaced FID (which measured only the first input) as a Core Web Vital in 2024. Good: **≤ 200 ms**.
- **CLS (Cumulative Layout Shift)** — the summed score of unexpected layout jumps during load (an image loading late and shoving text down). Good: **≤ 0.1**.

## Supporting metrics

- **FCP (First Contentful Paint)** — when *any* content first renders; the first sign something is happening.
- **TTI (Time to Interactive)** — when the page reliably responds to input within 50 ms.
- **TBT (Total Blocking Time)** — time between FCP and TTI where the main thread was blocked by long tasks (JavaScript over 50 ms). The lab proxy for INP.
- **Speed Index** — how quickly above-the-fold content is visually populated.
- **TTFB (Time to First Byte)** — from request sent to first response byte received. Measures the server and network, and it is the floor under every other metric.

## The components

- **Loading** — FCP, LCP, TTFB.
- **Interactivity** — INP, TBT, TTI.
- **Visual stability** — CLS.
- **Runtime rendering** — smoothness of animation and scroll, aiming for 60 fps.
- **Perceived performance** — the subjective feel, shaped by loading indicators, **skeleton screens**, progressive loading of critical content first, and **optimistic UI** (update immediately, reconcile with the server after).

## Strategies, by where they act

**Assets:**
- Compress images, use WebP/AVIF, serve responsive sizes with `srcset` / `<picture>`.
- Minify HTML/CSS/JS, tree-shake unused code, `defer`/`async` non-critical scripts, inline critical CSS.
- Enable Gzip or Brotli compression on text responses.
- Audit third-party scripts (analytics, ads, widgets) — load them async or drop them.

**Delivery:**
- A [CDN](/citadel/interview/cdn) to serve static assets from an edge near the user.
- Browser [caching](/citadel/interview/caching) headers (`Cache-Control`, `ETag`) so repeat visits skip downloads.
- [HTTP/2 or HTTP/3](/citadel/interview/http) for multiplexing and header compression.

**Server (reducing TTFB):**
- Optimise backend logic and database queries; add [indexes](/citadel/interview/sql).
- Server-side [caching](/citadel/interview/caching) of hot data (Redis, Memcached).
- [Load balancing](/citadel/interview/load-balancing) across app servers.
- Offload slow work to background workers via a [queue](/citadel/interview/message-queue).

## Optimise the failing vital, then stop

The trap is optimising performance in general — there is always another millisecond. The discipline is: measure the three Core Web Vitals, find the one that fails its threshold, apply the strategies that move *that* metric (payload work for LCP, main-thread work for INP, reserved space for CLS), measure again, and stop when it passes. A site can be exhaustively tuned and still lose users at the one interaction where the thread was blocked.
