---
title: Database Practice - Index Sizing and Relational Algebra Queries
description: The concrete side of two DBMS topics - a worked method for estimating index block counts and how many disk reads a lookup costs for primary, secondary, and multilevel indexes, and a set of relational-algebra queries showing selection, join chains, and division for "for all" questions.
date: 2022-04-29
draft: false
slug: /dbms/calcs
tags:
  - Databases
  - Indexing
  - Query Languages
---

The [indexing](/citadel/dbms/relational-data-model) and [query-language](/citadel/dbms/relational-query-lang) posts are conceptual. This one is arithmetic and worked examples: given a file's parameters, how big is each kind of index and how many disk reads does a lookup take — and how do the relational-algebra operators combine to answer real queries, including the awkward "for all" ones.

## Sizing an index

Five parameters describe a data file and its index:

| Symbol | Meaning |
| --- | --- |
| `N` | number of records |
| `B` | block size (bytes) — one disk I/O reads one block |
| `R` | record size (bytes) |
| `K` | index key field size (bytes) |
| `P` | block/record pointer size (bytes) |

From these:

- **Data blocking factor** `bfr = floor(B / R)` — records per block.
- **Data blocks** `n = ceil(N / bfr)`.
- **Index entry size** `I = K + P`.
- **Index blocking factor** `bfri = floor(B / I)` — index entries per block.

### Block accesses per lookup

**Primary index** (sparse, one entry per data block). Index entries: `n`. Index blocks `m = ceil(n / bfri)`. A lookup binary-searches the index, then reads the data block:

$$\text{accesses} \approx \lceil \log_2 m \rceil + 1$$

versus `ceil(log2(n))` binary-searching the data file directly with no index.

**Secondary index** (dense, one entry per record). Index entries: `N`. Index blocks `m = ceil(N / bfri)`. Same lookup cost formula, `ceil(log2(m)) + 1` — but the alternative without an index is a linear scan of all `n` data blocks.

**Multilevel index**. Index the index. `m1` = first-level blocks (`m` from above). `m2 = ceil(m1 / bfri)`, `m3 = ceil(m2 / bfri)`, and so on until a level has a single block. With `k` levels:

$$\text{accesses} = k + 1$$

### Worked example

`N = 30,000`, `B = 1024`, `R = 100`, `K = 6`, `P = 9`.

- `bfr = floor(1024 / 100) = 10` records/block → `n = ceil(30000 / 10) = 3000` data blocks.
- `I = 6 + 9 = 15` → `bfri = floor(1024 / 15) = 68` entries/index block.

**Primary index**: `3000` entries → `m = ceil(3000 / 68) = 45` blocks → `ceil(log2 45) + 1 = 6 + 1 = 7` accesses (against `ceil(log2 3000) = 12` with no index).

**Secondary index**: `30,000` entries → `m = ceil(30000 / 68) = 442` blocks → `ceil(log2 442) + 1 = 9 + 1 = 10` accesses (against `3000` for a linear scan).

**Multilevel** (over the primary index): `m1 = 45`, `m2 = ceil(45 / 68) = 1`. So `k = 2` levels, `2 + 1 = 3` accesses — down from 7.

That last drop is the point: a second index level replaces a binary search over 45 blocks with a single direct lookup, and the pattern continues for larger files (which is exactly what a [B+-tree](/citadel/data-structures/b-tree) does dynamically).

## Relational algebra queries

Schema: `Sailors(sid, sname, rating, age)`, `Boats(bid, bname, color)`, `Reserves(sid, bid, day)`.

**Names of sailors who reserved boat 105.** Filter `Reserves` by `bid`, join to `Sailors` on `sid`, project the name:

$$\pi_{sname}\big(\sigma_{bid=105}(\text{Reserves}) \bowtie \text{Sailors}\big)$$

**Names of sailors who reserved a green boat.** A join chain — `Boats` filtered by colour, to `Reserves`, to `Sailors`:

$$\pi_{sname}\big(\sigma_{color='green'}(\text{Boats}) \bowtie \text{Reserves} \bowtie \text{Sailors}\big)$$

**`sid`s of sailors who reserved *every* boat.** "For all" means division — the (sailor, boat) pairs divided by the set of all boat ids:

$$\big(\pi_{sid,bid}(\text{Reserves})\big) \div \big(\pi_{bid}(\text{Boats})\big)$$

**Names of those sailors.** Divide first, then join for the name:

```
AllBoats ← (π sid,bid (Reserves)) ÷ (π bid (Boats))
Result   ← π sname (AllBoats ⋈ Sailors)
```

Schema: `Customer(cust_id, cust_name, annual_revenue)`, `Shipment(shipment_no, cust_id, weight, truckno, destination_city)`, `Truck(truckno, driver_name)`, `City(city_name, population)`.

**Names and revenues of customers who sent a shipment over 100 pounds.**

$$\pi_{cust\_name,\ annual\_revenue}\big(\sigma_{weight>100}(\text{Shipment}) \bowtie \text{Customer}\big)$$

**Customers whose shipments were delivered by driver 'Ramesh'.**

$$\pi_{cust\_name}\big(\sigma_{driver\_name='Ramesh'}(\text{Truck}) \bowtie \text{Shipment} \bowtie \text{Customer}\big)$$

**Customers who shipped to *every* city with population over 500,000.** Division again:

```
BigCities  ← π city_name (σ population > 500000 (City))
CustCities ← π cust_id, destination_city (Shipment)
CustIDs    ← CustCities ÷ BigCities
Result     ← π cust_name (CustIDs ⋈ Customer)
```

## The one idea to keep

Both halves are the same lesson from opposite ends. The index arithmetic shows *why* indexes exist — a lookup that was `log2(3000)` block reads becomes 3 — and the algebra shows *how* a declarative query decomposes into ordered operations. When you write SQL, the optimizer does both at once: choosing an access path from the index costs, and an operator order from the algebra.
