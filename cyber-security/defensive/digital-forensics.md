---
title: Digital Forensics - Disk, Memory, and Network Artifacts
description: The technical layer under incident response. Sound acquisition in order of volatility, the Windows filesystem and registry artifacts that prove a program ran or persistence was installed, memory forensics with Volatility, and reconstructing an intrusion from a packet capture.
date: 2023-04-27
draft: false
slug: /cyber-security/digital-forensics
tags:
  - Security
  - Forensics
  - Blue Team
---

When [incident response](/citadel/cyber-security/blue-team) needs to know *what actually happened* — which files the attacker touched, whether that program ever executed, when persistence was installed, what left over the network — the answer comes from forensic artifacts. Operating systems record far more about their own activity than users realize, in dozens of places, and a large part of forensics is knowing which artifact answers which question and how to read it.

This post is a tour of those artifacts across the three domains — disk, memory, network — plus the acquisition principles that keep the evidence sound.

## Acquisition principles

- **Order of volatility.** Collect the most ephemeral data first: CPU registers and cache (rarely practical), then **memory**, then network connections and ARP/routing state, then running processes, then disk, then remote logs, then physical config. Pull a machine's power and you lose everything in RAM — including injected code, decryption keys, and clipboard.
- **Write blockers** for disk imaging — a hardware or software device that permits reads but blocks writes, so the act of imaging doesn't alter the source.
- **Imaging** — a bit-for-bit copy (`dd`, `ewfacquire` / E01 format with compression and metadata, FTK Imager). Never analyze the original; work on verified copies.
- **Hashing and chain of custody** — hash the image (MD5 *and* SHA-256) immediately; any later verification re-hashes and compares. Document who had the evidence, when, and why — because it may end up in court.
- **The forensic timeline** — the deliverable that ties it together: a single chronological sequence of events from all sources, which is how you establish the intrusion's story.

## Windows filesystem artifacts (NTFS)

- **$MFT** (Master File Table) — one record per file, with four timestamps (**MACB**: Modified, Accessed, Changed/MFT-modified, Born/created) in *two* attributes (`$STANDARD_INFORMATION`, which tools and malware can alter — **timestomping** — and `$FILE_NAME`, which is harder to forge and usually reflects the true creation). A mismatch between the two is a timestomping indicator. Small files' contents live *resident* in the MFT record itself.
- **$UsnJrnl** ($Extend\$UsnJrnl:$J) — the change journal: a rolling log of every file create, delete, rename, and data change with timestamps. Recovers evidence of files that were created and then deleted.
- **$LogFile** — NTFS transaction log, finer-grained and shorter-lived than the USN journal.
- **Volume Shadow Copies** — point-in-time snapshots; a file deleted now may still exist in a shadow copy from last week.
- **Alternate data streams** — data hidden in a named stream on a file (`file.txt:hidden`); a classic hiding spot.
- **$Recycle.Bin** — deleted files with `$I` (metadata: original path, deletion time) and `$R` (content) pairs.

## Windows execution and persistence artifacts

Evidence that a program *ran*:

- **Prefetch** (`C:\Windows\Prefetch\*.pf`) — created/updated when an executable runs; records the executable name, run count, last (and up to 8) run times, and files it loaded. Strong "this ran, this many times, at these times" evidence.
- **Amcache.hve** and **Shimcache** (AppCompatCache in the SYSTEM hive) — record executables that were present/run, with paths and (Amcache) SHA-1 hashes. Shimcache entries can exist without execution, so corroborate.
- **SRUM** (System Resource Usage Monitor) — per-process network bytes sent/received and execution time over ~30 days. Great for spotting exfiltration volume by process.
- **UserAssist**, **RunMRU**, **jump lists**, **LNK files** — GUI program launches and recently opened documents, per user.
- **Shellbags** — folder access via Explorer, including folders that no longer exist and network/removable paths.

Evidence of **persistence**:

- **Run/RunOnce** registry keys (HKLM and HKCU), **Scheduled Tasks** (`C:\Windows\System32\Tasks\*` XML + registry), **Services** (SYSTEM hive), **WMI event subscriptions** (`__EventFilter` / `CommandLineEventConsumer` in the `OBJECTS.DATA` repository — fileless, reboot-surviving, historically under-checked), Startup folders, and image-file-execution-options / debugger hijacks.

