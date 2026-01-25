---
title: Indexing Calculations & Relational Algebra Queries
description: Get hands-on with DBMS concepts! Learn how to calculate index sizes and performance, and see practical examples of complex queries using Relational Algebra.
date: 2023-03-27
draft: false
slug: /pensieve/dbms/calcs-queries
tags:
  - DBMS
  - CS Basics
---

Hey data explorers and query wizards! In our previous posts, we've covered a lot of conceptual ground on database design, querying, and internals. Sometimes, though, it helps to get a bit more concrete. Today, we're diving into two specific areas:

1. **Indexing Calculations:** How do we estimate the size and performance impact of different index types?
2. **Relational Algebra in Action:** Seeing more complex query examples to solidify understanding of the operators.

Let's crunch some numbers and build some queries!

## Part 1: Sizing Up Your Indexes - Performance Calculations

We know indexes speed up queries, but how much space do they take, and how much faster is the access? We can estimate this using some basic calculations.

**Let's define some terms:**

* `N`: Total number of records in the data file.
* `B`: Block size (in bytes) - the amount of data read from disk in one I/O operation.
* `R`: Record size (in bytes).
* `K`: Size of the index key field (in bytes).
* `P`: Size of a block pointer or record pointer (in bytes).

**Common Calculations:**

1. **Data Blocking Factor (`bfr`):** How many data records fit in one disk block?
   * `bfr = floor(B / R)`
2. **Number of Data Blocks (`n`):** How many blocks are needed to store the entire data file?
   * `n = ceil(N / bfr)`
3. **Index Entry Size (`I`):** Size of one entry in the index file.
   * `I = K + P`
4. **Index Blocking Factor (`bfri`):** How many index entries fit in one disk block?
   * `bfri = floor(B / I)`

**Scenario 1: Primary Index**

* Recall: Sparse index, one entry per data block.
* **Number of Index Entries:** `n` (same as number of data blocks)
* **Number of Index Blocks (`m`):** `m = ceil(n / bfri)`
* **Block Accesses (Avg):** Searching the index (binary search on `m` blocks) + reading the data block.
  * `Block Accesses ≈ ceil(log2(m)) + 1`
* **Comparison:** Without index (binary search on data file): `ceil(log2(n))` block accesses.

**Scenario 2: Secondary Index (Dense)**

* Recall: Dense index, one entry per data record.
* **Number of Index Entries:** `N` (same as number of data records)
* **Number of Index Blocks (`m`):** `m = ceil(N / bfri)`
* **Block Accesses (Avg):** Searching the index + reading the data block.
  * `Block Accesses ≈ ceil(log2(m)) + 1` (assuming pointer goes to block)
* **Comparison:** Without index (linear scan): `n` block accesses (worst case).

**Scenario 3: Multi-Level Index**

* Let's calculate levels iteratively.
* **Level 1 Index Blocks (`m1`):** Calculate `m` as above (depends if primary/secondary). `m1 = ceil(Num_Index_Entries_L1 / bfri)`
* **Level 2 Index Blocks (`m2`):** Indexing the Level 1 index blocks. `m2 = ceil(m1 / bfri)`
* **Level 3 Index Blocks (`m3`):** Indexing the Level 2 index blocks. `m3 = ceil(m2 / bfri)`
* ...Continue until a level (`mk`) has only 1 block.
* **Total Levels:** `k`
* **Block Accesses:** One access per index level + one access for the data block.
  * `Block Accesses = k + 1`

**Example Calculation:**

* `N = 30,000` records
* `B = 1024` bytes
* `R = 100` bytes
* `K = 6` bytes
* `P = 9` bytes

1. `bfr = floor(1024 / 100) = 10` records/block
2. `n = ceil(30000 / 10) = 3000` data blocks
3. `I = 6 + 9 = 15` bytes/index entry
4. `bfri = floor(1024 / 15) = 68` entries/index block

* **Primary Index:**

  * Index Entries = `n = 3000`
  * Index Blocks `m = ceil(3000 / 68) = 45` blocks
  * Block Accesses ≈ `ceil(log2(45)) + 1 = 6 + 1 = 7`
  * (Without index ≈ `ceil(log2(3000)) = 12`)
* **Secondary Index:**

  * Index Entries = `N = 30000`
  * Index Blocks `m = ceil(30000 / 68) = 442` blocks
  * Block Accesses ≈ `ceil(log2(442)) + 1 = 9 + 1 = 10`
  * (Without index ≈ `3000` worst case)
