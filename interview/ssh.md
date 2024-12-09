---
title: SSH - The Handshake Behind a Secure Shell
description: SSH gives you an encrypted channel to a remote machine over an untrusted network. The connection setup — version and algorithm negotiation, Diffie-Hellman key exchange, host-key verification, user authentication — plus the layered protocol and what channels enable.
date: 2024-12-09
draft: false
slug: /interview/ssh
tags:
  - System Design
  - Interview Prep
  - Security
---

**SSH (Secure Shell)** is the standard way to operate a remote machine over an untrusted network: a shell, a command, or a file transfer, inside an encrypted channel. It replaced Telnet, which sent everything — passwords included — in plaintext. It is client-server, normally on TCP port 22, and it guarantees **confidentiality and integrity** of everything exchanged.

This walks the connection setup, the layered protocol, and the channel multiplexing that makes SSH more than a remote terminal.

![A sequence diagram titled How SSH Works. Steps: establish TCP connection; negotiate supported version; negotiate supported algorithms; the client generates a user key pair; the client sends its public key; a login request; the server finds a matching key in authorized_keys and encrypts a random number with the public key; the client decrypts it with its private key and sends it back; the server verifies the decryption; then session requests, encrypted commands, and encrypted results flow, each decrypted with the session key. A lower panel shows SSH local forwarding tunnelling a local port to a server port through a secure tunnel across a firewall.](../images/ssh.png "The SSH handshake: negotiate, exchange keys, verify the host, authenticate the user, then everything runs under a shared symmetric session key. Below: local port forwarding tunnels another connection through the SSH channel. Source: ByteByteGo.")

## Setting up the connection

1. **TCP connect** to port 22.
2. **Version exchange** — both sides announce supported SSH protocol versions (SSH-2.0) and agree on one.
3. **Algorithm negotiation** — client and server each send lists of supported algorithms and pick a mutual choice for four things: the **key exchange** method (a Diffie-Hellman variant), the **host key** type (RSA, ECDSA, EdDSA), the **symmetric cipher** for bulk data (`aes256-ctr`, `chacha20-poly1305`), and the **MAC** for integrity (`hmac-sha2-256`).
4. **Key exchange** — run Diffie-Hellman so both sides independently compute the same shared secret *without ever sending it*, then derive the session's encryption and MAC keys from it. As part of this, the **server sends its host public key**; the client checks it against `~/.ssh/known_hosts`. A match means the server is authentic. A first connection prompts the user to verify the fingerprint; a *changed* key triggers a warning, because it could mean a man-in-the-middle. This host-key check is what stops an imposter server.
5. **User authentication** — now over the encrypted channel:
   - **Password** — sent encrypted to the server to check.
   - **Public key** — the preferred method. The user's public key sits in `~/.ssh/authorized_keys` on the server; the server issues a challenge, the client signs it with its private key, the server verifies the signature. The private key never leaves the client.
6. **Session established** — all further traffic is encrypted with the session cipher and integrity-checked with the MAC.

## The layered protocol

SSH is three protocols stacked:

- **Transport (SSH-TRANS)** — version and algorithm negotiation, key exchange, server authentication, and the encrypted, integrity-protected byte stream.
- **Authentication (SSH-AUTH)** — runs over the secure transport, authenticates the user.
- **Connection (SSH-CONN)** — multiplexes the one encrypted tunnel into multiple logical **channels**, so a single connection can carry an interactive shell, X11 forwarding, several port forwards, and an SFTP session at once.

## What it is used for

- **Remote login** — a shell on a remote host.
- **File transfer** — SFTP and SCP run over SSH.
- **Remote command execution** — `ssh user@host 'ls -l'`, no interactive session.
- **Port forwarding (tunnelling)** — carry another TCP connection through the SSH channel. *Local forwarding* makes a service on the remote network (a database) look local; *remote forwarding* exposes a local service to the remote side.
- **Git** — `git push`/`pull` authenticated by SSH key.
- **Automation** — scripted access to remote systems.

## The pattern is the same as HTTPS

SSH and [HTTPS](/citadel/interview/https) solve the same problem the same way: negotiate algorithms, use [public-key cryptography](/citadel/interview/encoding) and a Diffie-Hellman exchange to agree a shared secret over an open channel, verify the other end's identity with a key (a host key here, a CA-signed certificate there), then switch to fast symmetric encryption for the session. The differences are in the identity model — SSH's trust-on-first-use `known_hosts` versus HTTPS's certificate authorities — and in what rides the channel afterwards.
