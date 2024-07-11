---
title: Deployment Strategies - Blue-Green, Canary, and A/B
description: Cutting a new version into production is a risk you can shape. Four strategies — all-at-once, blue-green, canary, and A/B — and how each one trades infrastructure cost against blast radius and rollback speed.
date: 2024-07-11
draft: false
slug: /interview/deployments
tags:
  - System Design
  - Interview Prep
  - Deployment
---

The moment a new build takes production traffic is when its bugs become incidents. A deployment strategy is the shape you give that moment: how many users are exposed if the build is bad, how fast you can undo it, and how much extra infrastructure you pay to keep those two numbers low.

Four strategies cover most of the field. They are the production-rollout step of a [CI/CD pipeline](/citadel/interview/ci-cd), and they all exist to serve the same goals — no downtime, small blast radius, fast rollback, and a chance to validate under real load.

![Four deployment strategies as before-and-after diagrams: multi-service upgrades every service at once; blue-green swaps a fully staged environment for production; canary moves a fraction of identical instances to the new version then verifies before continuing; A/B runs versions side by side splitting traffic by percentage.](../images/deployments.png "Multi-service, blue-green, canary, and A/B rollouts. Canary and A/B both split traffic across versions; blue-green keeps a whole second environment ready to switch to.")

## All-at-once (multi-service)

Deploy the new version everywhere simultaneously. Easy to run and needs no extra infrastructure. But every user meets the new build at once, dependencies between updated services are hard to test in that instant, and rollback means redeploying the old version under fire. Fine for low-stakes internal tools; risky for anything user-facing.

## Blue-green

Keep two full production environments. **Blue** serves live traffic; **green** is the next version, deployed and tested in isolation. When green is verified, flip the router — all traffic moves to green, and blue becomes the standby for the next release or an instant rollback target.

Rollback is the strength: switching back to blue is one routing change, seconds not minutes. The costs are real: two production-grade environments running in parallel, and stateful components — database schema changes especially — that cannot simply be duplicated and swapped, so migrations have to be written to work against both versions during the cutover.

## Canary

Roll the new version to a small slice of servers or users — the canaries — while everyone else stays on the old one. Watch the canaries' error rates and latency; if they hold, widen the slice in steps until the fleet is converted. If they don't, route away from the canaries and you have exposed a few percent of traffic, not all of it.

Cheaper than blue-green — no full duplicate, just a few instances on the new build — and rollback is a traffic shift. The cost is operational: there is no separate staging here, so validation happens on production infrastructure with real users, and doing it safely needs solid monitoring and usually feature flags to gate the new behaviour independently of the deploy.

## A/B

Run two or more versions in production at once and route a defined percentage of users to each — 50% to V1.1, 25% to V1.2 — then compare them on chosen metrics: conversion, engagement, latency. Mechanically it resembles canary, but the goal is different: canary asks "is the new build safe?", A/B asks "which version performs better?".

Cheap way to get real data on a change before committing to it. The cost is the machinery — user bucketing, per-variant metrics, experiment analysis — and the discipline to halt a losing variant quickly and not leak a variant to users it was not meant for.

## The strategy follows the stakes

The honest default is: all-at-once for what does not matter, canary for what does, blue-green when a botched release must be undone in seconds and the second environment is affordable, A/B when the question is about impact rather than safety. Most mature teams run a mix, and lean on [immutable releases](/citadel/interview/12-factor-app) so that "roll back" always means "point at the previous artifact" rather than "rebuild and hope".
