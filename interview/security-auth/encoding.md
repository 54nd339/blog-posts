---
title: Encoding, Encryption, and Tokenization - Three Different Jobs
description: All three transform data, but for different reasons and with different security value. Encoding for representation, encryption for confidentiality, tokenization for taking sensitive data out of scope.
date: 2025-01-26
draft: false
slug: /interview/encoding
tags:
  - System Design
  - Interview Prep
  - Security
---

All three take data in one form and produce another. The confusion is worth clearing up because their purposes barely overlap: one makes data portable, one makes it secret, one removes it from your systems entirely. Using the wrong one — Base64 to "hide" a value, say — is a classic mistake.

This lays out what each does, how it is reversed, and where it belongs.

## Encoding: representation, not protection

Encoding converts data to another format for **usability, interoperability, or efficiency**. It uses a public algorithm and no key, so anyone who knows the scheme can reverse it. It provides zero confidentiality — treat encoded data as plaintext.

Common schemes:

- **Base64** — binary as a radix-64 ASCII string, so binary rides inside text formats (a [JWT](/citadel/interview/jwt) payload, a small image in CSS, a binary blob in JSON).
- **URL (percent) encoding** — reserved and unsafe characters become `%` plus two hex digits; a space is `%20`.
- **Character encodings** — ASCII, UTF-8, UTF-16 define how characters map to bytes. UTF-8 dominates the web.
- **Serialization formats** — JSON, XML, and binary ones like [Protocol Buffers](/citadel/interview/grpc) and Avro, which turn a data structure into something storable or transmissible and back. Protobuf's binary form is smaller and faster to parse than text.
- **Binary framing** — [HTTP/2](/citadel/interview/http) breaks messages into binary frames for efficient parsing.

One security-adjacent use: *output encoding* (HTML-entity encoding of user input) prevents [XSS](/citadel/interview/secure-web-api) by rendering data as inert text rather than executable markup. That is encoding used defensively — still not secrecy.

## Encryption: confidentiality with a key

Encryption transforms **plaintext** into **ciphertext** with a cryptographic algorithm and a key, so only holders of the right key can recover the plaintext. Strength rests on the algorithm and the secrecy and length of the key.

**Symmetric** — one key encrypts and decrypts. Fast, so it does the bulk work: data at rest, and the payload of a secure session once a key is agreed. The hard part is distributing that shared key safely. AES and ChaCha20 are current; DES and 3DES are retired.

**Asymmetric** — a mathematically linked key pair. A **public key** anyone can hold, a **private key** the owner guards.

- *Confidentiality:* encrypt with the recipient's public key; only their private key opens it.
- *Signatures:* sign with your private key; anyone verifies with your public key, confirming origin and integrity.

It solves symmetric's key-distribution problem, but it is far slower, so it is used on small things — session keys, credentials, signatures.

**Hybrid** — what real systems do: asymmetric to exchange a symmetric session key, then symmetric for the actual data. Every [TLS](/citadel/interview/https) and [SSH](/citadel/interview/ssh) handshake works this way.

Encryption protects data in transit (HTTPS, SSH, VPNs) and at rest (disk and database encryption, S3 server-side encryption), with a key-management service like a cloud KMS holding the keys.

## Tokenization: removing data from scope

Tokenization replaces a sensitive value with a **token** — a substitute with no exploitable meaning — and keeps the real value in a separate, hardened **token vault**.

1. An app captures a sensitive value (a card number).
2. It sends the value to the tokenization service.
3. The service stores the original in the vault, mapped to a freshly generated token.
4. The token goes back to the app, which uses it for storage, analytics, and most processing.
5. When the real value is genuinely needed (charging a card), an authorized system exchanges the token for it — *detokenization*.

Because the token has no algorithmic relationship to the original, it cannot be reversed without the vault. The payoff is **scope reduction**: far fewer systems ever touch real data, which is why it is standard for shrinking [PCI DSS](/citadel/interview/payment-ecosystem) obligations on card numbers and for keeping PII out of analytics and support tools. Tokens can be **format-preserving** (same length, passes a Luhn check) so existing systems need no changes.

## Side by side

| | Encoding | Encryption | Tokenization |
| --- | --- | --- | --- |
| Goal | representation, compatibility | confidentiality | remove sensitive data from systems |
| Reversal | public algorithm, no key | correct key only | vault lookup only |
| Security value | none | high | high |
| Key involved | no | yes | no (but the vault must be protected) |
| Data location | flows everywhere | flows everywhere, scrambled | real data isolated in a vault |
| Examples | Base64, UTF-8, Protobuf | AES, RSA, TLS | card and PII tokenization |

## Match the transform to the reason

The question to ask is not "how do I change this data" but "why". Needs to survive a text-only channel — encode. Must stay secret from anyone without the key — encrypt. Should not be in your database at all if you can avoid it — tokenize. They compose, too: a tokenized field, encrypted at rest, encoded for transport. What none of them is a substitute for is [hashing a password](/citadel/interview/salting), which is deliberately one-way.
