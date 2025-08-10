---
title: Protecting Sensitive Data - A Defense-in-Depth Checklist
description: What counts as sensitive data, what a breach costs, and the layered controls that guard it through its lifecycle — classification, access control, encryption in transit and at rest, secret management, safe password storage, masking, DLP, and compliance.
date: 2025-08-10
draft: false
slug: /system-design/sensitive-data
tags:
  - System Design
  - Security
  - Compliance
---

No single control protects sensitive data. Encryption doesn't help if access control is loose; least-privilege doesn't help if secrets are hard-coded in a repo. Protection is layered — defense in depth — so that one failure doesn't expose everything. This is a working checklist of those layers, organised by where they sit in the data's lifecycle.

## What is sensitive data

Any information that would harm a person's privacy or an organisation's security if disclosed:

- **PII** — name, address, email, phone; national IDs (US Social Security number, India's Aadhaar); driver's licence and passport numbers; date of birth.
- **Financial** — card numbers, bank details, CVV, transaction and investment history.
- **Protected health information (PHI)** — medical records, diagnoses, treatment, insurance.
- **Secrets** — passwords, API keys, access tokens ([JWTs](/citadel/interview/jwt) included), SSH keys, private encryption keys, database credentials, certificates.
- **Intellectual property** — trade secrets, proprietary algorithms, source code, designs, research.
- **Confidential business data** — unpublished financials, strategy, customer lists, M&A details.

## What a breach costs

Direct remediation (forensics, legal, fines) and indirect loss (churn, reputation). Regulatory penalties under **GDPR**, **CCPA**, India's **DPDPA**, **HIPAA**, or **PCI DSS** can be large. For individuals, identity theft and fraud. Trust, once lost, is slow to rebuild.

## The layers

### 1. Classify and minimise

Know what sensitive data you hold, where it lives, and who can reach it. Label it by sensitivity (public / internal / confidential / restricted) so controls can be applied by tier. Then **collect and keep only what you need** — data you don't hold can't leak.

### 2. Access control

- **Authentication** — strong password policy, [2FA/MFA](/citadel/system-design/2fa), token-based auth for APIs.
- **Authorization** — the **principle of least privilege**: every user and service gets the minimum permissions for its job, nothing spare.

### 3. Encryption

- **In transit** — TLS/HTTPS on every hop, including service to service, against eavesdropping and man-in-the-middle. See [HTTPS](/citadel/interview/https).
- **At rest** — encrypt data in databases, file systems, object storage, and backups. Cloud object stores offer options like S3's SSE-S3, SSE-KMS (customer-managed keys), SSE-C (client-provided keys), and client-side encryption.
- **Key management** — use a dedicated KMS (AWS KMS, Azure Key Vault, Google Cloud KMS, HashiCorp Vault) to generate, store, rotate, and gate access to the keys themselves.

### 4. Secret management

Hard-coded credentials and plaintext config secrets are a leading breach cause. Use a dedicated system — **HashiCorp Vault**, **AWS Secrets Manager**, **Azure Key Vault**, **Google Cloud Secret Manager** — for encrypted central storage, fine-grained policies, audit trails, versioning, **dynamic secrets** (short-lived credentials issued on demand), automatic rotation, and CI/CD integration.

### 5. Safe password storage

- Never plaintext.
- A **slow hash** — Argon2 (OWASP's first choice), scrypt, bcrypt, or PBKDF2 — not MD5 or SHA-1.
- A unique per-user **salt** before hashing (`hash(password + salt)`), stored alongside the hash, defeating rainbow tables. See [salting](/citadel/interview/salting).
- Optionally a server-side **pepper** (`hash(password + salt + pepper)`) held outside the database.

### 6. Mask, anonymise, pseudonymise

For test and analytics environments, replace real values so the data is useful without being exposing.

### 7. Input validation and output encoding

Validate every incoming value to block injection (SQLi, XSS, command injection). Encode data before rendering it in a UI to stop XSS from stealing session cookies.

### 8. Data loss prevention

DLP tooling to monitor and block unauthorised movement of sensitive data — in use on endpoints, in motion on the network, at rest in storage.

### 9. Secure development and API security

Security through the whole lifecycle (DevSecOps): security-focused code review; SAST, DAST, and IAST testing; and hardened APIs — authentication, authorization, rate limiting — since APIs are the usual door to sensitive data. See [securing a web API](/citadel/interview/secure-web-api).

### 10. Logging, monitoring, auditing

Keep secure audit trails of all access to sensitive data. Monitor for suspicious access in real time. Review the logs.

### 11. Compliance

Know which regimes apply — GDPR, CCPA, DPDPA, HIPAA, PCI DSS — and meet their mandated controls; the penalties for not doing so are the point.

### 12. Secure deletion

When data is no longer needed, destroy it so it can't be recovered, digital media and physical storage alike.

## Key takeaways

- Sensitive data spans PII, financial, health, secrets, IP, and confidential business data; a breach carries financial, legal, and trust costs.
- No control stands alone — layer classification and minimisation, least-privilege access, encryption in transit and at rest, dedicated secret management, and slow salted password hashing.
- Back the technical controls with input/output validation, DLP, secure development, and continuous logging and monitoring.
- Compliance with GDPR, DPDPA, HIPAA, and PCI DSS is mandatory where it applies, and prescribes specific measures.
