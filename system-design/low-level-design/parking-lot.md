---
title: LLD - Parking Lot
description: The canonical machine-coding problem. Multiple floors and spot sizes, a ticket with a fee computed at exit, and pluggable strategies for which spot to assign and how to price - built so that "add EV charging" is a new class, not a rewrite.
date: 2026-03-18
draft: false
slug: /system-design/parking-lot
tags:
  - System Design
  - Low-Level Design
  - OOD
---

Park a car, get a ticket, leave, pay by duration. Multiple floors, spots of different sizes, and vehicles that fit some spots and not others. It's the parking lot because it exercises exactly the things an LLD round grades: a small domain model, two obvious extension points (which spot, what price), and one real concurrency question (two cars, one spot).

Following the [method](/citadel/system-design/object-oriented-design): scope it, find the nouns, put each rule on its owner, and make the varying parts interfaces.

## Scope

- Multiple floors; spots typed `MOTORCYCLE`, `COMPACT`, `LARGE`.
- Operations: `park(vehicle) → Ticket`, `unpark(ticket) → Fee`, plus `availability()`.
- Fee by parked duration, with the pricing rule swappable.
- Out of scope: reservations, monthly passes, multiple sites — mention them as extensions.

## The model

```java
enum SpotType { MOTORCYCLE, COMPACT, LARGE }
enum VehicleType { MOTORCYCLE, CAR, TRUCK }

abstract class Vehicle {
    final String plate;
    abstract VehicleType type();
    boolean fitsIn(SpotType s) {            // the rule lives on the vehicle
        return switch (type()) {
            case MOTORCYCLE -> true;                       // fits anywhere
            case CAR        -> s != SpotType.MOTORCYCLE;   // compact or large
            case TRUCK      -> s == SpotType.LARGE;
        };
    }
}

class ParkingSpot {
    final String id; final SpotType type;
    private Vehicle current;                // null = free
    synchronized boolean tryPark(Vehicle v) {
        if (current == null && v.fitsIn(type)) { current = v; return true; }
        return false;
    }
    synchronized void free() { current = null; }
    boolean isFree() { return current == null; }
}

class Ticket {
    final String id; final ParkingSpot spot; final Vehicle vehicle;
    final Instant entryTime;
    Instant exitTime;                       // set at unpark
}
```

`Floor` holds a `List<ParkingSpot>` plus, for O(1) allocation, a `Map<SpotType, Deque<ParkingSpot>>` of free spots per type.

## Extension point 1: spot assignment

```java
interface SpotAssignmentStrategy {
    Optional<ParkingSpot> assign(List<Floor> floors, Vehicle v);
}

class NearestFirstStrategy implements SpotAssignmentStrategy {
    public Optional<ParkingSpot> assign(List<Floor> floors, Vehicle v) {
        for (Floor f : floors)                       // floor 0 first
            for (SpotType t : SpotType.values())     // smallest fitting type first
                if (v.fitsIn(t)) {
                    ParkingSpot s = f.pollFree(t);   // atomic remove-from-free
                    if (s != null) return Optional.of(s);
                }
        return Optional.empty();
    }
}
```

Swap in `RandomStrategy`, `HighestFloorFirst`, or a strategy that spreads load — no change to `ParkingLot`.

## Extension point 2: pricing

```java
interface PricingStrategy {
    Money fee(Ticket t);
}

class HourlyPricing implements PricingStrategy {
    private final Map<SpotType, Money> ratePerHour;
    public Money fee(Ticket t) {
        long hours = Math.ceilDiv(Duration.between(t.entryTime, t.exitTime).toMinutes(), 60);
        return ratePerHour.get(t.spot.type).times(hours);
    }
}
```

`FlatPricing`, `TieredPricing` (first hour free, then rising), `DayNightPricing` — all new classes. `Money` is an immutable value object (amount + currency), never a bare `double`.

## The coordinator

```java
class ParkingLot {                         // Singleton for the process
    private final List<Floor> floors;
    private final SpotAssignmentStrategy assigner;
    private final PricingStrategy pricing;
    private final Map<String, Ticket> active = new ConcurrentHashMap<>();

    Ticket park(Vehicle v) {
        ParkingSpot spot = assigner.assign(floors, v)
            .orElseThrow(() -> new LotFullException(v.type()));
        Ticket t = new Ticket(newId(), spot, v, Instant.now());
        active.put(t.id, t);
        return t;
    }

    Money unpark(String ticketId) {
        Ticket t = active.remove(ticketId);
        if (t == null) throw new UnknownTicketException(ticketId);
        t.exitTime = Instant.now();
        t.spot.free();
        return pricing.fee(t);
    }
}
```

`ParkingLot` only coordinates. It doesn't know the fitting rules (on `Vehicle`), the allocation order (in the strategy), or the fee formula (in the strategy) — single responsibility.

## Concurrency

The race is two threads calling `park` and both being handed the last COMPACT spot. It's resolved by making "take a free spot" atomic: `Floor.pollFree(type)` removes from the free-deque under the floor's lock (or uses a `ConcurrentLinkedDeque` and relies on `poll()` being atomic), and `ParkingSpot.tryPark` is `synchronized` as a second guard. One lock per floor, not one global lock, so different floors park in parallel.

## Extensions the interviewer will ask for

- **EV charging** — an `ElectricSpot` (or a `Chargeable` capability interface on some spots) and a vehicle attribute; the assigner prefers a charging spot for an EV. New classes, no edits.
- **Reservations** — a `Reservation` holding a spot with a TTL; the assigner skips reserved spots.
- **Multiple entrances / exits** — `EntryGate` / `ExitGate` objects; `park`/`unpark` move onto them.
- **Payments** — a `PaymentProcessor` interface; `unpark` returns a `Fee`, `pay(fee, method)` settles it and closes the ticket.

## The one idea to keep

Model the parking lot so the two things that vary — which spot a vehicle gets, and what it's charged — are `SpotAssignmentStrategy` and `PricingStrategy` interfaces injected into a thin `ParkingLot` coordinator. Put the "does this vehicle fit this spot" rule on `Vehicle`, keep money as an immutable value object, and make "claim a free spot" atomic per floor. Then every follow-up requirement — EV charging, reservations, tiered rates — lands as a new class.
