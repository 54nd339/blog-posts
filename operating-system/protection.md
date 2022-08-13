---
title: Operating System Protection - Domains, Access Matrices, and Capabilities
description: "How the OS controls which process may do what to which resource - the least-privilege and need-to-know principles, protection domains and how they switch, the access matrix as the formal model, and its practical forms: access control lists, capability lists, and the lock-key hybrid, plus RBAC."
date: 2022-08-13
draft: false
slug: /operating-system/protection
tags:
  - Operating Systems
  - File Systems
  - Processes
---

Managing resources is only half the [OS](/citadel/operating-system/os)'s job; the other half is *controlling access* to them. On a machine with many users, concurrent processes, and network connections, something has to guarantee that one process cannot read another's memory and an ordinary user cannot delete system files. That is **protection** — the internal mechanisms, distinct from external [security](/citadel/interview/identity-management), that enforce who may do what.

This post covers the two guiding principles, the formal model (the access matrix), and the three ways real systems implement it.

## Two principles

- **Least privilege.** An entity — user, process, program — should run with the *minimum* privileges needed for its *current* task. If a program only reads a file, it should not have write permission. This bounds the damage from a bug or a breach. It can be enforced statically (fixed for a session) or dynamically (privileges acquired when needed, dropped after).
- **Need-to-know.** Closely related: a process should have access only to the resources its current task requires, granted by function rather than convenience.

The **grain** matters. **Rough-grained** control (UNIX user versus root) is simple but imprecise; **fine-grained** control (detailed ACLs, role-based access) adheres better to least privilege at the cost of complexity.

## Protection domains

A **domain** is a set of **access rights**, each a pair "permission to perform operation X on object Y" (read a file, write a segment, execute, delete). A process runs within a domain — the set of objects it can touch and the operations it may perform — and there must be a mechanism to **switch** domains when a task legitimately needs different rights.

Real domain systems:

- **UNIX / Linux** — a domain is a **user ID**. A process runs with the UID of whoever started it. Domain switching happens via the **setuid bit** on an executable: when set, the program runs with the *file owner's* UID (often root) for the duration, then reverts — temporary, targeted privilege escalation. `su` and `sudo` do the same explicitly, gated by a password and the `sudoers` configuration. Setuid programs must be written very carefully; they are a classic source of holes.
- **Multics rings** — hierarchical protection levels numbered 0 (most privileged) to N. Each segment has a ring number; a process in ring $i$ can freely access segments in rings $\ge i$, and reaching a more privileged ring requires a controlled transfer through a **gate**. More levels than the plain user/kernel split, but still rigid: a more privileged domain could access *everything* a less privileged one could, which violates strict need-to-know.

## The access matrix

The relationship between domains, objects, and rights is a matrix:

- **Rows** are domains $D_1, D_2, \dots$
- **Columns** are objects $O_1, O_2, \dots$ (files, devices, memory segments, even other domains).
- **Entry** $A[i, j]$ is the set of operations a process in $D_i$ may perform on $O_j$.

The matrix is **dynamic** — special rights let it change:

- **`owner`** — a domain owning an object can add or remove rights on it.
- **`copy`** (written `*`) — a domain can copy one of its rights on an object into another domain's entry.
- **`control`** — if $O_j$ is a domain object, `control` in $A[i, j]$ lets $D_i$ modify the rights in domain $D_j$.
- **`transfer`** — lets a process switch from $D_i$ to $D_j$.

The matrix cleanly separates **mechanism** (the OS providing and enforcing it) from **policy** (administrators deciding what rights go where).

## Implementing it

The matrix is sparse and potentially huge, so it is never stored directly. Three practical forms:

- **Global table** — every non-empty right as a `<domain, object, rights>` triple in one table. Simple, but slow to search and awkward to group rights, and the table itself grows large.
- **Access control lists** — store the matrix **by column**. Each *object* carries a list of `<domain, rights>` pairs. To check an access, look up the object's ACL for the requesting domain. This is how file systems work (NTFS, POSIX ACLs). Easy to manage per-object permissions; answering "what can this domain do everywhere?" means scanning many ACLs.
- **Capability lists** — store the matrix **by row**. Each *domain* holds a list of **capabilities**, each an unforgeable token `<object, rights>`. Possessing the capability *is* the permission. Capabilities must be protected by the OS (kept in kernel space or hardware-tagged) so processes cannot forge them. Easy to see what a domain can do; **revocation** is hard — capabilities can be copied and passed around, so finding and invalidating every copy is a problem.
- **Lock-key** — a hybrid. Each object has a list of **locks** (bit patterns); each domain has a list of **keys**. A domain accesses an object only if it holds a key matching one of the object's locks. Keys can be passed between domains, and revocation means removing a lock from the object.

Most systems combine approaches: ACLs for persistent objects, plus short-lived capabilities (file descriptors, handles) created on first access for speed within a session.

## Revocation

Taking a right back after granting it is surprisingly hard:

- **ACLs** — easy: delete the entry from the object's list; it takes effect on the next access.
- **Capabilities** — hard: copies may be anywhere. Solutions use indirection (capabilities point at a table entry that can be invalidated), back-pointers (track where capabilities went), or periodic mass invalidation.

## Beyond the matrix

- **Capability-based systems** — designed around capabilities as the *primary* protection model, not just an implementation detail. Often need hardware or language support. The focus is holding the authority, not checking identity against a list.
- **Role-based access control (RBAC)** — grant permissions to **roles**, assign users to roles, and let a user activate a role (often with authentication) to perform a privileged task. Administration is per-role rather than per-user, and it fits least privilege well. Solaris uses it heavily.
- **Language-based protection** — enforced by the compiler and runtime, not the OS: type safety in Java or C# prevents forging pointers or reaching arbitrary memory. Fine-grained within one address space, but it relies on the language implementation and does not manage OS-level resources.

## The one idea to keep

Protection is one abstract model — the domain-by-object access matrix — implemented by slicing it the cheap way for the question you ask most. Slice by object and you get ACLs, good for "who can touch this file". Slice by domain and you get capabilities, good for "what can this process do" but painful to revoke. Least privilege is the policy you are trying to express through whichever slice you pick.
