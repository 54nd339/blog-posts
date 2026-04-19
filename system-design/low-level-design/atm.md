---
title: LLD - ATM
description: A state machine wrapped around a hardware abstraction. Card in, PIN, pick a transaction, dispense cash - and if any step fails, roll back cleanly so the account is not debited for money that never came out. The transactions themselves are a small class hierarchy.
date: 2026-04-19
draft: false
slug: /system-design/atm
tags:
  - System Design
  - Low-Level Design
  - OOD
---

An ATM walks every user through the same sequence: insert card, enter PIN, choose a transaction, get cash or a balance, take the card. The behaviour at each step depends on where you are — a PIN entry means nothing before a card is inserted — so it's a [State pattern](/citadel/system-design/vending-machine) problem like the vending machine, with two extra concerns: real hardware (card reader, cash dispenser, printer) behind interfaces, and a withdrawal that must not debit the account if the cash dispense fails.

## Scope

- Authenticate (card + PIN), then: balance inquiry, withdraw, deposit, mini-statement.
- Hardware: card reader, cash dispenser (with denomination bins), receipt printer, keypad, screen.
- Talk to a bank backend for auth and account operations.
- Out of scope: cardless withdrawal, cheque imaging (mention them).

## States

```java
interface AtmState {
    void insertCard(Card card);
    void enterPin(String pin);
    void selectTransaction(TxnType type, Map<String,Object> params);
    void cancel();
}
```

- **`IdleState`** — waiting for a card. `insertCard` reads the card, moves to `CardInsertedState`. Everything else is a no-op.
- **`CardInsertedState`** — has a card, needs a PIN. `enterPin` calls the bank to verify; success → `AuthenticatedState`; failure increments a retry counter and, past the limit, **captures the card** and returns to idle.
- **`AuthenticatedState`** — `selectTransaction` builds the right `Transaction` and moves to `TransactionState`.
- **`TransactionState`** — executes the transaction, prints a receipt, then ejects the card and returns to idle. `cancel` from most states ejects the card and resets.

A **session timeout** (a timer in `CardInsertedState`/`AuthenticatedState`) ejects the card if the user walks away.

```java
class ATM {
    private AtmState state;
    private final AtmState idle, cardInserted, authenticated, transaction;
    final CardReader cardReader;
    final CashDispenser dispenser;
    final ReceiptPrinter printer;
    final Screen screen;
    final BankService bank;          // remote

    Card currentCard;
    Account currentAccount;
    // public API forwards to state
}
```

## The withdrawal, and its rollback

This is the part that isn't like a vending machine. A withdrawal has steps that can each fail:

```java
class WithdrawTransaction extends Transaction {
    void execute(ATM atm, Money requested) {
        // 1. checks that don't touch money
        if (requested.compareTo(atm.dailyLimitRemaining(atm.currentAccount)) > 0)
            throw new LimitExceededException();
        if (!atm.dispenser.canDispense(requested))          // enough cash, right denominations?
            throw new InsufficientCashInAtmException();

        // 2. reserve/hold on the bank side (or a two-phase: hold -> confirm)
        String holdId = atm.bank.hold(atm.currentAccount, requested);
        try {
            // 3. physically dispense
            List<Note> notes = atm.dispenser.dispense(requested);   // may throw HardwareException
            // 4. confirm the debit only after cash is out
            atm.bank.confirm(holdId);
            atm.printer.print(receiptFor(atm.currentAccount, requested, notes));
        } catch (HardwareException e) {
            atm.bank.release(holdId);       // roll back: no cash, no debit
            atm.screen.show("Unable to dispense. Your account was not charged.");
            throw e;
        }
    }
}
```

The ordering is the rule: **check everything, hold the funds, dispense the cash, then confirm the debit**. If the dispenser jams after the hold but before confirm, `release` undoes the hold and the customer isn't charged. If the machine loses power between dispense and confirm, a reconciliation job at the bank matches ATM dispense logs against holds and settles the difference — the physical cash count is the tiebreaker.

`selectTransaction` on `WithdrawTransaction` failing leaves the user in `AuthenticatedState` to try again or cancel.

## Cash dispensing

`CashDispenser` holds a count per denomination bin. `canDispense(amount)` and `dispense(amount)` do a greedy largest-note-first make-up of the amount and fail if they can't hit it exactly with notes on hand (same as the [vending machine](/citadel/system-design/vending-machine)'s change logic). Servicing mode refills the bins and resets the counts.

## Transaction hierarchy

`BalanceInquiryTransaction`, `DepositTransaction`, `WithdrawTransaction`, `MiniStatementTransaction` — each subclass of `Transaction` with its own `execute`. A `TransactionFactory` builds the right one from the `TxnType`. Balance inquiry and mini-statement are pure reads; deposit credits after the envelope/notes are accepted (and often holds the funds until verified).

## Concurrency

An ATM is single-session — one card at a time — so the state machine is single-threaded. The **bank backend** is the shared, concurrent part: the account debit must be atomic there (a transaction, or the hold/confirm protocol above), and the `hold` carries an idempotency key (ATM ID + local sequence) so a retried request doesn't double-hold.

## Extensions

- **Cardless (OTP)** — a new entry state that authenticates via a code from the mobile app instead of a card.
- **Multiple accounts per card** — `AuthenticatedState` prompts for which account; `currentAccount` is chosen per transaction.
- **Multi-currency dispenser** — bins per currency; `canDispense` per currency.
- **Fraud checks** — velocity limits, unusual-location flags, enforced by `BankService`.

## The one idea to keep

An ATM is a State machine (`Idle → CardInserted → Authenticated → Transaction`) over hardware interfaces, and the one non-obvious rule is the withdrawal ordering: run all checks, place a hold on the funds, dispense the physical cash, and only then confirm the debit — so a dispenser failure releases the hold and the customer is never charged for cash they didn't get. Reconciliation against dispense logs handles the power-loss-mid-transaction case.
