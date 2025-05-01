---
title: How a Visa Card Payment Works - Authorization, Settlement, and the Fees
description: "A card tap is two separate processes: a real-time authorization that checks you can pay, and a batched clearing and settlement, hours later, that actually moves the money. Plus where the merchant's fee goes."
date: 2025-05-01
draft: false
slug: /system-design/visa
tags:
  - System Design
  - Payments
  - Case Study
---

The card tap that "just works" in two seconds is only half the transaction, and it's the half where no money moves. Approving the purchase and paying the merchant are separate flows on separate timescales: authorization happens in real time at the terminal, and settlement happens hours later in a batch. Understanding a card payment means keeping those two apart.

## The players

- **Cardholder** — you.
- **Merchant** — the business being paid.
- **Acquiring bank (acquirer)** — the merchant's bank; provides the POS terminal and the account that receives funds.
- **Issuing bank (issuer)** — your bank, which issued the card.
- **Card network (Visa)** — the intermediary that routes messages and coordinates clearing and settlement between acquirer and issuer. Without it, every bank would have to settle bilaterally with every other bank. Mastercard and American Express are the other large networks.

![The authorization flow: card at POS terminal, to acquirer, to Visa network, routed to issuer for approval, and the response back along the same path](../images/visa.png "Authorization: POS to acquirer to Visa to issuer, and the approval back.")

## Act 1: authorization (real time)

1. Your issuer issued the card.
2. You present it at the merchant's POS terminal.
3. The terminal captures the card details and amount and sends them to the acquirer.
4. The acquirer forwards an authorization request to the Visa network.
5. Visa routes it to your issuer.
6. The issuer checks: is the card valid and active, are there sufficient funds or credit, does anything trip fraud detection. If it passes, the issuer approves and places a **hold** on the amount in your account.
7. The approve/decline travels back through Visa to the acquirer to the terminal — "Approved" or "Declined", in a few seconds.

No money has moved. The merchant has a promise.

## Act 2: capture and settlement (batch)

1. At the end of the day, the merchant **captures** the day's approved transactions into a batch and sends it to the acquirer, which passes it to Visa.
2. **Clearing.** Visa receives batches from many acquirers and sorts them by issuer. It then **nets**: if an issuer owes acquirers ₹1,000 for its cardholders' purchases and acquirers owe that issuer ₹800 for refunds, Visa computes the single net figure rather than moving both. It sends clearing files with these net obligations to each issuer and acquirer.
3. **Settlement.** Issuers confirm the clearing files and transfer the net amounts to the acquirers, through accounts the banks hold at Visa or at a designated settlement bank (often a central bank).
4. The acquirer credits the merchant's account with the captured total, minus fees.

## Where the fee goes

On a ₹100 purchase:

- The merchant pays a **merchant discount fee** to its acquirer — say 2%, so ₹2.
- Most of that, the **interchange fee** — say ₹1.75 — is passed from the acquirer to your issuer. Visa sets interchange rates. The issuer gets the larger share because it carries the risk: cardholder fraud and default, the interest-free period on credit cards, plus account servicing and its share of clearing and settlement work.
- The acquirer keeps the rest — ₹0.25 — as its markup for serving the merchant.
- **Visa's own revenue** is separate: it charges both banks a **network assessment** (a percentage of volume, e.g. 0.11%) and a small fixed **usage fee** per transaction (e.g. ₹1.50, about $0.0195).
- You may also pay your issuer directly — annual fees, interest on balances, foreign-transaction fees.

## The takeaway

The design trick is the split: authorization is optimised for latency (a synchronous round trip in seconds), settlement for efficiency (batched and netted, so a day of transactions becomes a handful of interbank transfers). The same information-flow-versus-fund-flow separation shows up in [account-to-account payments](/citadel/system-design/e-payments), and [SWIFT](/citadel/system-design/swift-payment) is the cross-border version of the messaging half. See [the payment ecosystem](/citadel/interview/payment-ecosystem) for how these fit together.
