---
title: HTTPS - The TLS Handshake, Step by Step
description: HTTPS is HTTP inside a TLS tunnel. How the handshake authenticates the server with a certificate, agrees a shared session key using public-key crypto, then switches to fast symmetric encryption for the rest of the session.
date: 2024-07-07
draft: false
slug: /interview/https
tags:
  - System Design
  - Interview Prep
  - Security
---

HTTPS is plain [HTTP](/citadel/interview/http) carried inside a **TLS** tunnel. Intercept the traffic and you get unreadable bytes; tamper with it and the other end notices. The interesting part is the setup: before any request is sent, client and server run a handshake that authenticates the server, agrees a secret key over a channel an eavesdropper is watching, and then hands off to fast symmetric encryption.

![A sequence diagram of an HTTPS connection. First a TCP handshake (SYN, SYN-ACK, ACK). Then the TLS handshake under asymmetric encryption: Client Hello, Server Hello, Certificate, Server Hello Done, Client Key Exchange (the client encrypts a session key with the server's public key), Change Cipher Spec, Finished both ways. Then data transmission under symmetric encryption using the shared session key.](../images/https.png "The handshake uses the server's public/private key pair only to get a shared session key across; all application data then uses fast symmetric encryption with that key. Source: ByteByteGo.")

## Step 1: TCP handshake

Before TLS, the normal [TCP](/citadel/interview/socket-program) three-way handshake (SYN, SYN-ACK, ACK) establishes a connection both sides agree is open.

## Step 2: hello and certificate

1. **Client Hello** — the browser sends the TLS versions it supports and a list of cipher suites (encryption algorithm combinations) it can use.
2. **Server Hello** — the server picks the strongest version and suite it shares with the client.
3. **Certificate** — the server sends its TLS certificate: its **public key**, the domain names it is valid for, expiry dates, and the **Certificate Authority (CA)** that issued and signed it.
4. **Validation** — the browser checks the certificate against a built-in list of trusted CAs: is the issuer trusted, is it unexpired, does the domain match the site being visited, does the CA's signature verify? Any failure aborts the connection with a warning.

## Step 3: agree a session key

The certificate proves the server's identity and hands over its public key. Now both sides need one shared secret — the **session key**.

In the classic RSA key-transport form: the browser generates a random session key, encrypts it with the server's public key, and sends it. Only the holder of the matching **private key** — the real server — can decrypt it. Both ends now have the same session key, and nobody watching the wire got it.

Modern TLS 1.3 replaces this with an **ephemeral Diffie-Hellman** exchange: both sides contribute to deriving the session key, and the key material is thrown away after the session, so recording today's traffic and stealing the server's private key later still does not decrypt it — the property called **forward secrecy**.

## Step 4: switch to symmetric

Both sides send **Change Cipher Spec** and **Finished**, then all application data — the actual HTTP requests and responses — is encrypted with the shared session key using **symmetric encryption** for the rest of the session.

## Why not use public-key crypto for everything

- **Speed.** Asymmetric operations are far more expensive than symmetric ones. They are fine for a one-time exchange of a small key; they would be a bottleneck applied to every byte of a page.
- **Symmetry.** Public-key encryption is one-directional — anyone with the public key can encrypt *to* the private-key holder. A shared symmetric key gives efficient two-way encryption that only the two participants can read.

## The handshake is the whole security story

Once the session key is set, the encryption is routine; everything that makes HTTPS trustworthy happens in the handshake — the certificate check that you are talking to the right server, and the key exchange that no observer can unwind. This is the same asymmetric-to-establish, symmetric-to-transfer pattern that [SSH](/citadel/interview/ssh) uses, and the [encoding post](/citadel/interview/encoding) covers the two kinds of encryption in their own right.
