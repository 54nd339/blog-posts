---
title: Software Security - Vulnerabilities, Exploits, and Defences
description: The recurring classes of software flaw - buffer overflows, race conditions, input-validation failures - how they become exploits and malware, the high-impact web vulnerabilities (SQL injection, XSS, CSRF, path traversal), reverse engineering and tamper resistance, and the role of penetration testing.
date: 2023-09-04
draft: false
slug: /cyber-security/info-security
tags:
  - Security
  - Network Security
---

Cryptography protects data in transit and at rest, but the software handling that data is itself built from code, and code has bugs. A large share of real-world compromise comes not from breaking a cipher but from a program that trusts input it should not, writes past the end of a buffer, or exposes a file it should keep private. This post covers the common flaw classes, how they become exploits, the specific web vulnerabilities worth knowing by name, and the proactive work — reverse engineering, tamper resistance, penetration testing — done to stay ahead of them.

## Common flaw classes

**Buffer overflows.** A program writes more data into a fixed-length buffer than it holds, and the excess spills into adjacent memory. At best this corrupts data or crashes; at worst it overwrites control information such as a function's return address on the stack, letting an attacker redirect execution to code they supplied. Mitigations: memory-safe languages, bounds checking, stack canaries, address space layout randomisation (ASLR), and data execution prevention (DEP). See [memory management](/citadel/operating-system/memory-management) for the stack layout involved.

**Race conditions.** The program's behaviour depends on the unpredictable timing of concurrent events. The classic pattern is **TOCTOU** (time-of-check to time-of-use): a program checks that a user may access a file, and before it opens the file an attacker swaps it for another. Mitigations: proper synchronisation (mutexes, semaphores), atomic operations, careful design of shared-resource access.

**Input validation errors.** The application does not properly check, sanitise, or filter input, so crafted input makes it misbehave. This umbrella covers SQL injection, cross-site scripting, command injection (input treated as OS commands), and format-string bugs (mishandled `%s`/`%x` specifiers leading to disclosure or code execution). Mitigations: strict validation of type, length, format, and range; output encoding; parameterised queries; least privilege.

**Others:** insecure deserialisation (deserialising untrusted data leading to remote code execution — the [Log4Shell / JNDI attack](/citadel/pentesting/log4j-jndi-attack) is a notable case), use-after-free (using a pointer after its memory is freed), and integer overflow or underflow (arithmetic producing a value the type cannot hold).

## From flaw to exploit to malware

An **exploit** is a piece of software, data, or a command sequence that takes advantage of a flaw to cause unintended behaviour. When an exploit exists before the developer knows of the flaw, it is a **zero-day**. Exploits are used to gain unauthorised access, escalate privileges, steal data, disrupt service, or install **malware**:

- **Virus** — attaches to a legitimate program and spreads when that program runs.
- **Worm** — self-replicates across networks by exploiting service or OS vulnerabilities, with no host program needed.
- **Trojan** — disguised as legitimate software; relies on the user running it, then abuses the permissions the user has.
- **Ransomware** — encrypts the victim's files or locks the system and demands payment; spreads via phishing or exploited vulnerabilities.
- **Spyware / adware** — covertly gathers information or shows ads, often via browser vulnerabilities or bundled installers.
- **Botnet** — a network of compromised machines ("bots") under remote control.

## High-impact web vulnerabilities

**SQL injection.** Malicious SQL is inserted into an input field of a data-driven application. If the app builds queries by concatenating unvalidated input, supplying `' OR '1'='1` into a login field can turn the query into one that always succeeds. Impact: unauthorised read, modification, or deletion of data, and sometimes command execution on the database server. Mitigation: **parameterised queries (prepared statements)** — see [query languages](/citadel/dbms/relational-query-lang) — plus input validation and least-privilege database accounts.

**Cross-site scripting (XSS).** An attacker injects a script into a trusted site that then runs in another user's browser.

- **Stored** — the script is saved on the server (a comment, a forum post) and served to everyone who views the page.
- **Reflected** — the script is in a URL or input that the server echoes back in its response; the victim must click a crafted link.
- **DOM-based** — the flaw is in client-side code, not the server.

Impact: stealing session cookies, defacement, redirection, keylogging. Mitigation: **output encoding** (so data is not interpreted as active content), input validation, and a **Content Security Policy**.

**Cross-site request forgery (CSRF).** A logged-in user is tricked into making a state-changing request to a site where they are authenticated — the browser automatically attaches the session cookie, so the request looks legitimate. Impact: unauthorised transfers, account changes. Mitigation: anti-CSRF tokens, checking the `Origin` header, and the `SameSite` cookie attribute.

**Path traversal.** Manipulating a file-path parameter with `../` sequences to reach files outside the web root — configuration, source code, credentials. Mitigation: validate and sanitise path parameters, run the server with least privilege, use a chroot jail.

The defensive side of these is covered in [securing a web API](/citadel/interview/secure-web-api).

## Reverse engineering

**Software reverse engineering** analyses a program to recover its structure and behaviour at a higher level of abstraction. Legitimate uses: understanding legacy code, interoperability, security auditing, malware analysis. Malicious uses: finding vulnerabilities, cracking licensing or DRM, stealing intellectual property, building targeted malware. Techniques: disassembly (machine code to assembly), decompilation (toward source), debugging, and static and dynamic analysis. It is a cat-and-mouse game between developers protecting code and analysts trying to understand it.

## Tamper resistance and DRM

**Software tamper resistance** makes modification hard, or at least detectable: code obfuscation, anti-debugging checks, self-modifying code, integrity checks (hashing code sections to detect changes), and guards. The goal is to raise the effort of reverse engineering; perfect resistance is not achievable.

**Digital rights management** is access-control technology limiting the use, copying, and distribution of copyrighted works, typically by encrypting the content and controlling decryption keys through a licensing system. It is controversial for its effect on fair use and interoperability, and determined attackers circumvent it — but it deters casual copying.

## Network operating system vulnerabilities

The software on routers, switches, firewalls, and servers is critical infrastructure. A compromised network OS can mean device takeover, traffic interception or redirection, denial of service for the whole network, or a foothold for further attacks. Vulnerabilities come from the same sources as any software — TCP/IP stack bugs, management interfaces (web, CLI, SNMP), routing protocols. Mitigation: keep firmware patched, disable unused services, use strong credentials and access-control lists, segment the network, and monitor.

## Penetration testing

Rather than waiting for attackers to find weaknesses, organisations look for them first.

**Ethical hacking** is using offensive skills defensively, with the owner's permission. A **penetration test** is a simulated attack on a system, network, or application to find exploitable vulnerabilities. Its phases:

1. **Reconnaissance** — gather information about the target.
2. **Scanning** — identify live hosts, open ports, running services.
3. **Vulnerability assessment** — find known weaknesses.
4. **Exploitation** — attempt to gain access or control.
5. **Post-exploitation** — assess what an attacker could do from inside.
6. **Reporting** — findings and remediation recommendations.

Engagements are **black-box** (no prior knowledge), **white-box** (full knowledge), or **grey-box** (partial). The benefits: a realistic picture of the security posture, prioritised remediation, validation of existing controls, and compliance evidence. The [pentesting](/citadel/pentesting/pentesting) category goes into methodology and [tooling](/citadel/pentesting/tools).

## The one idea to keep

Most compromise starts with software trusting something it should not — input, timing, a buffer length, a file path. The flaw classes are few and old, and the web vulnerabilities have names and standard fixes: parameterised queries for SQL injection, output encoding for XSS, tokens for CSRF. Penetration testing is the discipline of finding your instance of these before someone else does.
