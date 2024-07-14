---
title: Securing a Web API - Authentication and Defence in Depth
description: An API exposes data and logic to callers you do not control. The authentication mechanisms — token, HMAC, API key, OAuth 2.0 — and the layers around them, from input validation to rate limiting to the OWASP API Top 10.
date: 2024-07-14
draft: false
slug: /interview/secure-web-api
tags:
  - System Design
  - Interview Prep
  - Security
---

An API exposes business logic and data to clients you do not fully control — mobile apps, single-page apps, third parties, other services. A compromised one means data theft, unauthorised access, service disruption, and lost trust. Security here is not one control but a stack of them. This covers the authentication options first, then the layers that sit around authentication.

Three principles run through all of it: **defence in depth** (never one mechanism), **least privilege** (grant the minimum a caller needs), and **secure by design** (built in from the start, not bolted on).

## Authenticating the caller

![Two flows. Token-based: the client sends a password to an authentication server, receives a token, then sends requests to the web server with the token to get resources. HMAC: the client requests an API key from the authentication server, then for each request generates hmac A from the public app ID, request URI, request content, HTTP method, timestamp, and nonce signed with the API key; the server regenerates hmac B the same way and compares.](../images/secure-web-api.png "Token-based auth trusts a signed token per request; HMAC signs each request's contents with a shared secret, so tampering and replay are both detectable. Source: system-design illustration.")

### Token-based

The common choice for web and mobile. The client sends credentials to an auth server, gets back a short-lived access token (usually a [JWT](/citadel/interview/jwt)), and attaches it as `Authorization: Bearer <token>` on every request. The API validates the token's signature, expiry, and claims. Access lasts until the token expires.

### HMAC

Suited to server-to-server calls. The server issues the client a public app ID and a secret API key. Per request, the client builds a string from the app ID, request URI, body, HTTP method, a timestamp, and a nonce, then signs it with the secret key to produce `hmac A`, sent in a header. The server rebuilds the same string, signs it with its copy of the key to get `hmac B`, and compares. Because the body is in the signature, tampering is caught; because the timestamp and nonce are in it, a captured request cannot be replayed.

### API key

A static secret string per client, sent in `X-API-Key` or a query parameter. Simple, good for identifying the calling *application* for quotas and analytics, and for low-sensitivity access. Its weaknesses: it is long-lived, so a leak persists until rotation; on its own it gives no integrity or replay protection; and it authenticates the app, not the user. **Levelled keys** — separate read-only and read-write keys — apply least privilege.

### OAuth 2.0

An open standard for **delegated authorization**: let a third-party app act on a user's resources without seeing the user's password. Four roles — **resource owner** (user), **client** (the app), **authorization server** (authenticates the user, issues tokens), **resource server** (holds the data, validates tokens).

![Four OAuth 2.0 flows drawn as sequence diagrams. Authorization Code: browser to server to identity provider, returning an authorization code that the server exchanges for an access token. Client Credentials: a client app authenticates with client ID and secret to get an access token directly. Implicit: the token is returned straight to the browser. Resource Owner Password: the user hands their username and password to the client app, which exchanges them for a token.](../images/oauth2.0.png "The four grant types. Authorization Code (with PKCE) is the default; Client Credentials is for machine-to-machine; Implicit and Resource Owner Password are legacy. Source: ByteByteGo.")

The **authorization code** flow, the default:

1. The client redirects the user to the authorization server with its client ID, requested **scopes**, and a redirect URI.
2. The user authenticates there and approves the scopes on a consent screen.
3. The authorization server redirects back to the client with a single-use **authorization code**.
4. The client exchanges that code — plus its client ID and secret, over a back channel — for an **access token** (and often a refresh token).
5. The client calls the resource server with `Authorization: Bearer <token>`.
6. The resource server validates the token and serves the request.

Other grants: **client credentials** for machine-to-machine (no user involved); **implicit** and **resource owner password**, both legacy and discouraged — single-page apps now use authorization code with **PKCE**. OAuth also underpins "Log in with Google" and [SSO](/citadel/interview/sso).

## The layers around authentication

- **HTTPS always.** Encrypt everything in transit; see [HTTPS](/citadel/interview/https).
- **Authorization.** Authentication says *who*; authorization checks *may they* — enforce least privilege per resource and action. The `401` versus `403` distinction is in [status codes](/citadel/interview/http-codes).
- **Input validation.** Validate every parameter, body, and header against expected type, length, and range. The primary defence against injection and XSS.
- **Rate limiting.** Cap requests per client per window, keyed on user, IP, or action, to blunt DoS and runaway clients (`429`).
- **API gateway.** A single entry point centralising auth, rate limiting, routing, and logging.
- **Error handling.** Useful messages to the client; never a stack trace, a SQL error, or an internal path.
- **Versioning.** `/v1/`, `/v2/` so breaking changes do not break existing clients.
- **Security headers.** `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options`.
- **The OWASP API Security Top 10.** The current list of the most common API vulnerabilities — check against it.
- **Logging and monitoring.** To spot abuse and diagnose incidents.

## Authentication is the front door, not the whole house

The auth mechanism gets the most attention in an interview, but every real breach post-mortem is a chain: a valid token *plus* missing authorization on an endpoint, or unvalidated input *plus* a verbose error that leaked the schema. That is why the principle is defence in depth — each layer here is cheap on its own, and the security comes from an attacker having to defeat several at once. Pick the auth method by the caller (token for users, HMAC or client-credentials for services, OAuth for third parties), then assume it will be bypassed and build the rest.
