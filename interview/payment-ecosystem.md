---
title: The Payment Ecosystem - Who Moves the Money
description: "The cast behind a card tap — cardholder, merchant, acquirer, card network, issuer — and the two flows a transaction runs through: a real-time authorization and a batched clearing and settlement where the money actually moves."
date: 2024-11-05
draft: false
slug: /interview/payment-ecosystem
tags:
  - System Design
  - Interview Prep
  - Payments
---

Tapping a card feels instant. It is not — it is a real-time yes/no check now, and an actual transfer of money between banks hours or days later. Between those two moments sits a chain of institutions, each with a defined role. This lays out who they are and the two flows a transaction passes through.

## The participants

![The payments ecosystem as a diagram. On the acquiring side: merchant to payment gateway to acquiring processor, with a merchant account at the acquiring bank. In the middle, the card network (Visa, Mastercard). On the issuing side: issuing processor to the customer account at the issuing bank, and the cardholder. ISO/MSP partners feed merchants to the acquiring side.](../images/payment-ecosystem.png "The acquiring side (merchant's banks and processors) and the issuing side (cardholder's bank) connected through the card network. Source: ByteByteGo.")

- **Cardholder** — the person paying, using a card from their bank.
- **Merchant** — the business accepting the card.
- **Issuing bank (issuer)** — the cardholder's bank. It issued the card, holds the account, and ultimately owes the merchant's side for approved transactions.
- **Acquiring bank (acquirer)** — the merchant's bank. It contracts with the merchant to accept cards and holds the merchant account through settlement.
- **Card network (scheme)** — Visa, Mastercard, Amex, RuPay. The rails between issuer and acquirer; it sets the rules and technical standards and runs clearing.
- **Payment processor** — handles the technical routing of transaction data, security, and communication with the card networks, on behalf of the acquirer (the *acquiring processor*) or the issuer (the *issuing processor*).
- **Payment gateway** — the online equivalent of a card terminal: captures payment details on the merchant's site and passes them securely to the processor.
- **ISO / MSP** — third parties that sign merchants up for card processing on behalf of an acquirer.

## Flow 1: authorization (real time)

The "is this okay?" check, in seconds:

1. The cardholder presents the card to the merchant.
2. The terminal or gateway sends an authorization request — card details plus amount — to the acquirer/processor.
3. The acquirer forwards it to the card network.
4. The network routes it to the issuing bank.
5. The issuer checks funds or credit, card validity, and fraud signals, then approves or declines. On approval it places a **hold** on the amount in the cardholder's account.
6. The decision travels back — issuer to network to acquirer to terminal — and the cardholder sees "approved".

No money has moved yet.

## Flow 2: clearing and settlement (batched)

Where funds actually transfer, usually end-of-day:

1. **Capture** — the merchant batches the day's authorized transactions.
2. **Submit** — the batch goes to the acquirer, then to the card network.
3. **Clearing** — the network aggregates transactions across all acquirers, **nets** the mutual obligations between issuing and acquiring banks, and sends each issuer its clearing file.
4. **Settlement** — issuers transfer the net amounts to acquirers, often through settlement banks (central banks).
5. **Funding** — the acquirer credits the merchant's account, minus fees (the merchant discount fee; interchange, set by the network, is a component of it).

## Why the split matters

The **information flow** (authorization, clearing data) and the **fund flow** (money between banks) are separate systems on separate timelines. The authorization makes it *look* instantaneous; settlement is a batched, netted, multi-hour-to-multi-day process. Anyone building payments has to design for both — a successful authorization is a promise, not a completed payment.

Knowing the chain also tells you where the engineering problems are: external processor calls need [timeouts and idempotent retries](/citadel/interview/data-systems) because they cross company boundaries; card numbers get [tokenized](/citadel/interview/encoding) so most systems never store a real PAN; and the fee structure (interchange plus the acquirer's markup) is the economics every payment product is built around.
