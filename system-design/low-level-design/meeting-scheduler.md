---
title: LLD - Meeting Room Scheduler
description: Book a room for an interval without overlap, and find a slot that is free for everyone in a group. The overlap check is a sorted-interval search; the common-slot search is a merge of busy intervals. Recurrence and time zones are the extensions that catch people out.
date: 2026-05-16
draft: false
slug: /system-design/meeting-scheduler
tags:
  - System Design
  - Low-Level Design
  - OOD
---

Two operations carry this design. **Book** a room for `[start, end)` — allowed only if nothing overlaps. **Find** a time when a room *and* every required attendee are free for a given duration. The first is an interval-overlap check; the second is a merge of busy intervals across several calendars. Everything else — recurrence, notifications, priority — sits on top.

## Scope

- Rooms with a capacity.
- `book(room, interval, organizer, attendees)`; `cancel`; `list`.
- `findSlot(attendees, duration, window)` → available `(room, start)`.
- Recurring meetings.
- Out of scope: video links, catering, floor plans (mention them).

## Model

```java
record Interval(Instant start, Instant end) {
    boolean overlaps(Interval o) { return start.isBefore(o.end) && o.start.isBefore(end); }
    Interval(Instant s, Instant e) { if (!s.isBefore(e)) throw new IllegalArgumentException(); }
}

class Room { final String id; int capacity; }

record Meeting(String id, Room room, Interval interval,
               User organizer, Set<User> attendees, RecurrenceRule recurrence) {}

class Calendar {                       // one per room, and one per user
    // busy intervals kept sorted by start
    private final NavigableMap<Instant, Interval> busy = new TreeMap<>();

    boolean isFree(Interval slot) {
        var lower = busy.floorEntry(slot.start());          // meeting starting at or before
        if (lower != null && lower.getValue().overlaps(slot)) return false;
        var higher = busy.ceilingEntry(slot.start());       // next meeting
        return higher == null || !higher.getValue().overlaps(slot);
    }

    void add(Interval i)    { busy.put(i.start(), i); }
    void remove(Interval i) { busy.remove(i.start()); }
}
```

Keeping intervals in a `TreeMap` keyed by start time makes the overlap check O(log n): only the meeting just before and just after `slot.start()` can overlap it, so you check two entries, not all of them. For many rooms and a heavy schedule, an **interval tree** answers "any overlap with `[s,e)`?" in O(log n + k) across a whole set.

## Booking

```java
class SchedulerService {
    private final Map<String, Calendar> roomCalendars;
    private final Map<String, Calendar> userCalendars;

    synchronized Meeting book(Room room, Interval when, User organizer, Set<User> attendees) {
        Calendar rc = roomCalendars.get(room.id);
        if (!rc.isFree(when)) throw new RoomBusyException(room.id, when);
        if (attendees.size() + 1 > room.capacity) throw new CapacityExceededException();

        Meeting m = new Meeting(newId(), room, when, organizer, attendees, null);
        rc.add(when);
        for (User u : union(attendees, organizer)) userCalendars.get(u.id()).add(when);
        return m;
    }
}
```

Booking updates the room's calendar and every attendee's calendar, so a later `findSlot` sees them as busy.

## Finding a common slot

Free for a room and everyone means free in the *union* of all their busy intervals:

```java
Optional<Slot> findSlot(Set<User> attendees, Duration duration, Interval window) {
    for (Room room : rooms) {
        if (room.capacity < attendees.size()) continue;
        List<Interval> busy = new ArrayList<>();
        busy.addAll(roomCalendars.get(room.id).within(window));
        for (User u : attendees) busy.addAll(userCalendars.get(u.id()).within(window));

        // merge overlapping busy intervals, then scan the gaps
        busy.sort(comparing(Interval::start));
        Instant cursor = window.start();
        for (Interval b : merge(busy)) {
            if (Duration.between(cursor, b.start()).compareTo(duration) >= 0)
                return Optional.of(new Slot(room, new Interval(cursor, cursor.plus(duration))));
            cursor = maxOf(cursor, b.end());
        }
        if (Duration.between(cursor, window.end()).compareTo(duration) >= 0)
            return Optional.of(new Slot(room, new Interval(cursor, cursor.plus(duration))));
    }
    return Optional.empty();
}
```

Collect everyone's busy intervals in the search window, merge the overlapping ones into a set of blocked ranges, then walk the gaps between them for one at least `duration` long. Restrict `window` to working hours so it doesn't propose 3 a.m.

## Recurrence

A `RecurrenceRule` (frequency, interval, end condition, by-day) is stored on the meeting; **don't** materialize every occurrence up front. Expand it lazily for the requested window when checking conflicts or rendering. Store **exceptions** (a single occurrence moved or cancelled) as overrides keyed by the original occurrence's date. A recurring booking that conflicts on *one* future date is a real edge case — book the series and flag the clashing occurrence, or reject; state which.

## Time zones

Store instants in UTC. Attach the organizer's zone to the meeting for display, and interpret "working hours" and "weekly on Tuesday" in each attendee's local zone (a meeting at 9 a.m. New York is 2 p.m. London — both may be inside working hours, or not). DST means a "weekly 09:00 local" meeting shifts its UTC instant twice a year; compute occurrences from local wall-clock rules, then convert.

## Concurrency

Two organizers booking the same room and slot: guard the room's calendar. A lock per room (or a conditional insert — "add this interval only if `isFree`") is enough; different rooms book in parallel. Notify on conflict rather than silently failing.

## Extensions

- **Buffer time** — pad each meeting by N minutes so back-to-backs have a gap; `isFree` checks the padded interval.
- **Priority / bumping** — a high-priority meeting can displace a lower one, notifying the bumped organizer.
- **Equipment** — treat a projector like a room (its own calendar); `findSlot` must satisfy all resources.
- **Waitlist** — if no slot, queue the request and notify when a cancellation opens one.

## The one idea to keep

Booking is an interval-overlap check — keep each room's and each user's meetings in a structure sorted by start time so only the neighbouring two intervals can overlap a candidate slot. Finding a common time is a merge: gather everyone's busy intervals in the window, merge the overlaps, and scan the gaps for one long enough. Store instants in UTC but evaluate recurrence and working hours in each attendee's local zone, because DST will otherwise drift your weekly meeting.
