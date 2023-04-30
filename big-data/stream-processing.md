---
title: Stream Processing - Windows, Watermarks, and Exactly-Once
description: Batch jobs run on a bounded dataset that sits still; stream jobs run forever on data that keeps arriving, often out of order. Event time versus processing time, windows that close on a watermark, and checkpointed state are what let you compute a correct "last five minutes" over an infinite input.
date: 2023-04-30
draft: false
slug: /big-data/stream-processing
tags:
  - Big Data
  - Distributed Systems
  - Data Engineering
---

A batch job reads yesterday's logs, computes a report, and exits. The input is bounded, it doesn't change while you read it, and if the job crashes you just rerun it. A stream job computes "clicks per minute" continuously, forever, over events that are still arriving — some of them late, some out of order, from a phone that was in a tunnel. There's no "rerun from scratch," and "the last minute" is a moving target.

The concepts that make streaming correct rather than approximately-right are **event time**, **windowing**, and **watermarks**, plus checkpointed **state** for fault tolerance. This post is those, and the architectural choice they inform.

## Two notions of time

Every event has two timestamps:

- **Event time** — when the thing actually happened (stamped on the phone when the user tapped).
- **Processing time** — when your system got around to handling it.

They diverge, sometimes a lot: the tunnel phone's events have an event time from five minutes ago but a processing time of now. If you bucket "clicks per minute" by *processing* time, that tunnel's worth of clicks all land in the current minute — wrong. You almost always want **event-time** semantics, which means the system has to handle events arriving out of order and after their window "should" have closed.

## Windows

Since the stream is infinite, aggregation happens over **windows** — finite slices:

- **Tumbling** — fixed size, non-overlapping (`[12:00, 12:01)`, `[12:01, 12:02)`). Every event in exactly one window. "Clicks per minute."
- **Sliding** — fixed size, overlapping by a step (5-minute window every 1 minute). Every event in several windows. "5-minute moving average, updated each minute."
- **Session** — dynamically sized, defined by a gap of inactivity: events less than N minutes apart are one session; a gap ends it. "A user's browsing session."
- **Global** — one window for all time, only useful with custom triggers.

Each incoming event is assigned to its window(s) by its event time. The window's aggregate updates as events arrive.

## Watermarks: when is a window done

If events can arrive late, when do you emit the result for `[12:00, 12:01)`? Wait forever and you never produce output; close at 12:01 processing time and you drop every late event.

A **watermark** is the system's assertion: "I believe I have now seen every event with event time ≤ T." It's an estimate, usually `max_event_time_seen − allowed_lateness`. When the watermark passes a window's end, the window **fires** — emits its result. Events arriving after that are **late**:

- **Drop** them (simple, lossy).
- **Allowed lateness** — keep the window's state around a bit longer and emit an *updated* result when a late event lands.
- **Side output** — route late events to a separate stream for reconciliation.

Watermarks trade **latency against completeness**: a conservative watermark (large lateness allowance) waits longer so results are more complete; an aggressive one emits sooner but revises more often. There's no free lunch — out-of-order data forces the choice.

## Triggers and accumulation

- **Trigger** — *when* to emit: on watermark (the default), early (every N seconds, for a running estimate), late (on each late event).
- **Accumulation mode** — does a re-fire *replace* the previous result (accumulating) or emit a *delta* (discarding)? Matters for whatever consumes the output.

The Google Dataflow model (`what / where / when / how`) generalises all of this: *what* you compute (the aggregation), *where* in event time (windows), *when* results are emitted (triggers + watermark), *how* refinements relate (accumulation).

## State and fault tolerance

A windowed aggregation *is* state — partial sums, session buffers — held in the operators. A crash must not lose it or double-count it.

The mechanism is **distributed snapshots** (the Chandy–Lamport algorithm, as implemented by Flink): periodically, a special **barrier** marker flows through the dataflow; when an operator receives barriers on all its inputs, it snapshots its state to durable storage; when the barrier reaches the sinks, that snapshot is a consistent global checkpoint. On failure, every operator rewinds to the last checkpoint and the source replays from the offset recorded there.

That gives **exactly-once state**: the internal aggregates are correct despite replays. For **exactly-once output**, the sink must also be transactional — either idempotent writes (keyed upserts) or a two-phase commit that ties the external write to the checkpoint. Without that, output is at-least-once and downstream must dedupe.

Delivery semantics, then:

- **At-most-once** — no replay; fast, lossy.
- **At-least-once** — replay on failure; no lost data, but duplicates.
- **Exactly-once** — checkpointed state + transactional sink; correct, with a small latency cost per checkpoint.

## Engines

- **Kafka Streams** — a library, not a cluster; state in local RocksDB backed by Kafka changelog topics; exactly-once via Kafka transactions. Good when your data already lives in Kafka and you want no extra infrastructure.
- **Apache Flink** — a true streaming engine with sophisticated event-time, watermark, and state support; the reference implementation of the Dataflow model.
- **Spark Structured Streaming** — micro-batch (and a lower-latency continuous mode); reuses the Spark batch engine, so the same code runs batch or stream.
- **Apache Beam** — a portable API implementing the Dataflow model, running on Flink, Spark, or Google Dataflow.

## Lambda vs Kappa

- **Lambda architecture** — run a **batch** layer (slow, exact, reprocesses all history) and a **speed** layer (fast, approximate, streaming) in parallel, and merge their outputs at query time. Robust but you maintain the same logic twice, in two systems.
- **Kappa architecture** — one streaming layer only. To "reprocess," replay the input log from the beginning through a new version of the job. Simpler, and viable now that streaming engines are exactly-once and logs (Kafka) retain history. The trend is toward Kappa.

The **stream–table duality** underlies both: a table is the current state of a stream of changes; a stream is the changelog of a table. [Change data capture](/citadel/tech/kafka) turns a database's writes into a stream; a windowed aggregation turns a stream back into a table.

## The one idea to keep

Streaming is aggregation over an infinite, out-of-order input, so it's organised around event time: assign events to windows by when they happened, and fire a window when the **watermark** — the system's estimate of "seen everything up to here" — passes its end, choosing how long to wait for stragglers. Checkpointed distributed snapshots give exactly-once state on replay; a transactional sink extends that to exactly-once output. With those in place you can run a single streaming layer (Kappa) and reprocess by replaying the log.