* **Multi-Level (using Primary Index example):**

  * Level 1 blocks `m1 = 45`
  * Level 2 blocks `m2 = ceil(45 / 68) = 1` block
  * Total Levels `k = 2`
  * Block Accesses = `2 + 1 = 3`

These calculations show how indexing, especially multi-level indexing, dramatically reduces the number of disk I/Os needed to find data.

## Part 2: Relational Algebra Queries in Action

Let's revisit Relational Algebra with some specific examples based on common database scenarios (similar to Section 13 in the PDF).

**Schema Definitions:**

* `Sailors(sid, sname, rating, age)`
* `Boats(bid, bname, color)`
* `Reserves(sid, bid, day)`

**Example Queries:**

1. **Find the names of sailors who've reserved boat 105:**

   * *Thought:* We need info from `Reserves` (to filter by `bid=105`) and `Sailors` (to get `sname`). We need to connect them on `sid`. Natural join is perfect here.
   * *Algebra:* `π sname (σ bid=105 (Reserves) ⋈ Sailors)`
   * *Alternative (step-by-step):*
     ```
     Temp ← σ bid=105 (Reserves)
     Result ← π sname (Temp ⋈ Sailors)
     ```
2. **Find the names of sailors who've reserved a green boat:**

   * *Thought:* Need `Sailors` for name, `Reserves` to link sailor to boat, `Boats` to filter by color. Chain of joins needed.
   * *Algebra:* `π sname ( (σ color='green' (Boats)) ⋈ Reserves ⋈ Sailors )`
3. **Find the sailor IDs (`sid`) of sailors who have reserved *all* boats:**

   * *Thought:* This "for all" phrasing screams DIVISION. We need the pairs of (sailor, boat reserved) from `Reserves` and divide by the list of all boat IDs from `Boats`.
   * *Algebra:* `(π sid, bid (Reserves)) ÷ (π bid (Boats))`
4. **Find the *names* of sailors who've reserved all boats:**

   * *Thought:* First find the `sid`s using division (as above), then join with `Sailors` to get the names. Use assignment for clarity.
   * *Algebra:*
     ```
     AllBoatsReservedBy ← (π sid, bid (Reserves)) ÷ (π bid (Boats))
     Result ← π sname (AllBoatsReservedBy ⋈ Sailors)
     ```

**Another Schema:**

* `Customer(cust_id, cust_name, annual_revenue)`
* `Shipment(shipment_no, cust_id, weight, truckno, destination_city)`
* `Truck(truckno, driver_name)`
* `City(city_name, population)`

5. **Find the names and revenues of customers who sent shipments weighing > 100 pounds:**

   * *Thought:* Filter `Shipment` by weight, join with `Customer` on `cust_id`, project desired columns.
   * *Algebra:* `π cust_name, annual_revenue ( σ weight > 100 (Shipment) ⋈ Customer )`
6. **List customers (names) whose shipments were delivered by driver 'Ramesh':**

   * *Thought:* Filter `Truck` by driver name, join with `Shipment` on `truckno`, join with `Customer` on `cust_id`, project name.
   * *Algebra:* `π cust_name ( (σ driver_name='Ramesh' (Truck)) ⋈ Shipment ⋈ Customer )`
7. **Find customers who have sent shipments to *every* city with population > 500,000:**

   * *Thought:* Another division query. Dividend needs `(cust_id, destination_city)` pairs from `Shipment`. Divisor needs `city_name` from `City` filtered by population. Need a rename potentially if attribute names don't match exactly (`destination_city` vs `city_name`). Let's assume they match for simplicity here, or use rename `ρ`.
   * *Algebra:*
     ```
     BigCities ← π city_name ( σ population > 500000 (City) )
     CustomerShipmentsToCities ← π cust_id, destination_city (Shipment)
     CustomerIDs ← CustomerShipmentsToCities ÷ BigCities
     Result ← π cust_name (CustomerIDs ⋈ Customer)
     ```

These examples illustrate how combining the basic relational algebra operators allows us to formulate complex queries procedurally.

## Conclusion

Getting hands-on, whether through calculation or query formulation, solidifies understanding. Estimating index performance helps justify design choices, while working through relational algebra examples clarifies how data can be manipulated step-by-step to answer complex questions. These practical skills complement the conceptual knowledge needed to build and manage effective databases.
