---
title: SWIFT and Forex - How Cross-Border Payments Move Messages and Convert Value
description: SWIFT is a secure messaging network for banks, not a way to move money. How a payment message travels through regional and slice processors with acknowledgements at each hop, and how the Forex market handles the currency conversion SWIFT does not.
date: 2024-12-28
draft: false
slug: /system-design/swift-payment
tags:
  - System Design
  - Payments
  - Case Study
---

SWIFT does not move money. It is the **Society for Worldwide Interbank Financial Telecommunication** — a secure, standardised messaging network linking the world's banks, run by its members, based in Belgium, carrying millions of messages a day. It sends payment *instructions*; the funds move separately through banks' correspondent accounts, and any currency conversion happens in the Forex market. This post covers both halves.

![The SWIFT message path from a sending bank through its regional and slice processors, across the network to the receiver's regional and slice processors, and on to the receiving bank, with acknowledgements returning along the way](../images/swift-payment.png "A SWIFT message: regional processors validate and queue, slice processors store and route, acknowledgements flow back.")

## The message journey

Trace a payment message from Bank A (New York) to Bank B (London). The components:

- **Regional processors** in each region — validate incoming messages, queue outgoing ones.
- **Slice processors** — securely store and route messages.

**Phase 1 — sender side (New York).**
1. Bank A sends the message to Regional Processor A, destination Bank B.
2. RP-A validates the format and forwards it to Slice Processor A.
3. SP-A stores the message securely.
4. SP-A confirms storage to RP-A.
5. RP-A returns an **ACK** (will be processed and sent onward) or **NAK** (will not be sent, e.g. a format error) to Bank A.

**Phase 2 — cross-border.**
6. SP-A sends the stored message across the network to Regional Processor B in London.

**Phase 3 — receiver side (London).**
7. RP-B stores the message temporarily.
8. RP-B assigns a **Message Output Number (MON)** and sends it to Slice Processor B.
9. SP-B validates the MON.
10. SP-B authorises RP-B to deliver.
11. RP-B sends the message to Bank B.

**Phase 4 — confirmation back to sender.**
12. Bank B stores the message.
13. Bank B returns a **UAK** (user positive acknowledgement — received without error) or **UNK** (negative — e.g. a checksum failure).
14. RP-B builds a delivery report from that response and sends it to SP-B.
15. SP-B stores the report.
16. SP-B sends a copy back to SP-A in New York.
17. SP-A stores the final delivery report, closing the lifecycle.

Every hop stores and acknowledges — the network guarantees a message is persisted before it is forwarded, and the sender learns the final delivery status.

## Forex: converting the value

When the payer's currency and the payee's differ, the **Foreign Exchange market** does the conversion. Say Bob must pay Alice 100 USD but Alice can only receive EUR, via PayPal:

1. Bob sends 100 USD; it moves from Bob's bank to PayPal's USD account at a bank (Bank P1).
2. PayPal moves the 100 USD to its own USD account at an FX provider, Bank E.
3. The 100 USD is sold into Bank E's **funding pool** — its reserve of currencies.
4. The pool returns the equivalent, say 88 EUR after rates and fees, into PayPal's EUR account at Bank E.
5. PayPal transfers the 88 EUR to Alice's EUR bank account.

### The market's layers

- **Retail** — end-users and PSPs like PayPal, which keep pre-purchased **funding pools** of common currencies for efficiency.
- **Wholesale** — investment and commercial banks and large FX providers, handling the accumulated orders that flow up from retail.
- **Top level** — major multinational banks holding vast currency reserves, often as certificates of deposit from many countries, trading in large volume.

When Bank E's EUR pool runs low, it goes to the wholesale market to sell USD and buy EUR; when the wholesale market has accumulated enough demand, it turns to the top-level participants.

## Messages versus value

Keep the two apart: **SWIFT carries the secure instruction**; the **Forex market and correspondent banking convert and move the value**. A SWIFT message may kick off a payment that needs a conversion, but the conversion itself is entirely Forex-market infrastructure. The same messages-versus-funds split runs through [card payments](/citadel/system-design/visa) and [account-to-account clearing and settlement](/citadel/system-design/e-payments); [the payment ecosystem](/citadel/interview/payment-ecosystem) ties them together.
