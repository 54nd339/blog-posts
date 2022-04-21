---
title: The Relational and E/R Models - Design and Indexing
description: From a conceptual blueprint to fast physical access - the relational model and its integrity constraints, E/R and EER diagrams with the rules for converting them to tables, and the index structures (primary, secondary, multilevel, B+-tree, hashing) that turn a table scan into a few disk reads.
date: 2022-04-21
draft: false
slug: /dbms/relational-data-model
tags:
  - Databases
  - Relational Model
  - Indexing
---

Designing a database happens twice. First the **conceptual** design — deciding what things exist, how they relate, and what rules constrain them, usually drawn as an E/R diagram and mapped to relational tables. Then the **physical** design — deciding how those tables sit on disk and what indexes speed up the queries you actually run.

This post covers both. The relational model and its integrity constraints, E/R and Extended E/R notation with a systematic recipe for turning a diagram into a schema, and then indexing: the single-level index types, why they don't survive updates, and the B+-tree and hash structures that do.

## The relational model

A **relation** is a table. Its columns are **attributes** (each with a unique name and a **domain** — the set of permitted values); its rows are **tuples**. The **degree** is the number of attributes, the **cardinality** the number of tuples. Every cell value must be **atomic** — one indivisible value, no lists crammed into a cell. Row and column order carry no meaning.

Distinguish the **schema** (the design — `STUDENT(StudentID, Name, Major, GPA)`) from an **instance** (the actual set of tuples right now, which changes constantly while the schema stays put).

### Integrity constraints

Rules the data must always satisfy:

- **Domain constraints** — each value belongs to its attribute's domain (enforced by data types).
- **Entity integrity** — every table has a **primary key**, and its attributes are `NOT NULL`.
- **Referential integrity** — a **foreign key**'s value must match some primary-key value in the referenced table, or be NULL. You can't reference a row that doesn't exist.
- **Operational (business) rules** — real-world limits like "salary ≤ manager's salary" or "class size ≤ 50".

### Codd's rules

E.F. Codd defined 13 rules (0–12) for what makes a DBMS genuinely relational. Condensed: the system is managed entirely relationally (0); all data and metadata are **values in tables** (1); every value is reachable by table + primary key + column name (2); NULLs are handled uniformly regardless of type (3); the catalog is itself relational and queryable (4); there is one **comprehensive sublanguage** for definition, manipulation, constraints, authorization, and transactions (5); all theoretically updatable views are updatable (6); insert/update/delete work **set-at-a-time** (7); applications are insulated from storage changes — **physical data independence** (8); and from non-destructive schema changes — **logical data independence** (9); integrity constraints live in the catalog, not application code (10); data location is transparent — **distribution independence** (11); and a low-level interface can't be used to **bypass** the integrity rules (12).

## The E/R model

Peter Chen's **Entity-Relationship** model is a diagram language for conceptual design, later mapped to tables.

**Entities** (rectangles) are the things you store data about; an **entity set** is all entities of one type. **Attributes** (ovals) describe them:

- **simple** (atomic) vs **composite** (`address` = street + city + zip);
- **single-valued** vs **multi-valued** (`phone_numbers`, drawn as a double oval);
- **stored** vs **derived** (`age` computed from `date_of_birth`, drawn as a dashed oval);
- **NULL** for absent values.

**Relationships** (diamonds, named with verbs) associate entity sets. Their **degree** is the number of participants (binary, ternary, …). A **recursive** relationship links an entity set to itself (`EMPLOYEE` *supervises* `EMPLOYEE`), with **role names** ("supervisor", "subordinate") on the lines. **Descriptive attributes** hang off the diamond (`enrollment_date` on *Enrolls*).

**Keys**: a **superkey** is any attribute set that uniquely identifies an entity; a **candidate key** is a minimal superkey; the **primary key** is the chosen candidate key (underlined); the others are **alternate keys**.

A **weak entity set** (double rectangle) can't be identified by its own attributes — it depends on an owner via an **identifying relationship** (double diamond), and its **partial key** (discriminator, dashed underline) is unique only *within* one owner.

**Cardinality and participation**: mapping ratios (**1:1**, **1:M**, **M:N**) on the relationship lines, plus **total participation** (every entity must participate — double line) or **partial** (single line). The **(l..h)** notation on an edge gives minimum `l` (0 partial, 1 total) and maximum `h` (1 or N).

### Design choices

- **Entity set vs attribute** — if a concept has its own attributes or relationships, make it an entity (`Manufacturer` with an address, not just a string on `Product`).
- **Entity set vs relationship set** — an action/association is a relationship; an object with identity is an entity.
- **Binary vs n-ary** — prefer binary; any n-ary relationship among A, B, C can be replaced by a new entity E and three binary 1:M relationships.
- **Where relationship attributes go on conversion** — M:N: the new relationship table. 1:M: the "M"-side table. 1:1: either table.

### Extended E/R

- **Specialization** (top-down) carves subclasses out of a superclass; **generalization** (bottom-up) merges entity sets into one. Both draw an **ISA** triangle; subclasses **inherit** the superclass's attributes and relationships.
- Constraints: membership is **condition-defined** or **user-defined**; subclasses are **disjoint** (`d`, an instance is in at most one) or **overlapping** (default); the specialization is **total** (every superclass instance is in some subclass — double line) or **partial** (default).
- **Aggregation** treats a whole relationship-plus-entities as a single higher-level entity so it can participate in *another* relationship (drawn as a dashed box).

