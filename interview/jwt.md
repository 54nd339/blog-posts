---
title: JWT - Stateless Authentication With a Signed Token
description: A JSON Web Token packs identity claims into a signed, self-contained string the server can verify without a session store. How the header, payload, and signature fit together, the request flow that uses them, and the ways JWT authentication goes wrong.
date: 2024-08-22
draft: false
slug: /interview/jwt
tags:
  - System Design
  - Interview Prep
  - Auth
---

Sign in once, and every later request carries proof of who you are — with no session row stored on the server. A **JSON Web Token** (JWT, said "jot") is how that usually works: a compact string that carries your identity claims plus a signature that makes tampering detectable.

This walks through the three parts of a token, the login-to-request flow that uses it, why verifying state instead of looking it up is worth the trouble, and the handful of mistakes — the `none` algorithm, secrets in the payload, the wrong client-side store — that turn JWT auth into a liability.

## The shape of a token

A JWT is three Base64Url-encoded segments joined by dots: `header.payload.signature`. Base64Url encoding is not encryption — anyone holding the token can decode the first two segments and read them.

**Header** — metadata about the token: `typ` (always `JWT`) and `alg`, the algorithm used for the signature, such as `HS256` (HMAC with SHA-256) or `RS256` (RSA signature).

```json
{ "alg": "HS256", "typ": "JWT" }
```

**Payload** — the **claims**, meaning statements about the user plus any data you want to carry. Three kinds:

- **Registered claims** — a standard set (not mandatory) with defined meanings: `iss` (issuer), `sub` (subject, usually the user ID), `aud` (audience, who the token is for), `exp` (expiry timestamp), `iat` (issued-at timestamp).
- **Public claims** — names you choose, namespaced by URI or registered in the IANA registry so they don't collide with anyone else's.
- **Private claims** — custom names agreed between the parties using the token, specific to your application.

```json
{ "sub": "user123", "name": "Aditi Rao", "admin": true, "iat": 1516239022, "exp": 1516242622 }
```

**Signature** — what makes the header and payload trustworthy. Concatenate the encoded header and payload with a dot between them, then sign that string with the algorithm from `alg`:

```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

With a symmetric algorithm like `HS256`, the same `secret` signs and verifies, so every party that can check a token can also mint one. With an asymmetric pair like `RS256`, the private key signs and the public key only verifies — so verifiers cannot forge tokens, and a changed byte anywhere in the header or payload breaks the check.

## The authentication flow

1. **Login** — the client sends credentials to an authentication server.
2. **Issue** — the server validates them, builds the header and payload, signs the token, and returns it. Because the claims travel inside the token, the server stores nothing.
3. **Store** — the client keeps the token: `localStorage`/`sessionStorage`, mobile secure storage, or an `HttpOnly` cookie.
4. **Send** — on each request to a protected route, the client attaches it, conventionally as `Authorization: Bearer <token>`.
5. **Verify** — the server checks the signature with the shared secret or public key, then checks the claims: `exp` not passed, and `iss`/`aud` as expected.
6. **Decide** — valid token, the server trusts the claims and authorizes using fields like `sub` and roles; invalid token, it returns `401` or `403`.

Any server instance holding the key can do step 5, which is the whole point: no shared session store to consult or replicate.

## Why verify instead of look up

Session authentication stores a session ID on the server and reads the matching record on every request. JWT moves that record into the token itself, which buys a few things:

- **Statelessness** — no per-request session lookup, and any instance behind a load balancer can validate a token without sticky routing or a shared cache.
- **Compactness** — a JWT fits in an HTTP header and is far smaller than an XML token like SAML.
- **Self-contained** — the payload already carries the user ID and permissions, so routine authorization needs no database hit.
- **One standard** — RFC 7519 is implemented in every mainstream language, so tokens issued by one service verify cleanly in another.

## Where JWT auth goes wrong

- **Send over HTTPS only.** A bearer token is a password equivalent; intercepted, it is replayable until it expires.
- **Protect the key.** An `HS256` secret or an `RS256` private key that leaks lets anyone mint valid tokens. Rotate it if you suspect exposure.
- **Always set `exp`, and keep it short.** Minutes, not days. For long sessions, pair a short-lived access token with a separate refresh token that can be revoked server-side.
- **Treat the payload as public.** It is encoded, not encrypted — no passwords, no card numbers. If a claim genuinely must be secret, encrypt it with JWE.
- **Pin the algorithm.** Verify against an allowlist you control; never let the incoming `alg` header decide how you validate. Two classic breaks: accepting `alg: none` (a token with no signature) lets an attacker forge anything, and a server that will verify either `RS256` or `HS256` can be fed an `HS256` token signed with the *public* RSA key as the HMAC secret — a public value — and accept it. Reject `none`, and pin to exactly one algorithm per key.
- **Pick client storage deliberately.** `localStorage` is readable by any script, so an XSS bug leaks the token. An `HttpOnly` cookie is not script-readable but rides along on cross-site requests, so it needs CSRF protection.

## Revocation is the trade you accept

A JWT swaps a database lookup for a signature check, and that is a good deal only while the signature is the hard part to forge — so the key stays secret, the token stays short-lived, and the algorithm stays pinned. The cost is that a signed token is valid until it expires, with no easy "log this user out everywhere" switch. Teams get it back by keeping access tokens brief and putting revocation on the refresh token, or by checking a small deny-list of token IDs — at which point some server-side state has quietly returned.
