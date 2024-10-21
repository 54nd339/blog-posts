---
title: Identity Management - Who You Are and What You Can Do
description: "The three questions behind every login — who are you, can you prove it, what may you do — and the mechanisms that answer them: password hashing, MFA and TOTP, tokens, OAuth 2.0, RBAC, and single sign-on."
date: 2024-10-21
draft: false
slug: /interview/identity-management
tags:
  - System Design
  - Interview Prep
  - Auth
---

Every access decision comes down to three questions asked in order: **who are you** (identification), **can you prove it** (authentication), and **what are you allowed to do** (authorization). Identity management is the discipline of answering all three and managing the identities involved over their lifetime. This walks each, and the federation layer that lets one login work across many systems.

## Identification

A user or service claims an identity with a unique identifier — a username, email, employee ID. No verification yet; just the claim.

## Authentication

Confirming the claim. Modern systems combine factors from different categories.

**Something you know — passwords.** The default, and the weakest alone. Storing them safely means **salting and hashing**: prepend a unique random salt to each password before hashing, so two users with the same password get different stored hashes and precomputed (rainbow-table) attacks are defeated. The salt is stored in plaintext beside the hash. To verify a login: fetch the user's salt, hash `salt + entered password`, compare to the stored hash. The details are in [salting](/citadel/interview/salting).

**Something you have — MFA.** Require a second, independent credential. The common form is a **TOTP** app (Google Authenticator): at setup the server shares a secret with the app via QR code; thereafter both compute the same 6-digit code from that secret plus the current 30-second time window. An attacker with the password still lacks the code.

**Something you are — biometrics.** Fingerprint, face, voice. Convenient as a factor; not revocable if compromised.

**Tokens.** After a successful login the server issues a token — usually a [JWT](/citadel/interview/jwt) — carrying signed claims (user ID, roles, expiry). The client sends it in the `Authorization` header on later requests, and the server verifies the signature and claims without a session store.

**API keys.** A static secret per client, for server-to-server calls. Simple, but lacking expiry, scopes, and the other controls tokens and OAuth provide.

## Authorization

What an authenticated principal may do.

**RBAC.** Attach permissions to **roles** (`admin`, `editor`, `viewer`) and assign users to roles, so permission changes happen in one place instead of per user.

**OAuth 2.0 — delegated authorization.** Lets a third-party app act on a user's resources at another service *without* seeing the user's password. Four roles: the **resource owner** (user), the **client** (third-party app), the **authorization server** (issues access tokens), the **resource server** (holds the data). After the user authenticates and consents, the authorization server issues the client an **access token**, which the client presents to the resource server. Grant types — authorization code, client credentials, and the legacy implicit and password grants — cover different client situations.

## Federation and single sign-on

**SSO** lets one login open many independent applications:

1. The user opens app A and is not logged in.
2. App A redirects to a central SSO server.
3. The user authenticates there; the SSO server creates a global session and a token.
4. App A is told the user is authenticated (a redirect back with the token).
5. The user opens app B on the same SSO system; the SSO server sees the existing global session and grants access with no second login.

**Identity federation** is the broader arrangement: separate security domains establish trust so a user from one can access resources in another, using **SAML** or **OpenID Connect** (built on OAuth 2.0). SSO is usually the visible result of federation. The flow is covered in detail in [single sign-on](/citadel/interview/sso).

## Why centralise it

- **Security** — one enforced place for authentication strength, password policy, and revocation, instead of each app rolling its own.
- **User experience** — SSO and smooth MFA cut login friction.
- **Compliance** — GDPR, HIPAA, and similar mandate strong access control and auditability.
- **Operations** — provisioning and, critically, *de-provisioning* a departing employee is one action, not twenty.

## The three questions do not collapse into one

The frequent mistake is treating authentication as the whole job — verify the password and you are done. But a valid login says nothing about permissions (that is authorization, and `401` versus `403` in [status codes](/citadel/interview/http-codes) marks the boundary), and neither says anything about lifecycle: the account that was correctly authenticated and authorized last year is a risk if nobody disabled it when the person left. Identity management is all three, continuously, not a login form.
