---
title: Block, File, and Object Storage - The Three Models
description: "The same bytes, exposed three ways: raw blocks a server formats itself, a shared hierarchy of files and folders, or immutable objects behind an HTTP API. How each works and which workloads it suits."
date: 2024-08-07
draft: false
slug: /interview/storage-systems
tags:
  - System Design
  - Interview Prep
  - Storage
---

Persistent storage comes in three architectural models, and they differ not in the bytes but in *how a system addresses them*: as raw fixed-size blocks, as a hierarchy of named files, or as opaque objects behind an API. Each trades performance, sharing, scale, and cost differently. This covers all three and where each fits, and it sits below the [memory hierarchy](/citadel/interview/computer-memories) at the persistent end.

## Block storage

The raw layer. Storage is presented to a server as fixed-size **blocks** grouped into **volumes**, and the server's OS — or a database engine, or a hypervisor — controls those blocks directly: format a filesystem on them, or manage them raw for performance. The oldest model, and the most flexible.

Blocks can be physically attached (an SSD in the machine) or network-attached over **iSCSI** or **Fibre Channel** — either way the server still sees raw blocks. A block volume is normally owned by **one server at a time**; it is not built for simultaneous shared access.

Good for: virtual machine disks, and databases that want direct control of layout for performance.

## File storage

Built *on top of* block storage, adding the abstraction everyone knows: **files** in **hierarchical directories**, addressed by path (`/home/user/report.docx`). The filesystem translates file operations into block operations so callers never touch blocks or format volumes.

Its defining strength is **sharing**: many servers and users can mount the same file storage at once over **SMB/CIFS** (Windows) or **NFS** (Unix/Linux).

Good for: general-purpose filesystem access and shared files within an organisation.

## Object storage

The newest model, and a deliberate trade — give up performance and in-place mutation for **durability, scale, and low cost**. Data is stored as **objects** in a **flat namespace** (no directories); each object is its payload plus arbitrary **metadata** plus a globally unique ID. Access is a **RESTful HTTP API**, so it is reachable from anywhere with permission and natural for web-native apps. Objects are effectively immutable — you version or replace, you do not edit in place.

Good for: unstructured and binary data, backups, archives, data lakes, static site hosting, cloud-native storage. S3, Google Cloud Storage, Azure Blob.

## Side by side

| | Block | File | Object |
| --- | --- | --- | --- |
| In-place update | yes | yes | no (versioning instead) |
| Cost | high | medium–high | low |
| Performance | medium–very high (SSD) | medium–high | low–medium |
| Access | iSCSI / Fibre Channel | SMB/CIFS, NFS | HTTP REST API |
| Sharing | one server at a time | many servers | many clients |
| Scalability | medium | high | vast |
| Fits | VM disks, databases | shared filesystems | backups, archives, data lakes |

## The model follows the access pattern

The choice is really about how the data is reached: something that needs byte-level control and one owner wants **block**; something many machines edit together as files wants **file**; something written once, read occasionally, and grown without limit wants **object**. Most cloud systems use all three — block for the database's disk, object for its backups and the data lake, file where a legacy component expects a mount — because they are not competitors so much as different answers to "who addresses these bytes, and how".