### E/R to relational schema

| E/R construct | Becomes |
| --- | --- |
| Strong entity set | a table with the same attributes; choose a PK |
| Weak entity set | a table + owner's PK as FK; PK = owner PK + discriminator |
| Binary M:N | a new table with both PKs as FKs (combined PK) + descriptive attributes |
| Binary 1:M | FK (of the "1" side) added to the "M"-side table + descriptive attributes there |
| Binary 1:1 | FK added to one table (prefer the total-participation side) |
| Recursive | same table: M:N → new table; 1:M → a self-referencing FK (`SupervisorID` → `EmpID`) |
| Composite attribute | flatten to one column per component (`address_street`, `address_city`, …) |
| Multi-valued attribute | a new table: entity PK + the attribute, combined PK |
| ISA hierarchy | (a) a table per subclass with superclass attrs duplicated; (b) a superclass table + a table per subclass keyed by the superclass PK; (c) one table with all attributes + a `type` discriminator (many NULLs) |
| Aggregation | a table for the aggregated relationship |

## File organisation

How records sit in disk blocks: **heap** (unordered — fast insert, slow search), **sequential** (ordered by a key — good for range scans, slow insert), **hashed** (placed by `h(key)` — fast exact-match), and clustered **B+-tree** files (organisation and index in one). The goal is always to minimise the disk blocks a query must read.

## Indexing

An **index** is a smaller, separate structure that provides a secondary access path on a **search key**, letting the DBMS jump to the right blocks instead of scanning the whole table.

### Single-level indexes

- **Primary index** — on the primary key of a file *physically ordered* by that key. One entry **per data block** (the block's first key + a block pointer), so it's **sparse**. Compact, binary-searchable, but only possible on a key-ordered file, and insertions into that file are painful.
- **Clustering index** — on a *non-key* field the file is physically ordered by. One entry **per distinct value** (the value + a pointer to the first block containing it), also **sparse**. Retrieves all records for a value efficiently since they're contiguous.
- **Secondary index** — on any field, where the file is *not* ordered by it. One entry **per record** (value + pointer), so it's **dense**, and duplicate keys are allowed. You can have many secondary indexes; each costs storage and maintenance.

**Dense** = an entry per record (secondary). **Sparse** = an entry per block (primary, clustering).

### Multilevel indexes

If a single-level index spans many blocks, searching it is still slow — so index the index. Build a sparse level-2 index over the level-1 blocks, a level-3 over that, until the top level fits one block. A lookup is then `levels + 1` block accesses. But a multilevel index over an *ordered file* still breaks down on insert and delete, which is why databases use dynamic trees instead ([the calculations post](/citadel/dbms/calcs) works the block-access numbers).

### B-trees and B+-trees

Both are balanced multi-way search trees sized so a node fills a disk block; all leaves sit at the same depth, and high fanout keeps the tree shallow (few disk reads). The [data-structures treatment](/citadel/data-structures/b-tree) has the full algorithms.

A **B-tree** stores `<pointer, key>` in internal nodes and can hold data pointers in internal *and* leaf nodes. A search can end at an internal node. Insertion adds to a leaf and, on overflow, splits and **promotes** the middle key upward; deletion borrows from a sibling or merges. Both can propagate to the root.

A **B+-tree** — the DBMS standard — puts data pointers **only in the leaves**; internal nodes hold only keys and tree pointers as a routing map (`Ki` is the maximum key in subtree `Pi`), and keys may be duplicated there. Leaves are chained with `Pnext` pointers. Insertion splits a leaf and **copies** the middle key up (rather than moving it) and links the new leaf; deletion redistributes with a sibling (updating the parent key) or merges.

| | B-tree | B+-tree |
| --- | --- | --- |
| Data pointers | internal and leaf nodes | leaves only |
| Search | may stop at an internal node | always reaches a leaf |
| Range queries | tree traversal | scan linked leaves — much faster |
| Insertion / deletion | promote/move logic | simpler copy-up / redistribute |
| Usage | rare now | standard for DBMS indexing |

### Hash indexes

A hash function `h(K)` computes the bucket (disk block) for key `K` — ideally one disk access to fetch a record. Problems: **collisions** need overflow pages, and **static hashing** (a fixed bucket count) degrades badly as the file grows or shrinks. Two dynamic schemes fix that:

- **Extendible hashing** — a **directory** of pointers to buckets, sized by a global depth `d`. Each bucket has a local depth `d'`. On overflow: if `d' < d`, split the bucket, redistribute, increment `d'`; if `d' = d`, first double the directory, then split.
- **Linear hashing** — no directory. A family of hash functions and a **split pointer** that advances one bucket at a time, splitting buckets in order regardless of which one overflowed, using overflow pages in between.

Hashing is excellent for exact-match lookups and useless for range queries — hashed values aren't ordered.

## The one idea to keep

Conceptual design decides *what's true* about the data — the entities, the relationships, the constraints — and the E/R-to-relational recipe turns that into tables mechanically. Physical design decides *how fast* you can get it, and the whole story there is the same trade: an index makes reads cheap and writes (and storage) more expensive, so you index the columns your queries filter and join on, and no more. B+-trees win the default because they serve both point and range queries while staying balanced under updates.
