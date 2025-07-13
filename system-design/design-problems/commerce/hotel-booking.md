---
title: Designing a Hotel Reservation System
description: Hotel inventory is room-nights, so a booking has to decrement availability across every date in the stay, atomically, without overselling - except that hotels deliberately oversell. The data model is a per-date count, the search is availability plus geo, and the booking is a conditional range update.
date: 2025-07-13
draft: false
slug: /system-design/hotel-booking
tags:
  - System Design
  - Design Problems
---

Searching hotels is "places near a location, available for these dates, in this price range." Booking one is decrementing a count for every night of the stay at once. The unit of inventory is a **room-night**, and a three-night reservation touches three separate availability records that all have to succeed or all roll back.

Unlike [movie](/citadel/system-design/movie-ticket-booking) or [train](/citadel/system-design/train-booking) booking, hotels usually don't assign a *specific* room at reservation time — you book "a king room," and the front desk picks the physical room at check-in. That makes the concurrency simpler (decrement a counter, not lock a seat) and lets hotels do something the others can't: **deliberately oversell**.

## What it has to do

- Search by location + date range + guests + filters; rank results.
- Reserve a room type for a stay; modify; cancel.
- Never oversell beyond policy.
- Rate plans, taxes, dynamic pricing.
- For an aggregator (Booking.com, MakeMyTrip): sync inventory from many hotels / channel managers.

## Inventory model

Not "rooms," but **counts per date**:

`room_inventory( hotel_id, room_type_id, date ) → { total, booked }`

Availability for a stay `[check_in, check_out)` = for every date in that range, `booked < total` (times the number of rooms requested). A row per room-type per date; a 200-room-type hotel over a 2-year booking window is ~150k rows — trivial.

Partition by `hotel_id` (or `hotel_id, room_type_id`) so one hotel's booking traffic stays on one shard.

## Booking: a conditional range update

```sql
UPDATE room_inventory
SET booked = booked + :rooms
WHERE hotel_id = :h AND room_type_id = :rt
  AND date >= :check_in AND date < :check_out
  AND booked + :rooms <= total + :overbook_allowance;
```

If the affected row count equals the number of nights, all dates had room — commit and create the reservation. If it's fewer, some night was full — roll back and tell the user. One statement, atomic within the transaction, no row locks held across a payment.

There's no long "hold" like a seat map needs, because you're incrementing a number, not reserving a labelled thing — but you still create the reservation in `PENDING` and confirm on the payment webhook, releasing (decrementing) on failure.

## Overbooking

Hotels know some fraction of reservations no-show or cancel late, so they sell a few percent past capacity — the `overbook_allowance` above. If everyone shows up, the hotel **walks** the overflow guests (pays for a room at a nearby hotel plus compensation). The allowance is set per hotel per date from a forecast (day of week, season, event calendar, historical no-show rate). It's a revenue optimization with a bounded, budgeted downside — the opposite of the "never oversell" rule for assigned-seat systems.

## Search

Two filters combined:

- **Geo** — hotels within the map viewport or near a landmark ([proximity service](/citadel/system-design/proximity-service): geohash/S2 index).
- **Availability** — of those hotels, which have a room type free for every night of the requested range.

Checking availability live for thousands of candidate hotels on every search is expensive, so keep a **precomputed availability cache** — per `(hotel, room_type, date)`, "available or not" and the current price — refreshed on every booking/cancellation and on rate updates. Search reads the cache; only the final booking hits the authoritative inventory table. A stale cache occasionally shows a room that just sold out; the booking then fails cleanly and the user retries.

Ranking blends distance, price, rating, and (for aggregators) commission and conversion history.

## Rate plans and pricing

A room type has multiple **rate plans** (flexible vs non-refundable, with/without breakfast), each a price rule. Prices are per date and dynamic — a pricing service adjusts by occupancy, lead time, competitor rates, and demand signals. The search cache stores the current computed price alongside availability.

## Aggregator inventory sync

An aggregator doesn't own the rooms. Hotels (or channel managers like SiteMinder) push availability and rates via API or feed. The aggregator holds a local mirror and books against the hotel's system, which can reject if the mirror was stale — so bookings are a two-phase confirm (reserve locally as `PENDING`, confirm with the supplier, then confirm to the user), a [saga](/citadel/interview/distributed-transactions) with a compensating release if the supplier declines.

## The one idea to keep

Hotel inventory is a count per room-type per date, so booking a stay is one conditional `UPDATE` that increments `booked` across the whole date range and checks the affected-row count for all-or-nothing. No seat-style hold is needed because you're bumping a number, not claiming a labelled object — which also lets hotels intentionally oversell by a forecast-driven allowance and absorb the rare walk. Search runs off a precomputed availability-and-price cache; only the commit touches the real inventory table.
