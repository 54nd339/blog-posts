---
title: Network Security - IPsec, TLS, and Application-Layer Protection
description: Security applied at each layer of the stack - Wi-Fi's WEP-to-WPA3 progression, IPsec's AH and ESP with IKE key exchange at the network layer, the TLS handshake at the transport layer, and HTTPS, SSH, and secure email above that.
date: 2022-10-04
draft: false
slug: /computer-networks/network-security
tags:
  - Networking
  - Network Security
  - Cryptography
---

Security isn't one shield at one layer. IPsec can encrypt every IP packet transparently to the applications above it. TLS secures a specific connection between a browser and a server. HTTPS, SSH, and secure email add protections tailored to what those applications do. If one layer is misconfigured, another may still hold.

This post walks the stack from the bottom: Wi-Fi link security and how it went from trivially broken to solid, IPsec at the network layer, TLS at the transport layer, and the application-layer protocols built on top. It assumes the cryptographic primitives — [symmetric](/citadel/cyber-security/symmetric-key) and [public-key](/citadel/cyber-security/public-key) encryption, [digital signatures](/citadel/cyber-security/digital-signatures) — and shows how protocols combine them.

## Wi-Fi security

A [wireless link](/citadel/computer-networks/wireless-networks) broadcasts, so anyone in range can listen. The threats: **eavesdropping** on unencrypted traffic, **unauthorized access** through weak protection, **rogue access points** and **evil twins** (an AP impersonating a legitimate one to intercept traffic), and **denial of service** by jamming or flooding.

The protection evolved in four steps:

- **WEP** — the original. It misuses the RC4 stream cipher with short initialization vectors and weak key management, and is broken in minutes. Obsolete.
- **WPA** — an interim fix: **TKIP** kept RC4 but added per-packet key mixing, a message integrity check (Michael), and dynamic keys. Better than WEP, still limited by RC4.
- **WPA2** — the long-lived standard: **AES** with **CCMP**, giving strong encryption and integrity.
- **WPA3** — **Simultaneous Authentication of Equals (SAE)** resists offline dictionary attacks on the passphrase; open networks get individualized per-device encryption; management frames are protected against forced deauthentication; IoT onboarding is simplified.

Use WPA3 where supported, otherwise WPA2 with a strong unique passphrase.

## Network layer: IPsec

**IPsec** is a framework for securing IP traffic, providing confidentiality, integrity, data-origin authentication, and anti-replay protection. Two modes:

- **Transport mode** — protect only the packet's payload, keep the original IP header. Host-to-host.
- **Tunnel mode** — encrypt the entire original packet and wrap it in a new IP header. Gateway-to-gateway or host-to-gateway **VPNs**.

What to protect is decided by the **Security Policy Database (SPD)**: for each packet, discard it, let it bypass IPsec, or protect it. If protected, the policy points to a **Security Association (SA)** — a one-way agreement fixing the protocol (AH or ESP), the mode, the algorithms, and the keys. Two-way traffic needs two SAs. **Selectors** (addresses, ports, protocol) match traffic to policies.

Two protection protocols:

- **Authentication Header (AH)** — integrity and origin authentication (and optional anti-replay) via an **Integrity Check Value**, a keyed hash (HMAC) over the immutable IP header fields plus the payload. **No encryption.**
- **Encapsulating Security Payload (ESP)** — **confidentiality** by symmetric encryption (AES), plus an optional ICV for integrity and authentication. In transport mode it encrypts the transport segment; in tunnel mode the whole inner packet. ESP is the common choice; if both are used, ESP is applied first and AH authenticates the result.

Keys and algorithm choices come from **IKE (Internet Key Exchange)**, a hybrid protocol: **Diffie-Hellman** establishes a shared secret, authenticated by pre-shared keys, certificates, or public-key encryption. **Phase 1** builds a secure authenticated channel (the IKE SA); **Phase 2** uses it to negotiate the IPsec SAs for AH/ESP. IKEv2 is current.

## Transport layer: TLS

**TLS** (and its deprecated predecessor SSL) sits between the application and TCP, giving a specific connection: server authentication (and optionally client) via certificates, negotiated algorithms and a shared session key, symmetric encryption of the application data, and integrity via MACs.

It has two sub-layers. The **Record Protocol** takes application data, fragments it, optionally compresses it, applies a MAC, encrypts it, and prepends a record header. Above it sit the **Handshake**, **Change Cipher Spec**, and **Alert** protocols.

A simplified handshake:

1. **ClientHello** — TLS version, list of supported cipher suites, a client random.
2. **ServerHello** — chosen cipher suite, version, the server's **certificate** (its public key), a server random; optionally a request for the client's certificate.
3. **(Certificate +) ClientKeyExchange** — the client generates a **pre-master secret**, encrypts it with the server's public key, and sends it (plus its own certificate and proof of private-key possession, if asked).
4. **ServerKeyExchange** — sent only for key-exchange methods like Diffie-Hellman that need extra parameters.
5. Both sides derive the same **master secret** from the pre-master secret and the two randoms, then derive the symmetric **session keys** from it.
6. **Client ChangeCipherSpec + Finished** — switch to the new keys; the encrypted Finished message carries a hash of the whole handshake so far, so tampering is detected.
7. **Server ChangeCipherSpec + Finished** — the server does the same.
8. Application data flows, encrypted and MAC-protected.

## Application layer

**HTTPS** is just HTTP carried over TLS. It encrypts the URL path, headers, cookies, and body, authenticates the server through its certificate, and shows as `https://` with a padlock. See the [HTTPS](/citadel/interview/https) post for the certificate and trust details.

**SSH** is a cryptographic protocol for running network services securely over an untrusted network — remote login (replacing Telnet), file transfer (SFTP, SCP), and tunnelling other protocols. It authenticates the server to the client with a **host key**, and the client to the server by password, public key, or Kerberos, over an encrypted channel.

**Secure email** adds what SMTP lacks (it sends plaintext):

- **PGP** — encrypts the message body with a symmetric key for speed, encrypts that session key with the recipient's public key (a digital envelope), and signs for integrity and sender authentication. Trust comes from a decentralized **web of trust** — users signing each other's keys.
- **S/MIME** — an IETF standard for the same goals (signatures, encryption, non-repudiation), but trust comes from a hierarchical **PKI** with X.509 certificates issued by Certificate Authorities. Built into mainstream mail clients.

## The one idea to keep

The lower the layer, the more it protects transparently but the less it knows. IPsec secures every packet without the application's cooperation, but can't tell a login from a download. TLS secures one connection and knows it's a connection. Application protocols like SSH and PGP know exactly what they're protecting and can make fine-grained choices — at the cost of every application needing its own security code. Real systems layer them: a TLS session inside an IPsec tunnel, an HTTPS request inside that.
