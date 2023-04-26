---
title: Defensive Security - Detection, Forensics, and Incident Response
description: The other side of the fence. Collect telemetry from hosts and network, write detections against the MITRE ATT&CK matrix, and when something fires, run the incident-response lifecycle - identify, contain, eradicate, recover - with digital forensics filling in what actually happened.
date: 2023-04-26
draft: false
slug: /cyber-security/blue-team
tags:
  - Security
  - Blue Team
  - Forensics
---

Offensive security finds the holes. Defensive security — the blue team — assumes some of them will be used and builds the capability to *notice*, *understand*, and *evict* an intruder. It's a different discipline: instead of one clever exploit chain, it's telemetry pipelines, detection rules that balance coverage against false positives, and a rehearsed process for the day an alert is real.

This post is the shape of that work: what to collect, how to turn it into detections, the incident-response lifecycle, and where [digital forensics](/citadel/cyber-security/digital-forensics) and [malware analysis](/citadel/cyber-security/malware-analysis) fit.

## Telemetry: what to collect

You can only detect what you have data on. The sources:

- **Endpoint** — an EDR agent (CrowdStrike, Defender for Endpoint, SentinelOne) or, at minimum, **Sysmon** on Windows and **auditd** / eBPF-based tools on Linux. The high-value events: process creation with full command line and parent (Sysmon EID 1 / Windows 4688), network connections by process (EID 3), image/DLL loads (EID 7), file creation (EID 11), registry modification (EID 12/13), and — critically — **handle opens to `lsass.exe`** (EID 10), one of the strongest single signals of credential theft.
- **Windows event logs** — 4624/4625 (logon success/failure) with logon type, 4672 (privileged logon), 4698 (scheduled task created), 7045 (service installed), 4104 (PowerShell script block logging — turn it on), 4662 (directory-service access, for DCSync), 1102 (log cleared).
- **Network** — NetFlow/IPFIX for connection metadata at scale, full packet capture at key chokepoints, DNS query logs (tunneling, C2 domains), proxy logs, and Zeek for protocol-level records.
- **Identity** — authentication logs from the IdP (Okta, Entra ID, AD), MFA prompts, conditional-access decisions, OAuth grants.
- **Cloud** — CloudTrail / Azure Activity / GCP audit logs (API calls), and the control-plane events that matter (`CreateAccessKey`, `AttachUserPolicy`, IMDS access).

These flow into a **SIEM** (Splunk, Elastic, Sentinel, Chronicle) — a store that normalizes, indexes, and correlates them and runs detection queries.

## Detection engineering

Turning telemetry into alerts is its own craft, with a spectrum from precise to broad:

- **Signatures / IOCs** — match known-bad: a file hash, a C2 IP, a domain. Precise, zero false positives, but trivially evaded (recompile, rotate infrastructure) — the bottom of the **Pyramid of Pain** (David Bianco): hashes and IPs are cheap for an attacker to change; tools, and especially *TTPs* — the behaviours — are expensive.
- **Behavioural / TTP detections** — match *how* attackers operate regardless of tooling: "`rundll32` with no command-line arguments," "Office spawning `powershell`," "a service binary in a user-writable path," "`lsass` accessed by a non-system process," "4624 type-3 from one account to twenty hosts in five minutes." These survive an attacker swapping tools, which is why detection engineering aims here.
- **Anomaly detection** — statistical/ML baselines ("this user never logs in from that country," "this host never talks to that subnet"). Higher false positives; useful as a lead, not an alert.

Detections are written as **Sigma** rules (a vendor-neutral format that compiles to Splunk SPL, Elastic queries, etc.) and mapped to the **MITRE ATT&CK** matrix — a catalog of tactics (the *why*: initial access, persistence, credential access, lateral movement, exfiltration) and techniques (the *how*, e.g. T1003 OS Credential Dumping, T1053.005 Scheduled Task). Mapping every detection to ATT&CK shows your **coverage** — which techniques you'd catch and which are blind spots. `chainsaw` and `Hayabusa` run Sigma rules over EVTX files for fast triage.

## Threat hunting

Detections catch what you thought to look for. **Threat hunting** is proactively searching for what you didn't — form a hypothesis ("an attacker using WMI for persistence would leave `__EventFilter` entries"), query for the evidence, and either find something or turn the finding into a new detection. Hunts are informed by threat intelligence and by ATT&CK coverage gaps.

## Threat intelligence

Context about adversaries: **IOCs** (short-lived), **TTPs** (durable), campaign and actor attribution, and the **Diamond Model** (adversary, capability, infrastructure, victim) for organizing what you know. Shared via **STIX/TAXII** feeds. The value is prioritization — knowing which techniques a group targeting your sector uses tells you where to spend detection effort.

## The incident-response lifecycle

When an alert is real, the process (NIST 800-61 / the SANS **PICERL** mnemonic):

1. **Preparation** — done in advance: the IR plan, the on-call rota, the tooling (a forensics jump kit, out-of-band comms), tabletop exercises, and legal/PR/exec contacts.
2. **Identification** — confirm it's an incident, scope it (which hosts, accounts, data), assign severity. Build a timeline from the telemetry.
3. **Containment** — stop the spread without tipping off the attacker prematurely or destroying evidence. Short-term (isolate a host at the network level, disable an account) then long-term (rebuild, block C2, rotate credentials). Snapshot volatile data (memory, network state) *before* pulling a machine offline.
4. **Eradication** — remove the foothold: delete persistence, close the initial vector, patch, reset every credential the attacker could have touched (assume everything on a compromised host is compromised).
5. **Recovery** — restore systems from known-good, monitor closely for reinfection, phase back to normal operations.
6. **Lessons learned** — a blameless post-incident review: what worked, what didn't, what detection or control would have caught it earlier. This feeds back into preparation.

## Forensics and malware analysis in support

- **Digital forensics** — sound acquisition (order of volatility: memory before disk), then reconstructing what happened from filesystem artifacts (MFT, USN journal), execution evidence (Prefetch, Amcache, Shimcache), registry, browser history, and memory (Volatility). Covered in [digital forensics](/citadel/cyber-security/digital-forensics).
- **Malware analysis** — pull apart the sample found on a host to learn its capabilities, its C2, its persistence, and the IOCs to sweep for across the fleet. Covered in [malware analysis](/citadel/cyber-security/malware-analysis).
- The two feed the timeline, which drives containment scope.

## Metrics, purple teaming, validation

- **MTTD / MTTR** — mean time to detect and to respond; the numbers leadership tracks.
- **Purple teaming** — the red team executes specific ATT&CK techniques while the blue team watches, so you learn *exactly* which are detected and which aren't, then fix the gaps. **Atomic Red Team** is a library of small, safe technique tests for this; **Caldera** automates adversary emulation.
- **Detection-as-code** — Sigma rules in version control, tested in CI against sample logs, deployed like software.

## The one idea to keep

Defensive security is a pipeline: collect the telemetry that matters (process creation with parent and command line, `lsass` access, authentication events, DNS, cloud API calls), write detections that target *behaviours* rather than hashes and IPs (because those are cheap for an attacker to change), and map them to MITRE ATT&CK so you know your coverage. When one fires, run the incident-response lifecycle — identify and scope, contain without destroying evidence, eradicate every credential the attacker could have touched, recover from known-good, and feed the lessons back into preparation.
