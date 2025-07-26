---
title: How QR Code Payments Work - Merchant-Presented vs Customer-Presented
description: "Scanning a QR code to pay comes in two shapes: the customer scans the merchant's code, or the merchant scans the customer's. How each flows through a payment service provider to the banks, and why UPI made the model ubiquitous in India."
date: 2025-07-26
draft: false
slug: /system-design/scan-to-pay
tags:
  - System Design
  - Payments
  - Interview Prep
---

Paying by scanning a QR code has gone from novelty to default across much of Asia, and it runs on cheaper rails than cards. The mechanics come in two arrangements depending on who shows the code and who scans it — and in both, the QR itself carries only identifiers, not your account details.

![A scan-to-pay transaction: a customer app and a merchant, one displaying a QR code and the other scanning it, both connected through payment service providers to the customer's and merchant's banks](../images/scan-to-pay.png "The customer app, the merchant, their PSPs, and the two banks.")

## The players

- **Customer** — with a payment app (a UPI app, a wallet).
- **Merchant** — displays a QR code, or has a device to scan one.
- **Payment service provider (PSP)** — moves the transaction between the two: PayPal, Venmo, or in India Paytm, PhonePe, Google Pay, all on UPI.
- **Customer's funding source** — bank account, linked card, or wallet balance.
- **Merchant's bank** — receives the credit.

## Merchant-presented (you scan, they receive)

The common case, especially for small businesses.

1. The merchant displays a QR code:
   - **Static** — printed, containing only the merchant's identifier (a UPI virtual payment address or merchant ID). The customer types the amount into their app.
   - **Dynamic** — generated per transaction by the merchant's POS, embedding the identifier *and* the exact amount, which the app pre-fills.
2. The customer's app scans it.
3. Knowing payee and amount, the app tells its PSP to authorise and initiate the payment. This is a **push** payment — the customer sends the funds.
4. The PSP debits the customer's bank and credits the merchant's, through the underlying network (UPI in India).
5. Real-time confirmation goes to both sides — a sound on the merchant's device, a success screen in the app.

Static QR codes cost a merchant nothing beyond printing, which is why street vendors use them; dynamic codes suit integrated retail.

## Customer-presented (they scan, you pay)

Roles reversed.

1. The customer's app generates a QR code — a one-time payment token or their authenticated payment identity.
2. The merchant's scanner or POS reads it.
3. The merchant's POS initiates the request through *its* PSP, either pulling a known amount or prompting the customer for final approval on their phone.
4. The PSP settles with the banks.
5. Both sides get confirmation.

This suits supermarkets, transit gates, and vending machines, where the merchant side already has scanning hardware.

## The common flow

Either direction reduces to: scan and initiate → app or POS sends details to its PSP → PSP gets authorisation and debits the payer → PSP credits the merchant → both systems confirm.

As with [card payments](/citadel/system-design/visa), the instant confirmation is the authorisation and information flow. The actual interbank movement happens later through [batched clearing and settlement](/citadel/system-design/e-payments).

## UPI

India's Unified Payments Interface made scan-to-pay routine, from tea stalls to chains. Its distinguishing traits:

- **Push model** — the customer initiates from their bank account after scanning.
- **Interoperability** — any UPI app can scan any UPI merchant's code; no app lock-in.
- **Static, dynamic, and P2P** — merchant VPAs, per-amount dynamic codes, and customer-generated codes for person-to-person transfers all coexist.

Its low fees relative to card processing are the reason merchants pushed adoption.

## Security and the takeaway

A QR code holds a tokenised identifier, not raw account numbers; the payment runs over the PSP's encrypted channels with a step-up check like a UPI PIN. The model's real advantage is cost — a printed code versus a card terminal and card interchange — which is what let it reach merchants that card payments never did. See [the payment ecosystem](/citadel/interview/payment-ecosystem) and the personal take in [where online payments are heading](/citadel/random-thoughts/online-payments).
