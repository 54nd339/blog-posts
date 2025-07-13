---
title: Designing a Train Ticket Booking System (IRCTC)
description: A train berth is not one seat - it is a seat for each leg of the journey, so availability is per segment and a booking from Delhi to Agra frees that berth again from Agra onward. Add quotas, a waitlist that promotes on cancellation, and the Tatkal stampede, and it is a harder problem than movie booking.
date: 2025-07-13
draft: false
slug: /system-design/train-booking
tags:
  - System Design
  - Design Problems
---

Movie booking has a fixed set of seats and one time. A train adds a dimension: the journey has many stations, and a berth is bookable *per segment*. If someone books berth B1 from Delhi to Agra, that berth is still free from Agra to Mumbai — and the system should sell it to another passenger for that stretch. Availability isn't one number per train; it's one number per berth per leg.

On top of that: reservation **quotas** (general, Tatkal, ladies), a **waitlist** with RAC (reservation against cancellation) that promotes people as others cancel, and the daily **Tatkal window** where a few thousand seats meet a few hundred thousand people at 10:00 sharp. That last part makes IRCTC as much a load problem as a data problem.

## What it has to do

- Search trains between two stations on a date; show class-wise availability per quota.
- Book, generating a PNR; support waitlist and RAC.
- Cancel, with refund rules, and promote the waitlist.
- Handle the Tatkal spike.

## The segment model

Number the stations on a train's route `0, 1, 2, …, k`. A journey from station `i` to station `j` occupies legs `i, i+1, …, j-1`.

Represent each berth's occupancy as a bitmap over the legs (or a small integer per leg = count of berths free). A berth is bookable for journey `(i, j)` iff it's free on **every** leg in `[i, j)`. Booking `(i, j)` marks those legs occupied; cancelling clears them; a booking `(j, m)` on the same berth is fine because the legs don't overlap.

"How many berths are available Delhi→Agra in Sleeper on train 12345 on 14 March" = count berths free on all legs of that sub-route — precomputable and cacheable per `(train, date, class, from, to)`.

Availability is stored per `(train, date, class)`; partition by that key so one popular train's booking traffic is localized.

## Quotas

Each `(train, date, class)` has multiple inventory pools: **General**, **Tatkal** (opens the day before, priced higher), **Ladies**, **Premium Tatkal**, plus route-specific quotas. They're separate counters over (usually) disjoint berth ranges, so a Tatkal booking never touches the general pool.

## Booking under contention

Same core as [movie booking](/citadel/system-design/movie-ticket-booking): a conditional update that claims specific berths on specific legs, all-or-nothing, with a short payment hold. The difference is the availability check spans legs, so the claim is "these berths are free on all of legs `[i, j)`" — a single `UPDATE ... WHERE` over the leg rows, checking the affected count.

Serializing per `(train, date, class)` partition is acceptable because a train has a bounded berth count; a queue of booking requests drained by one allocator per partition is a valid design and sidesteps subtle multi-leg race bugs.

## Waitlist and RAC

When the pool is exhausted:

- **RAC** — the passenger gets a confirmed *seat* (two share a berth) and is first in line for a full berth. An ordered queue.
- **Waitlist (WL)** — no seat yet; position `N` in an ordered queue.

On a **cancellation**, a background reconciler:

1. frees the berth's legs,
2. promotes the head of the RAC queue to a full berth,
3. promotes the head of the WL queue to RAC,
4. updates every affected PNR's status.

This runs continuously and again at **chart preparation** (a few hours before departure), after which the final status is fixed and further cancellations refund without promotion. Confirmation *probability* shown at booking time is a model over historical cancellation rates for that train/class/quota.

## PNR service

A booking produces a **PNR** — a record with the passengers, berths, and current status. PNR *status checks* are enormously read-heavy (people refresh obsessively), so the PNR store is fronted by a cache and even a CDN; the write path (booking, promotion) invalidates the relevant PNR.

## The Tatkal spike

At 10:00, requests jump by orders of magnitude for ~10 minutes against a tiny inventory. Defences:

- **Virtual waiting room** — admit users to the booking page at a controlled rate; everyone else sees a queue with a position.
- **CAPTCHA / bot detection** — most of the load is scripts; slowing them helps real users.
- **Aggressive per-user rate limits** and one-booking-in-flight per account.
- **Pre-warm** the `(train, date, Tatkal)` partitions and their caches before the window.
- **Fail fast** — once a quota is empty, reject immediately from cache rather than queuing doomed requests.

## Payment

Hold berths → `PENDING_PAYMENT` PNR → gateway → confirm on webhook → `CONFIRMED`/`RAC`/`WL`. Release on failure. Idempotency key = PNR. Refunds on cancellation follow a time-before-departure slab.

## The one idea to keep

A train berth is inventory *per leg*, so availability is "free on every leg of the requested sub-route" and a berth can be sold twice for non-overlapping stretches — model occupancy as a per-leg bitmap and claim with one conditional multi-leg update. The waitlist/RAC queues are ordered lists that a reconciler promotes on every cancellation and freezes at chart preparation. And Tatkal is a load-shedding problem: a virtual queue, hard per-user limits, and rejecting from cache the instant a quota empties.
