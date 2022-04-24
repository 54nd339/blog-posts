---
title: Normalization - Functional Dependencies and the Normal Forms
description: The systematic way to design tables that don't corrupt themselves - the update anomalies bad schemas cause, functional dependencies and Armstrong's axioms, computing attribute closure to find keys, and the normal forms from 1NF through BCNF to 5NF with lossless-join and dependency-preserving decomposition.
date: 2022-04-24
draft: false
slug: /dbms/rdb-design
tags:
  - Databases
  - Relational Model
  - Normalization
---

Put `(CourseID, CourseName, InstructorID, InstructorName, InstructorOffice)` in one table and three things go wrong. You can't record a new instructor's office until they're assigned a course (**insertion anomaly**). Deleting an instructor's only course erases the instructor entirely (**deletion anomaly**). Changing an instructor's office means updating every row for every course they teach, and missing one leaves the database self-contradictory (**update anomaly**).

All three come from **redundancy** — the same fact stored in many rows. **Normalization** is the systematic decomposition of a schema into smaller tables that eliminate that redundancy, driven by a formal theory of **functional dependencies**. This post covers that theory, the algorithm for finding keys, and the normal forms from 1NF to 5NF.

## Functional dependencies

A **functional dependency** `X → Y` (read "X determines Y") says: any two rows that agree on all attributes in `X` must also agree on all attributes in `Y`. In a `Students` table, `StudentID → StudentName` holds — one ID, one name. `StudentName → StudentID` does not, if two students can share a name.

An FD is **trivial** when `Y ⊆ X` (`{StudentID, Name} → StudentID`, always true) and **non-trivial** otherwise — the non-trivial ones are the real constraints.

### Armstrong's axioms

From a given set `F` of FDs, these rules infer every FD that must also hold:

- **Reflexivity** — if `Y ⊆ X` then `X → Y`.
- **Augmentation** — if `X → Y` then `XZ → YZ`.
- **Transitivity** — if `X → Y` and `Y → Z` then `X → Z`.

Derived from those:

- **Union** — `X → Y` and `X → Z` give `X → YZ`.
- **Decomposition** — `X → YZ` gives `X → Y` and `X → Z`.
- **Composition** — `X → Y` and `Z → W` give `XZ → YW`.
- **Pseudotransitivity** — `X → Y` and `YW → Z` give `XW → Z`.

### Closures

The **closure of an FD set**, `F⁺`, is every FD inferable from `F`. Its main use is checking whether two FD sets are equivalent (`F⁺ = G⁺`).

The **closure of an attribute set**, `X⁺`, is every attribute determined by `X`. Compute it by fixpoint: start with `X`; repeatedly, for any FD `Y → Z` in `F` where `Y ⊆ result`, add `Z`; stop when a full pass adds nothing.

```python
def attribute_closure(attrs, fds):
    """fds: list of (lhs_set, rhs_set). Returns X+ under fds."""
    closure = set(attrs)
    changed = True
    while changed:
        changed = False
        for lhs, rhs in fds:
            if lhs <= closure and not rhs <= closure:
                closure |= rhs
                changed = True
    return closure

def is_superkey(attrs, all_attrs, fds):
    return attribute_closure(attrs, fds) >= all_attrs

R = {"A", "B", "C", "D"}
F = [({"A"}, {"B"}), ({"B"}, {"C"}), ({"C", "D"}, {"A"})]
assert attribute_closure({"A"}, F) == {"A", "B", "C"}
assert is_superkey({"A", "D"}, R, F)          # A,D -> A,B,C,D
assert not is_superkey({"A"}, R, F)
```

`X` is a **superkey** iff `X⁺` is all the attributes; a **candidate key** iff it's a superkey and no proper subset is. To test whether `X → Y` follows from `F`, compute `X⁺` and check `Y ⊆ X⁺`.

### Minimal cover

A **minimal (canonical) cover** `Fc` is an FD set equivalent to `F` with all redundancy stripped: every RHS is a single attribute, no FD can be removed without changing the closure, and no attribute can be removed from any LHS. It's the input to the 3NF synthesis algorithm below.

