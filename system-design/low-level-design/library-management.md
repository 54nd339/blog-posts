---
title: LLD - Library Management System
description: Members, book copies, and the rules between them. A title has many physical copies; a loan has a due date and a fine policy; a returned copy promotes the reservation queue. The design separates the abstract book from the shelved item and pushes each rule onto its owner.
date: 2026-06-14
draft: false
slug: /system-design/library-management
tags:
  - System Design
  - Low-Level Design
  - OOD
---

A library has *Clean Code* — the title — and it has five physical copies of it, three on the shelf and two on loan. A member borrows "a copy," picks one up, and returns it two weeks later; if they're late, a fine accrues; if someone reserved the title while all copies were out, the returned copy goes to them first.

The modelling move that makes this clean is separating **`Book`** (the abstract title and its metadata) from **`BookItem`** (one physical, barcoded copy with a location and a status). Loans, reservations, and fines then attach to copies and members, and each rule lives on the object that owns its data.

## Scope

- Catalogue search (title, author, subject).
- Multiple copies per title; issue, return, renew, reserve.
- Due dates; fines for overdue returns; per-member borrow limits.
- Roles: member and librarian.
- Out of scope: acquisitions, inter-branch transfer, e-books (mention them).

## Model

```java
class Book {                       // the title
    final String isbn;
    String title, author;
    Set<String> subjects;
    // no copies here directly; a catalogue maps isbn -> List<BookItem>
}

enum ItemStatus { AVAILABLE, LOANED, RESERVED, LOST }

class BookItem {                    // one physical copy
    final String barcode;
    final Book book;
    String location;               // shelf / rack
    ItemStatus status = ItemStatus.AVAILABLE;
}

class Member {
    final String id;
    String name;
    final List<Loan> activeLoans = new ArrayList<>();
    Money outstandingFine = Money.ZERO;
    int maxLoans() { return 5; }
    boolean canBorrow() { return activeLoans.size() < maxLoans()
                              && outstandingFine.isLessThan(FINE_BLOCK_THRESHOLD); }
}

class Loan {
    final BookItem item;
    final Member member;
    final LocalDate issuedOn;
    LocalDate dueOn;
    LocalDate returnedOn;          // null while active
    int renewals = 0;
}

class Reservation {
    final Book book;               // reserve the title, not a specific copy
    final Member member;
    final Instant placedOn;
    ReservationStatus status;      // WAITING, READY, FULFILLED, CANCELLED
}
```

The borrow rule (`canBorrow`) is on `Member` because it's about the member's state. The fine calculation is a strategy (below). The catalogue is a service that indexes `Book`s and holds the `isbn -> [BookItem]` map.

## Issuing and returning

```java
class LendingService {
    Loan issue(Member m, String barcode) {
        if (!m.canBorrow()) throw new BorrowNotAllowedException(m.id);
        BookItem item = catalogue.item(barcode);
        if (item.status != ItemStatus.AVAILABLE) throw new ItemUnavailableException(barcode);
        item.status = ItemStatus.LOANED;
        Loan loan = new Loan(item, m, today(), today().plusDays(LOAN_DAYS));
        m.activeLoans.add(loan);
        return loan;
    }

    Money returnItem(String barcode) {
        Loan loan = findActiveLoan(barcode);
        loan.returnedOn = today();
        loan.member.activeLoans.remove(loan);
        Money fine = finePolicy.fineFor(loan);          // strategy
        loan.member.outstandingFine = loan.member.outstandingFine.plus(fine);

        BookItem item = loan.item;
        Optional<Reservation> next = reservations.headOfQueue(item.book);
        if (next.isPresent()) {
            item.status = ItemStatus.RESERVED;
            next.get().status = ReservationStatus.READY;
            notify(next.get().member, item);            // observer
        } else {
            item.status = ItemStatus.AVAILABLE;
        }
        return fine;
    }
}
```

Returning a copy checks the reservation queue for that title: if someone's waiting, the copy is held (`RESERVED`) for them rather than going back to `AVAILABLE`.

## Fine policy as a strategy

```java
interface FinePolicy { Money fineFor(Loan loan); }

class PerDayFinePolicy implements FinePolicy {
    private final Money perDay; private final Money cap;
    public Money fineFor(Loan loan) {
        long over = ChronoUnit.DAYS.between(loan.dueOn, loan.returnedOn);
        if (over <= 0) return Money.ZERO;
        return Money.min(perDay.times(over), cap);
    }
}
```

Swap in a grace-period policy, a tiered policy, or a "no fines, just a block after N overdue" policy.

## Reservations

An ordered queue per title (`Map<Book, Deque<Reservation>>`). Placing a reservation is allowed only when no copy is currently available (otherwise just borrow it). Return promotes the head; a `READY` reservation has a pickup window, after which it's cancelled and the next in line is promoted (a small scheduled sweep).

## Concurrency

Two members trying to borrow the last available copy: guard `issue` per `BookItem` (set `AVAILABLE → LOANED` atomically — a `compareAndSet` on the status, or a lock per item). Two members reserving the same title: the reservation queue append is synchronized per title.

## Extensions

- **E-books** — a `DigitalItem` with concurrent-license limits instead of one-physical-copy; the loan still works, "return" is automatic at expiry.
- **Inter-branch** — `BookItem` gains a `branch`; a transfer is a status/location change with an in-transit state.
- **Membership tiers** — `maxLoans()` and `LOAN_DAYS` vary by tier; make them a `MembershipPlan`.
- **Notifications** — observers on due-soon, overdue, and reservation-ready events.

## The one idea to keep

Separate the title (`Book`, metadata, one per ISBN) from the physical copy (`BookItem`, barcoded, has a status). Loans and fines attach to copies and members; the borrow-eligibility rule sits on `Member`; the fine formula is a swappable `FinePolicy`. A return does one extra thing — check the title's reservation queue and hold the copy for the next person instead of re-shelving it.
