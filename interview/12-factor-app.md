---
title: Twelve-Factor App - What Each Factor Buys You
description: Heroku's twelve rules for a service that an orchestrator can start, stop, and duplicate without special handling. Each factor stated plainly, with the operational property it delivers and a concrete example.
date: 2025-03-11
draft: false
slug: /interview/12-factor-app
tags:
  - System Design
  - Interview Prep
  - Cloud Native
---

A service that runs identically on a laptop and in a hundred production containers, that a scheduler can kill and restart at will, and that ships a config change without a rebuild — that is what the twelve-factor rules add up to. They were written by Heroku engineers around 2011 to describe apps that behave well on a platform-as-a-service, and the same properties are what a Kubernetes cluster assumes of every pod.

The factors are not equally deep. A few are near-trivial today; three or four carry the real weight. Here is each one, the operational property it buys, and where it bites if you skip it.

## I. Codebase

One codebase in version control, many deploys. Every environment — a developer's machine, staging, each production region — runs the same tracked repository at some revision. If two deployable things share code, that code is a library with its own repo, not a copied directory. This is the rule everything else assumes: without it, "the same app" has no fixed meaning.

## II. Dependencies

Declare every dependency explicitly in a manifest (`requirements.txt`, `pom.xml`, `package.json`), and isolate them at runtime so nothing leaks in from the host. A checkout plus one install command should produce a working environment. Skip it and you get "works on my machine" — a library that happened to be present in one place and absent in another.

## III. Config

Anything that differs between deploys — connection strings, credentials, per-environment hostnames — lives in environment variables, not in the repository. The test: could you open-source the codebase right now without leaking a secret? Config in the environment is what lets one build run in dev, staging, and prod with nothing changed but the variables.

## IV. Backing services

Databases, queues, caches, mail services — anything reached over the network — are attached resources named by a URL in config. The app does not care whether Postgres is a local container or a managed cloud instance; swapping one for the other is a config edit, not a code change.

## V. Build, release, run

Three strictly separate stages. **Build** turns a commit into an executable bundle. **Release** joins that bundle to a config to make an immutable, versioned release. **Run** launches processes from a release. Releases are append-only, so a bad one rolls back by launching the previous release — you never rebuild to undo.

## VI. Processes

The app runs as stateless, share-nothing processes. Anything that must survive a request goes to a backing service. Memory and local disk are a scratchpad that can vanish at any moment. This is the factor that makes horizontal scaling and crash recovery work: any instance can serve any request, and a dead instance costs nothing.

## VII. Port binding

The app is self-contained and exports HTTP by binding a port itself, rather than being injected into an external web server like a WAR file into Tomcat. It starts the same way everywhere, and one service can become another's backing service just by handing over its URL.

## VIII. Concurrency

Scale by running more processes, not by making one process bigger. Split work into process types — web processes for requests, worker processes for background jobs — and scale each independently against its own load. A Procfile or a set of Kubernetes deployments expresses this directly.

## IX. Disposability

Processes start fast and stop cleanly. Fast startup means a new instance joins the pool quickly during a scale-up or after a crash. Graceful shutdown means catching `SIGTERM`, finishing the in-flight request, releasing locks and connections, and — for a worker — returning the current job to the queue. Elastic scaling and rolling deploys both depend on this.

## X. Dev/prod parity

Keep the gaps between environments small: in *time* (deploy hours after writing, not months), in *personnel* (the author is involved in the deploy), and in *tools* (the same backing services everywhere, not SQLite in dev and Postgres in prod). Containers and infrastructure-as-code exist largely to close this gap; the payoff is bugs found before production.

## XI. Logs

The app writes its event stream to `stdout`, unbuffered, and never manages log files or rotation. The execution environment captures that stream and routes it — to the console in development, to an aggregator like the ELK stack or Splunk in production. Log handling becomes an environment concern, not application code.

## XII. Admin processes

One-off tasks — database migrations, a REPL, a backfill script — run as processes against a specific release, using the same codebase and config as the long-running processes (`heroku run`, `kubectl exec`). They ship with the app and go through the same release cycle, so they never drift out of sync with the code they operate on.

## The three that carry the rest

If you only internalise a few: config in the environment (III), stateless processes (VI), and strict build/release/run (V). Those are what let a platform treat your process as interchangeable and disposable — start it anywhere, kill it without warning, roll it back by pointing at an older release. The rest mostly make those three achievable. Not every app needs all twelve from day one, but every deviation is a case where something — the scheduler, a new hire, a 3 a.m. rollback — has to be told about your app as a special case. Related reading: [CI/CD](/citadel/interview/ci-cd), [deployment strategies](/citadel/interview/deployments), [cloud native architecture](/citadel/interview/cloud-native), and [scaling an application](/citadel/interview/scalability).