**Registry forensics** — the hives (SYSTEM, SOFTWARE, SECURITY, SAM, per-user NTUSER.DAT) hold configuration, MRU lists, mounted-device history, network history, and USB-device history. **RegRipper** and Eric Zimmerman's `Registry Explorer` parse them.

## Memory forensics

A RAM capture (`winpmem`, `DumpIt`, LiME on Linux, or from a VM's paused state / snapshot) contains what disk doesn't: injected code, unpacked malware, network connections, command history, decryption keys, and clipboard.

**Volatility 3** is the standard analysis framework. The go-to plugins:

- `windows.pslist` / `windows.psscan` / `windows.pstree` — running (and hidden/terminated) processes and their parentage. A `pstree` showing `winword.exe → cmd.exe → powershell.exe` is initial access via a malicious document.
- `windows.cmdline` — the full command line of each process.
- `windows.netscan` / `windows.netstat` — network connections and owning process; find the C2 channel.
- `windows.malfind` — memory regions that are private, executable, and not backed by a file — i.e. injected code.
- `windows.dlllist` / `windows.handles` / `windows.registry.*` — loaded modules, open handles (files, keys, mutexes — malware often uses a named mutex as a "already infected" check), in-memory registry.
- `windows.dumpfiles` / `windows.memmap` — carve files and process memory out for further analysis.

On a Linux dump the equivalents (`linux.pslist`, `linux.bash`, `linux.check_syscall` for rootkit hooks) apply; there's also a technique of carving the **page cache** to recover a file that was on disk but has since been wiped or truncated.

## Network forensics

From a packet capture (`.pcap`):

- **Triage** — `tshark -q -z io,phs` (protocol hierarchy — what's in this capture), `-z conv,ip` (top talkers), `-z http,tree`.
- **Follow streams** — `tshark -z follow,tcp,ascii,<n>` reassembles a conversation; export transferred files with `--export-objects http,<dir>`.
- **Cleartext credentials** — FTP/POP3/SMTP/Telnet/HTTP-Basic auth in the clear (`-Y 'ftp.request.command == "PASS"'`).
- **C2 identification** — beaconing (regular-interval connections), suspicious User-Agents, self-signed certs, JA3/JA3S TLS fingerprints, and **covert channels**: DNS tunneling (long, high-entropy subdomain labels; TXT-record abuse), ICMP with non-standard payloads, data hidden in HTTP headers or timing.
- **Decrypting C2** — if you also have a memory dump, recover the session key (an RSA private key, an AES key, an XOR key from strings) and decrypt captured frames to read the operator's tasking.

## Documents, macros, and stager analysis

Malicious documents are a common initial vector: a `.doc` that's actually a ZIP (DOCX) with an external relationship fetching a payload (**Follina**/MSDT), or a VBA macro with `AutoOpen` that stages PowerShell. `oletools` (`olevba`, `oleid`), `unzip -l` on the doc, and `rg 'ms-msdt|Invoke-Expression'` triage these; the dropped file's name and path (`C:\Windows\Tasks\`, `%TEMP%`) become IOCs.

## Anti-forensics and steganography

Attackers fight back: **timestomping** (see the MFT dual-timestamp check above), log clearing (Windows event 1102, `wevtutil cl`; gaps in continuous logs are themselves evidence), secure deletion, and living entirely in memory. **Steganography** hides data in carriers — LSB embedding in images (`zsteg`, `stegoveritas`), data appended after a file's logical end (`binwalk`), whitespace/zero-width characters in text, and polyglot files valid as two formats at once. Spectrogram analysis for audio.

## Tooling

**Autopsy** / The Sleuth Kit (disk analysis GUI/CLI), **Eric Zimmerman's tools** (MFTECmd, PECmd, AmcacheParser, Registry Explorer, Timeline Explorer), **plaso/log2timeline** (super-timeline from everything), **KAPE** (fast targeted collection), **Volatility 3** (memory), **Wireshark/tshark/Zeek** (network), **chainsaw**/**Hayabusa** (EVTX + Sigma).

## The one idea to keep

Operating systems record far more about their own activity than users think, and forensics is knowing which artifact answers which question: Prefetch and Amcache prove a program ran and when; the MFT's two timestamp sets expose timestomping; the USN journal recovers deleted files; Run keys, scheduled tasks, and WMI subscriptions hold persistence; and a memory dump analyzed with Volatility shows injected code (`malfind`), the C2 connection (`netscan`), and the process tree that led to it. Acquire in order of volatility, hash everything, and assemble one timeline.
