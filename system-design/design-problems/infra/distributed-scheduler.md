---
title: Designing a Distributed Job Scheduler
description: Run a job at 3am, or every five minutes, or once ten seconds from now - reliably, across a worker fleet, without ever running the same job twice or dropping one on the floor. The design is a durable job store, a dispatch mechanism, and leased execution with idempotency.
date: 2025-08-21
draft: false
slug: /system-design/distributed-scheduler
tags:
  - System Design
  - Design Problems
---

`cron` runs jobs on a schedule. It also runs on one machine, so if that machine is down at 3am the nightly report doesn't happen, and nobody finds out until morning. Scale up — a job queue that fires reminders, a billing run, retries with backoff, a "send this email in 24 hours" API — and you need cron's behaviour spread across a fleet that can lose nodes without losing jobs or double-executing them.

The design is three parts: a durable place to record what should run and when, a mechanism to notice when a job is due and hand it to a worker, and an execution contract that survives a worker dying mid-job.

## What it has to do

- **One-shot** ("run at time T", "run in N seconds") and **recurring** (cron expression).
- **At-least-once execution** with a bound on lateness.
- Workers can crash; jobs must not be lost or run twice (or, if run twice, that must be harmless).
- Visibility: history, why a job failed, manual re-trigger.
- Scale to millions of scheduled jobs and thousands of executions per second.

## The job store

A database table (or a partitioned key-value store) with a row per job:

```
job_id | type | payload | schedule | next_run_at | state | attempts | lease_owner | lease_expires_at
```

`state` is a small state machine: `PENDING → CLAIMED → RUNNING → SUCCEEDED | FAILED`. The load-bearing index is on `(state, next_run_at)` so "give me the jobs that are due" is a cheap range query.

For recurring jobs, store the cron expression and, on each successful run, compute the next fire time and set `next_run_at` — the row is reused, not recreated.

## Dispatch: how a due job reaches a worker

Three approaches, roughly in order of scale:

- **Polling.** Scheduler nodes query `SELECT ... WHERE state = 'PENDING' AND next_run_at <= now() LIMIT n FOR UPDATE SKIP LOCKED`, claim the rows, and dispatch. `SKIP LOCKED` lets many scheduler nodes poll the same table without blocking each other. Simple, and fine to a few thousand jobs/sec. The poll interval bounds lateness (poll every second ⇒ up to ~1s late).
- **Timing wheel.** Keep an in-memory hierarchical set of buckets by fire time; a tick advances the wheel and fires the current bucket. O(1) to schedule and to fire, near-zero lateness. Needs the wheel rebuilt from the store on restart, and the store is still the source of truth. Good when you have many short-delay jobs.
- **Two-tier.** A cheap store holds far-future jobs; a background process moves jobs into a "due soon" queue (or an in-memory wheel) as their time approaches. Keeps the hot path small.

**Thundering herd at round times.** Everyone schedules "midnight" and "top of the hour," so those buckets are huge. Add jitter when accepting a job (`fire_at + random(0, spread)` for jobs that tolerate it), and rate-limit dispatch so a giant bucket drains over a few seconds instead of all at once.

## Execution: leases and idempotency

A worker claims a job by writing `lease_owner = me, lease_expires_at = now + T, state = RUNNING`. It runs the job, then marks `SUCCEEDED`. If the worker dies, the lease expires; a reaper (or the next poll) finds `state = RUNNING AND lease_expires_at < now()` and returns the job to `PENDING` for another worker.

That reaping is exactly why execution is **at-least-once**: a worker can finish the job and die *before* writing `SUCCEEDED`, and the job runs again. So jobs must be **idempotent** — carry an idempotency key (the `job_id` plus the scheduled fire time) and make the effect safe to repeat: upsert instead of insert, check-then-act against a ledger, dedupe the outgoing email by key. True exactly-once would need the job's side effect and its state update in one transaction, which you only have if the side effect is in the same database.

Long-running jobs should **heartbeat** — periodically extend their lease — so a 30-minute job isn't reaped at the 5-minute lease timeout.

## Failure handling

- **Retries** with exponential backoff and jitter: on `FAILED`, if `attempts < max`, set `next_run_at = now + base * 2^attempts + random`, back to `PENDING`.
- **Dead-letter**: after `max` attempts, move to a `DEAD` state for inspection instead of retrying forever.
- **Poison jobs**: a job that crashes its worker (not just fails) will be reaped and retried, crashing the next worker too. Cap re-leases per job and quarantine after a threshold.
- **Missed window policy**: the scheduler was down from 02:55 to 03:10 and a 03:00 job's time passed. Decide per job type: run it now (catch-up), skip this occurrence, or run once regardless of how many were missed. Airflow calls this `catchup`.

## Coordination

For recurring jobs you must not have two scheduler nodes both firing the "midnight report." Options: **partition** the job space (each scheduler owns jobs whose `job_id` hashes to its shard, reassigned on membership change) or **leader election** (one scheduler is the dispatcher; see [consensus](/citadel/interview/consensus)) with the others hot standby. Partitioning scales better; leader election is simpler to get correct.

Workers autoscale on queue depth (jobs in `PENDING` past their fire time) — the standard [scaling](/citadel/interview/scalability) signal.

## The one idea to keep

A distributed scheduler is a durable job table with a `(state, next_run_at)` index, a dispatcher that range-queries it for due work, and leased execution: a worker claims a job with an expiring lease, and a reaper re-queues anything whose lease lapses. That reaping makes execution at-least-once, so every job needs an idempotency key. Coordinate recurring jobs by partitioning the job space or electing one dispatcher, and decide up front what "we were down when it should have fired" means for each job type.
