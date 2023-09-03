---
title: Web Application Security - The OWASP Top 10 in Depth
description: The defensive view of the vulnerability classes that dominate web compromise. Each has one root cause - input treated as code or as a capability - and one real fix that is not a WAF: parameterized queries, contextual encoding, deny-by-default access control, and not deserializing untrusted data.
date: 2023-09-03
draft: false
slug: /cyber-security/web-app-security
tags:
  - Security
  - Web
---

The [software security](/citadel/cyber-security/info-security) post covers memory-safety and native vulnerabilities. This one is the web: the classes of bug that turn up in almost every application security review, catalogued by the OWASP Top 10 and its more granular cousin the OWASP Application Security Verification Standard (ASVS). The [offensive walkthrough](/citadel/pentesting/web-exploitation) shows the exploitation; here the emphasis is on *why each bug exists* and *the fix that actually closes it* — which, for most of them, is not a web application firewall.

The unifying idea: a web app takes input from an untrusted source, and a vulnerability is what happens when that input is treated as **code** (executed) or as a **capability** (authorized) instead of as inert data.

## Broken access control

The most common serious finding. The app authenticates the user but doesn't consistently check *authorization* on every request. Symptoms: **IDOR** (change `?account=1023` to `?account=1024` and see someone else's data), forced browsing to admin URLs, a client-side role check with no server-side equivalent, missing function-level checks on an API.

**Root cause:** access control implemented per-endpoint, ad hoc, so some endpoints get missed. **Fix:** deny by default; centralize authorization (a middleware or policy layer that every request passes through); scope every data lookup to the authenticated principal (`WHERE owner_id = :current_user`), never trust an ID from the client to imply ownership; and check permissions on the *object*, not just the *action*.

## Injection

Input concatenated into a string an interpreter then parses — SQL, OS commands, LDAP, XPath, NoSQL query operators. **SQL injection** is the archetype and still appears constantly.

**Root cause:** the query's structure and its data are built in the same string, so data can change the structure. **Fix:** **parameterized queries / prepared statements** — the driver sends the query template and the parameters over separate channels, so a parameter can never become SQL. Same principle everywhere: pass command arguments as an array (`execve`-style), never a shell string; use parameterized LDAP/XPath APIs; validate that a value meant to be an integer *is* an integer. Escaping, blocklists, and WAFs are defence-in-depth, not the fix — they have bypasses.

## Cryptographic failures

Sensitive data exposed because it wasn't encrypted, or was encrypted badly: plaintext or reversibly-"encrypted" passwords, secrets in code, TLS not enforced, ECB mode, a hardcoded key or IV, a homemade cipher, MD5/SHA-1 where a slow KDF was needed. **Fix:** classify data, encrypt in transit (TLS everywhere, HSTS) and at rest; store passwords with **Argon2id** / scrypt / bcrypt (slow, salted); use vetted AEAD constructions (AES-GCM, ChaCha20-Poly1305) via a high-level library (libsodium); keep keys in a KMS/HSM and rotate them; never invent crypto.

## Insecure design

A category for flaws that aren't bugs in the code but in what the system was designed to do — no rate limiting on a password-reset flow, a "security question" as a factor, trusting the client to compute a price, a business-logic race. **Fix:** threat model during design (STRIDE, abuse cases), define security requirements alongside functional ones, and build in limits (rate, resource, transaction) as first-class features.

## Security misconfiguration

Default credentials, verbose error pages and stack traces, directory listing, unnecessary features enabled, missing security headers, an overly permissive CORS policy, cloud storage left public, an admin console exposed. **Fix:** hardened baseline images, infrastructure as code so config is reviewed and repeatable, least functionality (disable what you don't use), and a repeatable process to apply the same hardening to every environment.

## Vulnerable and outdated components

Most application code is dependencies, and a known-vulnerable version of one (Log4j, a serialization library, an image parser) is exploitable through your app. **Fix:** a software bill of materials, **SCA** scanning (`Dependabot`, `Snyk`, `Trivy`) in CI, a patch cadence, and removing dependencies you don't need.

## Identification and authentication failures

Weak passwords allowed, no MFA, credential stuffing not mitigated, session IDs in the URL, sessions that don't rotate on login (session fixation) or don't expire, password recovery that leaks whether an account exists. **Fix:** check passwords against breach lists (don't impose silly composition rules), offer phishing-resistant MFA, rate-limit and monitor login, generate long random session tokens, rotate on privilege change, set idle and absolute timeouts, and make error messages non-enumerable.

## Software and data integrity failures

Trusting code or data whose integrity you haven't verified: an unsigned auto-update, a CI/CD pipeline that pulls a build script from an unpinned source, and **insecure deserialization** — reconstructing objects from client-supplied bytes (Java `ObjectInputStream`, Python `pickle`, PHP `unserialize`, .NET `BinaryFormatter`), where a gadget chain of classes already on the classpath composes into remote code execution. **Fix:** sign and verify updates and pipeline artifacts (Sigstore, SLSA); pin dependencies by hash; **don't deserialize untrusted data** — use a data-only format with a strict schema, and if you must, sign the payload and restrict allowed types to a tight allowlist.

## Security logging and monitoring failures

You can't respond to what you can't see. Missing logs for auth events, access-control failures, and high-value transactions; logs without enough context; no alerting. **Fix:** log the security-relevant events (login success/failure, access denied, input validation failures, privilege changes) with correlation IDs, ship them somewhere tamper-resistant, and alert on patterns. Ties into [observability](/citadel/interview/observability) and [blue-team](/citadel/cyber-security/blue-team).

## Server-side request forgery (SSRF)

The app fetches a URL the user controls, and the request originates from *inside* the network — reaching the cloud metadata service (instance credentials), internal admin panels, or unauthenticated internal services. **Fix:** an allowlist of permitted destinations; resolve the hostname and block link-local and private ranges *after* resolution (defeats DNS rebinding); disable redirects; use IMDSv2 or drop the instance's ambient credentials.

## Defence in depth around all of it

- **Input validation** at the trust boundary (positive/allowlist where possible) — reduces surface but is not the primary fix for any class above.
- **Output encoding** contextual to the sink (HTML, attribute, JS, URL, CSS) — the real fix for XSS, paired with a **Content Security Policy** that forbids inline script and `HttpOnly` cookies.
- **Security headers** — CSP, HSTS, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`, and framing controls.
- **CSRF** — SameSite cookies plus anti-CSRF tokens for state-changing requests.
- **WAF** — catches opportunistic scanning and buys time to patch; never the control you rely on.
- **SAST / DAST / IAST** in CI, plus periodic manual testing against the [OWASP WSTG](/citadel/pentesting/pentesting).

## The one idea to keep

Nearly every web vulnerability is input treated as code (injection, XSS, SSTI, deserialization) or as a capability (broken access control, SSRF, IDOR), and each has one real fix that isn't a WAF: parameterized queries for injection, contextual output encoding plus CSP for XSS, deny-by-default authorization scoped to the authenticated principal for access control, a resolved-IP allowlist for SSRF, and not deserializing untrusted data. Everything else — input validation, security headers, SCA, logging — is layered defence that assumes one of those primary fixes will eventually be missed.
