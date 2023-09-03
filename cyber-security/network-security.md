---
title: Network Security - Authentication, Key Management, and Defences
description: "How entities prove their identity - passwords and MFA, biometrics, challenge-response, Kerberos, PKI - and the framework for defending a network: the threat landscape, the X.800 security services and mechanisms, defence-in-depth and zero trust, firewalls, and intrusion detection."
date: 2023-09-03
draft: false
slug: /cyber-security/network-security
tags:
  - Security
  - Network Security
---

Before two parties can communicate securely, each has to be sure who the other is, and the keys that protect the channel have to be managed through their whole lifecycle. This post covers **entity authentication** and **key management**, then widens to **network security** as a discipline: the threats, the services and mechanisms that counter them, the architectural models, and the perimeter tools — firewalls and intrusion detection.

The cryptographic protocols that secure traffic — IPsec, TLS, SSH, PGP — are covered in [the networks category's security post](/citadel/computer-networks/network-security).

## Authenticating an entity

**Entity authentication** verifies that a user, device, or process is what it claims to be.

### Passwords

Still the most common method. Systems store **salted, iteratively hashed** passwords, never the passwords themselves — a **salt** is a random value mixed in before hashing so that identical passwords hash differently and precomputed **rainbow tables** are useless. Passwords are vulnerable to weak choices, reuse, phishing, keylogging, and brute-force or dictionary attacks. Mitigations: strong policies, account lockout, and above all **multi-factor authentication** — combining two or more of *something you know* (password), *something you have* (a token or phone app), and *something you are* (biometrics).

### Biometrics

Authentication from biological or behavioural traits. **Physiological**: fingerprint, face, iris, retina, hand geometry, DNA. **Behavioural**: voice, signature dynamics, keystroke timing. The process is **enrolment** (capture and store a template) then **verification** (compare a live sample). Advantages: unique to the person, nothing to forget or lose. Challenges: balancing the **false acceptance rate** against the **false rejection rate**, cost, privacy of stored templates, and **irrevocability** — a compromised biometric cannot be reissued like a password.

### Challenge-response

The verifier sends a random challenge (a nonce); the claimant performs a cryptographic operation on it with a shared secret and returns the result; the verifier checks it. The secret is never transmitted, defeating eavesdroppers, and a fresh challenge each time defeats replay.

### Zero-knowledge proofs

A prover convinces a verifier that a statement is true — that they know a secret — while revealing nothing beyond the statement's truth. The properties are **completeness** (a true statement is provable), **soundness** (a false one is not, except negligibly), and **zero-knowledge** (the verifier learns nothing else). The "Ali Baba cave" is the standard intuition. ZKPs support privacy-preserving authentication and anonymous credentials, at a computational cost. The advanced forms — SNARKs, STARKs — are in [advanced cryptography](/citadel/cyber-security/advanced).

### Kerberos

A network authentication protocol using a trusted third party, the **Key Distribution Center**, with two parts: an **Authentication Server** that verifies users at login and a **Ticket-Granting Server** that issues tickets for specific services.

1. The client requests a **Ticket-Granting Ticket** from the AS using the user's credentials.
2. The AS authenticates the user and returns an encrypted TGT plus a session key.
3. The client presents the TGT to the TGS to request a **service ticket** for an application server.
4. The TGS validates the TGT and returns the service ticket plus a client-server session key.
5. The client presents the service ticket to the application server.

Kerberos uses symmetric cryptography, provides single sign-on, and resists eavesdropping and replay. It needs a secure, available KDC and synchronised clocks.

### Public Key Infrastructure

PKI is the framework for issuing, managing, and revoking **digital certificates**, which bind a public key to an entity. Components:

- **Digital certificate (X.509)** — contains a public key, the subject's identity, the issuer, a validity period, and the issuer's signature.
- **Certificate Authority (CA)** — issues and signs certificates. Root CAs are implicitly trusted by browsers and operating systems; subordinate CAs form a **chain of trust**.
- **Registration Authority (RA)** — verifies the identity of entities requesting certificates, on the CA's behalf.
- **CRL / OCSP** — a certificate revocation list, or the Online Certificate Status Protocol, to check whether a certificate has been revoked before expiry.
- **Certificate repository** — a store for certificates and CRLs.

PKI is what makes [HTTPS/TLS](/citadel/interview/https), S/MIME, and code signing scale. See [asymmetric key cryptography](/citadel/cyber-security/asymmetric-key) for how the signatures underneath work.

### CAPTCHA

A challenge-response test to tell humans from bots — distorted text, image selection, audio, simple arithmetic — used to stop automated abuse like spam account creation and poll skewing. It authenticates a *category* (human), not a specific identity.

## The network threat landscape

Networks face: eavesdropping and packet sniffing, data modification in transit, denial of service (and distributed DoS), spoofing (IP, MAC, email), man-in-the-middle, malware, insider threats, zero-day exploits, social engineering and phishing, and reconnaissance such as port scanning.

**TCP/IP-specific weaknesses** stem from a design era when security was not a concern:

- Core protocols (IP, TCP, UDP, DNS, HTTP) do not encrypt by default.
- IP does not strongly verify source addresses, enabling **IP spoofing** (see [the network layer](/citadel/computer-networks/newtwork-layer)).
- Predictable **TCP sequence numbers** in older stacks allow **session hijacking** (see [the transport layer](/citadel/computer-networks/transport-layer)).
- **Port scanning** discovers services — legitimate for administration, a first step for attackers.
- **Fragmentation attacks** exploit IP reassembly to evade security devices.

Higher-layer protocols (IPsec, TLS, SSH) and better implementations address most of these.

## Passive and active attacks

- **Passive attacks** observe without altering — packet sniffing, traffic analysis. Hard to detect; the defence is prevention (encryption).
- **Active attacks** modify or inject: **masquerade** (impersonation), **replay** (retransmitting captured data), **message modification**, **DoS**, **session hijacking**, **port scanning**, and **MAC flooding** — overwhelming a switch's address table so it fails open and broadcasts all traffic, letting the attacker sniff (see [the link layer](/citadel/computer-networks/datalink-layer)).

## Security services and mechanisms

The OSI security architecture (X.800) defines the **services** a system aims to provide:

| Service | Meaning |
| --- | --- |
| Confidentiality | Data not disclosed to the unauthorised |
| Authentication | The communicating entity, or the data's origin, is as claimed |
| Integrity | Data received is exactly as sent — no modification, insertion, deletion, or replay |
| Non-repudiation | Neither party can deny participating |
| Access control | Unauthorised use of a resource is prevented |
| Availability | Resources are usable on demand by authorised entities |

The **mechanisms** that implement them: encipherment, digital signatures, access-control mechanisms ([ACLs, RBAC](/citadel/interview/identity-management), capabilities), data-integrity mechanisms (hashes, MACs), authentication exchange, traffic padding (obscuring flow patterns), routing control, and notarisation (a trusted third party attesting to a data exchange).

## Security models and controls

- **Defence in depth (layered security)** — multiple independent controls, so bypassing one still leaves others in place.
- **Zero trust** — no implicit trust from network location; every user and device is verified for every access, granularly and dynamically. "Never trust, always verify."

Controls are categorised by when they act:

- **Preventive** — firewalls, encryption, strong authentication, access control, training.
- **Detective** — intrusion detection systems, logs, audits, honeypots.
- **Corrective** — incident response plans, backups, patching.

## Firewalls

A firewall filters traffic between networks by rule.

| Type | How it decides | Trade-off |
| --- | --- | --- |
| Packet-filtering | Inspects packet headers (IP, port) | Fast; no context |
| Stateful inspection | Tracks connection state | Smarter; more resource use |
| Proxy (application-level gateway) | Mediates traffic per application, can inspect content | Granular; slower |
| Next-generation (NGFW) | Adds deep packet inspection, intrusion prevention, application awareness, threat intelligence | Most capable; most complex |

See [firewalls](/citadel/interview/firewall) for placement and rule design.

## Intrusion detection and prevention

- **IDS** monitors network or host activity for malicious behaviour and reports it. **NIDS** watches the network, **HIDS** watches a host. Detection is **signature-based** (known attack patterns) or **anomaly-based** (deviation from a learned baseline).
- **IPS** is an IDS that can actively block or drop detected intrusions.

## Secure email

Standard email is plaintext. Two solutions add end-to-end encryption and authentication:

- **PGP** — symmetric encryption for the message body, public-key encryption for the session key, and digital signatures. Key validation uses a decentralised **web of trust**.
- **S/MIME** — an IETF standard using a hierarchical PKI for certificates, built into many mail clients.

## Malware

Software built to harm or exploit — viruses, worms, trojans, ransomware, spyware, adware, botnets. The taxonomy and how each exploits software flaws is in [software security](/citadel/cyber-security/info-security).

## The one idea to keep

Network security is layers of verification and containment. Authentication establishes who is at the other end — from a salted password hash up to a PKI certificate chain. The security services (confidentiality, integrity, authentication, non-repudiation, availability) name what you are protecting, mechanisms like encryption and signatures implement them, and defence in depth plus zero trust arrange them so that one failure is not a breach.
