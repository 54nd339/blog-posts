---
title: LLD - Vending Machine
description: The textbook State pattern. The machine behaves differently to the same coin-insert or button-press depending on whether it is idle, holding money, or dispensing - so each mode is a class, and adding "card payment" or "cancel mid-purchase" is a new transition, not a bigger switch.
date: 2026-03-26
draft: false
slug: /system-design/vending-machine
tags:
  - System Design
  - Low-Level Design
  - OOD
---

A vending machine reacts to a handful of inputs — select a product, insert a coin, press dispense, press cancel — but the *right* reaction depends on where it is in a transaction. Inserting a coin when idle should say "pick something first." Inserting a coin after a selection should add to the balance. Pressing dispense with enough money should vend; without enough, it should refuse.

Handle that with a single method full of `if (state == ...)` and every new feature makes the method worse. The **State pattern** puts each mode in its own class that knows how to handle every input, and transitions are just "become a different state." This is the canonical problem for it.

## Scope

- Select product, insert coins/notes, dispense with change, cancel with refund.
- Reject if out of stock or insufficient funds; reject if exact change can't be made.
- Restock and coin-refill (admin).
- Single user at a time (a physical machine).

## The state interface

```java
interface State {
    void selectProduct(String code);
    void insertCoin(Coin coin);
    void pressDispense();
    void pressCancel();
}
```

Every state implements all four. Most states make most inputs a no-op with a message; each state does the *one or two* things that make sense for it.

```java
class IdleState implements State {
    private final VendingMachine m;
    public void selectProduct(String code) {
        Product p = m.inventory().get(code);
        if (p == null || p.count == 0) { m.display("unavailable"); return; }
        m.setSelected(p);
        m.setState(m.itemSelectedState());
    }
    public void insertCoin(Coin c)  { m.display("select a product first"); m.returnCoin(c); }
    public void pressDispense()     { m.display("select a product first"); }
    public void pressCancel()       { /* nothing to cancel */ }
}

class HasMoneyState implements State {
    private final VendingMachine m;
    public void insertCoin(Coin c) {
        m.addBalance(c.value);
        m.display("balance: " + m.balance());
    }
    public void pressDispense() {
        Product p = m.selected();
        if (m.balance() < p.price) { m.display("need " + (p.price - m.balance()) + " more"); return; }
        if (!m.coinBank().canMakeChange(m.balance() - p.price)) {
            m.display("no change available"); m.refund(); m.setState(m.idleState()); return;
        }
        m.setState(m.dispenseState());
        m.state().pressDispense();      // delegate the actual vend
    }
    public void pressCancel() { m.refund(); m.setState(m.idleState()); }
    public void selectProduct(String code) { m.display("finish or cancel current purchase"); }
}

class DispenseState implements State {
    private final VendingMachine m;
    public void pressDispense() {
        Product p = m.selected();
        m.coinBank().deposit(m.paidCoins());
        int change = m.balance() - p.price;
        m.coinBank().dispense(change);          // greedy make-change
        p.count--;
        m.dropProduct(p);
        m.reset();
        m.setState(m.idleState());
    }
    // every other input: ignored, machine is busy
}
```

`ItemSelectedState` (product chosen, no money yet) mostly forwards `insertCoin` into `HasMoneyState`.

## The context

```java
class VendingMachine {
    private State state;
    private final State idle, itemSelected, hasMoney, dispense;
    private final Inventory inventory;
    private final CoinBank coinBank;
    private Product selected;
    private int balance;

    VendingMachine(Inventory inv, CoinBank bank) {
        this.inventory = inv; this.coinBank = bank;
        idle = new IdleState(this); itemSelected = new ItemSelectedState(this);
        hasMoney = new HasMoneyState(this); dispense = new DispenseState(this);
        state = idle;
    }
    // public API just forwards to the current state
    void selectProduct(String c) { state.selectProduct(c); }
    void insertCoin(Coin c)      { state.insertCoin(c); }
    void pressDispense()         { state.pressDispense(); }
    void pressCancel()           { state.pressCancel(); }
}
```

The public methods are one-liners. All the branching that would have been a giant `switch` is now distributed across the state classes, each small and focused.

## Making change

`CoinBank` holds a count per denomination. `canMakeChange(amount)` and `dispense(amount)` do a greedy largest-coin-first pass and fail if they can't hit the amount exactly with the coins on hand. Greedy is correct for standard coin systems; for an arbitrary denomination set it's a small DP (the coin-change problem). Checking `canMakeChange` *before* transitioning to dispense is what lets the machine refuse cleanly instead of taking the money and getting stuck.

## Concurrency

A physical machine serves one person, so the model is single-threaded — say so. If you must guard against a double-press or a restock during a sale, one lock around the state-transition methods is enough.

## Extensions the interviewer will ask for

- **Card payment** — a `PaymentMethod` strategy; `HasMoneyState` becomes payment-agnostic (`m.addBalance` works whether the balance came from coins or a card auth).
- **Cancel refund policy** — already a transition; the interviewer may want partial refunds or a timeout that auto-cancels (a `TimeoutState` or a timer in `HasMoneyState`).
- **Multiple simultaneous selections / a shopping basket** — a `basket` on the context and a `checkout` transition.
- **Telemetry** — an `Observer` on each vend for stock alerts.

## The one idea to keep

The vending machine reacts to the same four inputs differently depending on its mode, which is exactly what the State pattern is for: one class per mode (`Idle`, `ItemSelected`, `HasMoney`, `Dispense`), each implementing every input, and a transition is just `setState(...)`. The context's public methods become one-line forwards. Check `canMakeChange` before committing to dispense so the machine can refuse instead of eating your money.