## The normal forms

Each form forbids a class of dependency that causes anomalies.

**1NF — atomic values.** Every cell holds exactly one value; no repeating groups. Any properly defined relational table already satisfies this. It's the floor, and it still permits every anomaly.

**2NF — no partial dependencies.** Prerequisite: 1NF. No non-key attribute may depend on only *part* of a candidate key. This can only happen when the key is **composite**. Fix: if `(part of PK) → NonKeyAttr`, move `NonKeyAttr` and that part of the key into a new table. A single-attribute key means the relation is automatically in 2NF.

**3NF — no transitive dependencies.** Prerequisite: 2NF. No non-key attribute may depend on the key *through another non-key attribute* (`PK → A`, `A → B`). Formally: for every non-trivial `X → A`, either `X` is a superkey **or** `A` is a **prime attribute** (part of some candidate key). Fix: if `A → B` with both non-key, split `A` and `B` into their own table. 3NF removes most anomalies and is the usual practical target.

**BCNF — every determinant is a superkey.** Stricter than 3NF: for every non-trivial `X → A`, `X` must be a superkey — dropping 3NF's "or `A` is prime" escape. It matters when a relation has overlapping candidate keys. Fix: for a violating `X → A`, decompose `R` into `(X ∪ A)` and `(R − A)`, recursively.

**4NF — no non-key multivalued dependencies.** Prerequisite: BCNF. A **multivalued dependency** `X →→ Y` says that for each `X` value, the set of `Y` values is *independent* of the remaining attributes `Z = R − X − Y` (MVDs come in pairs: `X →→ Y` implies `X →→ Z`). Example: `Employee →→ Project` and `Employee →→ Skill` — an employee's projects and skills are unrelated, so one table `(Employee, Project, Skill)` forces every project × skill combination. 4NF requires `X` to be a superkey for every non-trivial MVD; fix by decomposing `R(X, Y, Z)` into `R1(X, Y)` and `R2(X, Z)`.

**5NF (PJ/NF) — no non-key join dependencies.** Prerequisite: 4NF. A **join dependency** holds when `R` can be losslessly reconstructed by joining three or more of its projections. 5NF requires every join dependency to be implied by the candidate keys. It addresses rare redundancy that only vanishes on a 3-or-more-way split, and is mostly of theoretical interest.

## Decomposing safely

Splitting a relation must preserve two properties.

**Lossless join.** Joining the pieces back must give *exactly* the original rows — no spurious extras, none lost. Test for a decomposition into `R1` and `R2`: the common attributes `R1 ∩ R2` must functionally determine all of `R1`, or all of `R2`. In other words the shared attributes form a key of at least one piece.

**Dependency preservation.** Every original FD must be checkable within a single decomposed table, without a join. Test: with `Fi` the FDs of `F` that mention only `Ri`'s attributes, `(F1 ∪ F2 ∪ … ∪ Fn)⁺` must equal `F⁺`.

### The two algorithms

**3NF synthesis.** Find a minimal cover `Fc`. For each FD `X → A` in `Fc`, make a schema `XA`. If no schema so far contains a candidate key of the original relation, add one more schema holding a candidate key. This guarantees lossless join, dependency preservation, *and* 3NF.

**BCNF decomposition.** Check `R` for BCNF. If it fails, pick a violating `X → Y` (`X` not a superkey) and split into `R1(X ∪ Y)` and `R2(R − Y)`; recurse on both. This guarantees lossless join and BCNF — but **may not preserve all dependencies**.

That's the trade-off: BCNF removes more redundancy, but reaching it can cost you the ability to enforce some FDs inside a single table. Most designs stop at 3NF for exactly this reason.

## The one idea to keep

Normalization is one move applied repeatedly: find a dependency where the determinant isn't a key, and split it into its own table so the fact is stored once. The normal forms just name which kind of dependency you're eliminating — partial (2NF), transitive (3NF), any non-superkey determinant (BCNF), multivalued (4NF). The constraint on all of it is that decomposition stays lossless, and ideally dependency-preserving, so you've reorganised the data without changing what it says or what it can enforce.
