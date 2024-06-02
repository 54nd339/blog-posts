---
title: Single Sign-On - One Login for Many Applications
description: SSO lets a user authenticate once with an identity provider and reach many independent applications without logging in again. The redirect dance between service provider and identity provider, the components, and what SAML and OAuth contribute.
date: 2024-06-02
draft: false
slug: /interview/sso
tags:
  - System Design
  - Interview Prep
  - Auth
---

**Single sign-on** lets a user authenticate once and then reach many independent applications without re-entering credentials. One strong password, held at one place, instead of a dozen weak reused ones. Like an all-access pass: checked once at the gate, honoured at every ride.

## The components

- **User** — the person accessing applications.
- **Service Provider (SP)** — an application the user wants in: Gmail, Salesforce, an intranet.
- **Identity Provider (IdP)** — the trusted system that authenticates the user and issues **authentication tokens** (assertions) to SPs. This is the "SSO server".
- **Authentication token** — the proof of authentication passed from IdP to SP.

## The flow

![A sequence diagram titled How does SSO Work. The user enters credentials on an SSO login page shown by Gmail (Domain 1); Gmail requests authentication from the SSO authentication server, which creates a token and a global session; Gmail authenticates with the token, the server registers the Gmail system and validates the token, and Gmail returns protected resources. The user then navigates to YouTube (Domain 2), which requests authentication; the server replies "already logged in" and issues a token; YouTube authenticates with it, the server registers the YouTube system and validates, and YouTube returns protected resources.](../images/sso.png "The second application gets the user in without a login prompt because the identity provider already holds a global session from the first. Source: ByteByteGo.")

Priya opens **Gmail**:

1. Gmail sees no session and redirects her to the IdP.
2. The IdP has no session either, so it shows the SSO login page. Priya enters her one set of credentials.
3. The IdP validates them, creates a **global session** for Priya, and issues a token.
4. The IdP redirects back to Gmail with the token. Gmail validates it with the IdP, which confirms it and records that Gmail is part of Priya's session.
5. Gmail grants access.

Priya then opens **YouTube**:

6. YouTube sees no local session and redirects to the IdP.
7. The IdP finds Priya's **existing global session** — no credential prompt. It returns a token to YouTube.
8. YouTube validates it with the IdP and grants access.

Two applications, one login.

## Why do it

- **User experience** — one password, faster logins, less frustration.
- **Security** — less password fatigue means fewer weak and reused passwords; [MFA](/citadel/interview/identity-management) and password policy are enforced once, centrally, at the IdP; and revoking a departing employee's access is one action instead of many.
- **Administration** — identities and permissions managed from one place.

## The standards underneath

- **SAML** — an XML standard for exchanging authentication and authorization assertions between an IdP and SPs. Common in enterprise SSO.
- **OpenID Connect** — an identity layer on top of [OAuth 2.0](/citadel/interview/secure-web-api), the modern choice for web and mobile SSO.
- **OAuth 2.0** itself is authorization, not authentication, but SSO systems lean on it heavily for third-party access and "Log in with Google".

## SSO concentrates trust on purpose

The trade is explicit: every application now trusts one identity provider, so a compromise of the IdP is a compromise of everything behind it — which is exactly why the IdP is where you put the strongest authentication, the tightest monitoring, and the fastest revocation. Done right, that concentration is a security *gain*, because it replaces a dozen inconsistently-secured login systems with one that gets all the attention. It is the visible half of the [identity management](/citadel/interview/identity-management) and [federation](/citadel/interview/identity-management) picture.
