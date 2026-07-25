---
title: LLD - Splitwise (Expense Sharing)
description: Record who paid for what and how it splits, keep a running balance sheet of who owes whom, and simplify a tangle of debts into the fewest transactions. The split rule is a strategy; the simplification is a small greedy graph algorithm.
date: 2026-07-25
draft: false
slug: /system-design/splitwise
tags:
  - System Design
  - Low-Level Design
  - OOD
---

Four friends on a trip. One pays for dinner, another for the cab, a third for the hotel. At the end nobody wants to remember six separate "you owe me" conversations — they want one number each, and ideally the fewest possible payments to settle up. Splitwise is the app that tracks this: every expense updates a **balance sheet** of pairwise debts, and a "settle up" button collapses the web of IOUs into a minimal set of transfers.

Two design pieces: how an expense splits (equal / exact amounts / percentages — a strategy), and how to turn `A owes B ₹100, B owes C ₹100` into `A pays C ₹100` (a greedy algorithm on net balances).

## Scope

- Users and groups.
- Add an expense: who paid, total, and a split among participants.
- Per-user balances (overall and per other user).
- Settle up (record a payment) and simplify debts.
- Out of scope: multi-currency, receipts, auth — mention them.

## Model

```java
class User { final String id; String name; }

class Expense {
    final String id;
    final User paidBy;
    final Money amount;
    final List<Split> splits;          // one per participant, must sum to amount
}

// Split strategy
interface Split { User user(); Money owed(Money total); }

record EqualSplit(User user, int n) implements Split {
    public Money owed(Money total) { return total.dividedBy(n); }   // last one absorbs the remainder
}
record ExactSplit(User user, Money exact) implements Split {
    public Money owed(Money total) { return exact; }
}
record PercentSplit(User user, double pct) implements Split {
    public Money owed(Money total) { return total.times(pct / 100.0); }
}
```

Adding a "share-based" split (2 shares for one person, 1 each for the others) is a new `Split` record — the open-closed point.

## The balance sheet

Store net pairwise balances, not raw expenses, for O(1) "what do I owe X":

```java
class BalanceSheet {
    // balances.get(a).get(b) = how much a owes b  (negative ⇒ b owes a)
    private final Map<String, Map<String, Money>> balances = new HashMap<>();

    void applyExpense(Expense e) {
        Money total = e.amount;
        for (Split s : e.splits) {
            if (s.user().equals(e.paidBy)) continue;
            Money share = s.owed(total);
            adjust(s.user().id, e.paidBy.id, share);   // participant owes payer their share
        }
    }
    private void adjust(String debtor, String creditor, Money d) {
        balances.computeIfAbsent(debtor, k -> new HashMap<>())
                .merge(creditor, d, Money::plus);
        balances.computeIfAbsent(creditor, k -> new HashMap<>())
                .merge(debtor, d.negate(), Money::plus);
    }
    Money balanceBetween(String a, String b) {
        return balances.getOrDefault(a, Map.of()).getOrDefault(b, Money.ZERO);
    }
}
```

**Validation**: before applying, check the splits sum to the total (exactly — rounding is a real bug here). For an equal split of ₹100 among 3, shares are ₹33.33, ₹33.33, ₹33.34; assign the leftover cent deterministically (to the payer, or the first participant) so the books balance to zero.

A `settleUp(from, to, amount)` is just an `adjust(to, from, amount)` — a payment reduces what `from` owes `to`.

## Simplifying debts

After many expenses the pairwise graph is tangled. Reduce it to the minimum number of transfers:

```java
List<Transfer> simplify(BalanceSheet sheet) {
    // 1. net every person's position: sum of (owed to them) - (they owe)
    Map<String, Long> net = new HashMap<>();
    // ... fold the balance sheet into a single net figure per user (in minor units)

    // 2. two heaps: creditors (net > 0), debtors (net < 0)
    PriorityQueue<Bal> creditors = new PriorityQueue<>(comparingLong(b -> -b.amt));
    PriorityQueue<Bal> debtors   = new PriorityQueue<>(comparingLong(b -> b.amt));
    // fill from `net`

    List<Transfer> out = new ArrayList<>();
    while (!creditors.isEmpty() && !debtors.isEmpty()) {
        Bal c = creditors.poll(), d = debtors.poll();
        long x = Math.min(c.amt, -d.amt);
        out.add(new Transfer(d.id, c.id, x));     // debtor pays creditor x
        if (c.amt - x > 0)  creditors.add(new Bal(c.id, c.amt - x));
        if (-d.amt - x > 0) debtors.add(new Bal(d.id, -d.amt - x));
    }
    return out;
}
```

Net everyone to a single number, then repeatedly match the biggest creditor with the biggest debtor. This produces at most `n - 1` transfers. (Finding the *provably* minimum is NP-hard in general — it's a partition problem — but this greedy is what apps use and it's near-optimal in practice.)

## Concurrency

Two expenses added to the same group concurrently both mutate the balance sheet. Lock per group (`synchronized` on the group's balance sheet, or a per-group lock), or make `adjust` atomic per pair. The critical section is tiny.

## Extensions

- **Multi-currency** — `Money` carries a currency; conversions at expense time using a rate; balances kept per currency or normalized.
- **Recurring expenses** — a scheduler creates the expense on a cadence.
- **Comments / attachments / activity feed** — separate entities hanging off `Expense`.
- **Partial settlements** — already supported by `settleUp` with any amount.

## The one idea to keep

Splitwise stores *net pairwise balances*, not a list of expenses, so "what do I owe you" is a map lookup. The split rule (equal / exact / percent / shares) is a `Split` strategy, and the only place rounding can bite is making the shares sum exactly to the total — assign the leftover minor units deterministically. "Settle up" nets every person to one number and greedily matches biggest creditor to biggest debtor, yielding at most `n - 1` transfers.
