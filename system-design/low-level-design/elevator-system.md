---
title: LLD - Elevator System
description: A bank of elevators, hall calls and car calls, and the real question - which car serves a request to minimize wait time. Model each car as a state machine, make the dispatch rule a strategy, and drive it with a tick loop rather than a thread per car.
date: 2026-04-22
draft: false
slug: /system-design/elevator-system
tags:
  - System Design
  - Low-Level Design
  - OOD
---

Press the up button on floor 3 (a **hall call**). Inside the car, press 7 (a **car call**). With several elevators in the building, something has to decide which car takes the hall call, and in what order each car visits its pending floors so nobody waits forever. That decision — dispatch — is the interesting part; the mechanics are a state machine.

Following the [method](/citadel/system-design/object-oriented-design): the varying thing is the dispatch algorithm, so that's an interface. Each car's motion is a small state machine. A central controller routes calls.

## Scope

- `N` elevators, `M` floors.
- Hall calls (floor + direction) and car calls (destination floor).
- Doors, capacity, an idle policy.
- Out of scope: physics, weight sensors beyond a simple capacity flag, maintenance scheduling (mention it).

## Car as a state machine

```java
enum Direction { UP, DOWN, IDLE }
enum DoorState { OPEN, CLOSED }

class ElevatorCar {
    final int id;
    int currentFloor = 0;
    Direction direction = Direction.IDLE;
    DoorState door = DoorState.CLOSED;
    final int capacity;
    int load = 0;

    // targets split by travel direction so we sweep, not bounce
    final TreeSet<Integer> upTargets = new TreeSet<>();
    final NavigableSet<Integer> downTargets = new TreeSet<>().descendingSet();

    void addTarget(int floor) {
        if (floor > currentFloor) upTargets.add(floor);
        else if (floor < currentFloor) downTargets.add(floor);
        // == currentFloor: open doors this tick
    }

    void step() {                          // one tick of simulated time
        Integer next = nextTargetInDirection();
        if (next == null) { direction = Direction.IDLE; return; }
        if (next == currentFloor) { openDoors(); removeTarget(currentFloor); return; }
        direction = next > currentFloor ? Direction.UP : Direction.DOWN;
        currentFloor += (direction == Direction.UP ? 1 : -1);
    }
}
```

Splitting targets into `upTargets` / `downTargets` gives the **LOOK** algorithm: keep going in the current direction, serving every target on the way, then reverse when there's nothing further ahead. This is what real elevators do — it beats "closest target next," which makes a car oscillate and starves the far floors.

## Extension point: dispatch strategy

```java
interface DispatchStrategy {
    ElevatorCar selectCar(List<ElevatorCar> cars, HallCall call);
}

class NearestCarStrategy implements DispatchStrategy {
    public ElevatorCar selectCar(List<ElevatorCar> cars, HallCall call) {
        return cars.stream()
            .filter(c -> c.load < c.capacity)
            .min(Comparator.comparingInt(c -> cost(c, call)))
            .orElseThrow();
    }
    private int cost(ElevatorCar c, HallCall call) {
        int dist = Math.abs(c.currentFloor - call.floor);
        // cheap if the car is idle or already heading toward the call in the
        // same direction; expensive if it must finish the opposite direction first
        if (c.direction == Direction.IDLE) return dist;
        boolean sameWay = (call.floor > c.currentFloor) == (c.direction == Direction.UP);
        return sameWay && matchesRequestedDirection(c, call) ? dist : dist + 2 * PENALTY;
    }
}
```

Swap in `LeastLoadStrategy`, `ZonedStrategy` (each car owns a floor band), or `CollectiveControl`. The controller doesn't change.

## The controller

```java
class ElevatorController {
    final List<ElevatorCar> cars;
    final DispatchStrategy dispatcher;
    final Queue<HallCall> pendingHallCalls = new ConcurrentLinkedQueue<>();

    void hallCall(int floor, Direction dir) { pendingHallCalls.add(new HallCall(floor, dir)); }
    void carCall(int carId, int floor)      { car(carId).addTarget(floor); }

    void tick() {                          // called on a fixed timer
        HallCall call;
        while ((call = pendingHallCalls.poll()) != null) {
            ElevatorCar chosen = dispatcher.selectCar(cars, call);
            chosen.addTarget(call.floor);
        }
        for (ElevatorCar c : cars) c.step();
        notifyDisplays();
    }
}
```

## Tick loop vs thread-per-car

A thread per car with `sleep`s to model travel time is tempting but makes the system hard to test and reason about. A **tick loop** — one method that advances every car by one unit of simulated time — is deterministic, easy to unit-test ("after 5 ticks, car 2 is on floor 4 with doors open"), and matches how the interviewer will probe it. Real hardware is event-driven, but the model is cleaner as discrete steps.

## Observer for buttons and displays

Hall buttons and floor displays are `Observer`s: a button press produces a call event; a car arriving/leaving a floor pushes a display update. This decouples the UI from the control logic — a new display type just subscribes.

## Extensions the interviewer will ask for

- **Express / sky-lobby elevators** — a car with a restricted `servableFloors` set; the dispatcher filters on it.
- **VIP / priority call** — a `priority` flag on the call; the dispatcher jumps it and the car inserts it as the next target.
- **Fire / emergency mode** — a global state that overrides dispatch: all cars go to the ground floor and park with doors open.
- **Energy optimization** — an idle policy that parks cars at floors with historically high demand.

## The one idea to keep

Model each elevator as a state machine with its pending floors split by direction, so it sweeps (LOOK) instead of bouncing to the nearest button. Make dispatch — which car answers a hall call — a `DispatchStrategy` interface, scoring cars by a distance that's cheap when the car is idle or already heading the right way and expensive when it must reverse first. Drive the whole thing with a deterministic tick loop, not a thread per car, so it's testable.
