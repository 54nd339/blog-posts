---
title: Query Languages - Relational Algebra, Calculus, and SQL
description: The three ways to ask a relational database a question - relational algebra as a procedural operator set, relational calculus as declarative logic, SQL as the practical standard with DDL, DML, joins, subqueries, and aggregation, and PL/SQL for procedural logic in the database.
date: 2022-04-23
draft: false
slug: /dbms/relational-query-lang
tags:
  - Databases
  - Query Languages
  - SQL
---

There are two formal ways to describe a query against a relational database, and one practical one. **Relational algebra** is procedural — a sequence of operators, each taking relations and producing a relation, that says *how* to compute the answer. **Relational calculus** is declarative — a logical formula describing *what* the answer looks like. **SQL** is the standard you actually type, and it's declarative: you state the shape of the result and the [query optimizer](/citadel/dbms/advanced) works out the algebra.

The two formal languages are provably equal in expressive power, and SQL is built on their ideas, so understanding them is understanding what SQL can and can't do.

## Relational algebra

Each operator returns a new relation. The unary ones:

- **Select** `σ_condition(R)` — keep rows matching the condition (comparisons `=`, `≠`, `<`, `≤`, `>`, `≥`, combined with `∧`, `∨`, `¬`). Same columns, fewer rows: `σ_{Major='CS' ∧ GPA>3.5}(STUDENT)`.
- **Project** `π_{attrs}(R)` — keep only those columns, and **remove duplicate rows**: `π_{Name, Major}(STUDENT)`.
- **Rename** `ρ_{NewName}(expr)` — name an intermediate result or its attributes.

Binary set operators need the two relations to be **union-compatible** — same number of attributes, compatible domains:

- **Union** `R ∪ S` — rows in either, deduplicated.
- **Set difference** `R − S` — rows in `R` but not `S`.
- **Cartesian product** `R × S` — every row of `R` paired with every row of `S`. Degree is `degree(R) + degree(S)`, cardinality is the product. Rarely useful alone; it's the raw material for joins.

Derived operators:

- **Intersection** `R ∩ S = R − (R − S)`.
- **Join** `R ⋈ S` — the workhorse. A **theta join** `R ⋈_θ S` is `σ_θ(R × S)`. An **equi join** restricts `θ` to equality. A **natural join** is an equi join on all identically-named attributes, projecting away the duplicate columns. **Outer joins** (`⟕` left, `⟖` right, `⟗` full) keep unmatched rows, padding the missing side with NULLs. A **self join** joins a relation to a renamed copy of itself (employee-to-manager).
- **Division** `R ÷ S` — for "for all" queries. If `R(A, B)` and `S(B)`, it returns the `A` values that appear in `R` paired with *every* `B` in `S`: `(π_{StudentID, CourseID}(ENROLLMENT)) ÷ (π_{CourseID}(σ_{Dept='CS'}(COURSE)))` finds students who took every CS course.
- **Assignment** `Temp ← expr` — break a complex query into named steps.

Extensions: **generalized projection** allows arithmetic in the list (`π_{Salary × 1.1}(EMP)`), and **aggregation** `_{group attrs} g _{SUM, AVG, COUNT, …}(R)` computes aggregates per group, like SQL's `GROUP BY`. The [calculations post](/citadel/dbms/calcs) works through non-trivial algebra queries step by step.

## Relational calculus

Instead of steps, describe the result with logic.

**Tuple relational calculus (TRC)** — `{ T | P(T) }`, "all tuples `T` where `P(T)` is true". The predicate `P` is built from atoms (`T ∈ Relation`, `T.attr op S.attr`, `T.attr op constant`), connectives (`¬`, `∧`, `∨`, `⇒`), and quantifiers (`∃T`, `∀T`). Sailors rated above 7:

$$\{\, T \mid \exists S \in \text{Sailors}\ (S.\text{rating} > 7 \land T.\text{sname} = S.\text{sname}) \,\}$$

**Domain relational calculus (DRC)** — `{ <x1, …, xn> | P(x1, …, xn) }`, with variables ranging over attribute *values* rather than whole tuples:

$$\{\, \langle N \rangle \mid \exists I, R, A\ (\langle I, N, R, A \rangle \in \text{Sailors} \land R > 7) \,\}$$

Both need **safe** expressions — ones guaranteed to produce a finite result drawn only from values in the database. Both are equivalent in power to relational algebra, and they're the theoretical basis for SQL.

## SQL

SQL is declarative: state what you want, let the DBMS pick the plan. Conceptually it has four sublanguages — **DDL** (structure), **DML** (data), **DCL** (`GRANT`, `REVOKE`), **TCL** (`COMMIT`, `ROLLBACK`, `SAVEPOINT`).

### DDL

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName  VARCHAR(50),
    Major     VARCHAR(30),
    GPA       DECIMAL(3, 2),
    DeptID    INT,
    UNIQUE (FirstName, LastName),
    FOREIGN KEY (DeptID) REFERENCES Departments(DeptID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

- **`PRIMARY KEY`** — unique and not null.
- **`FOREIGN KEY … REFERENCES`** — enforces referential integrity; `ON DELETE` / `ON UPDATE` say what happens when the referenced key changes (`CASCADE`, `SET NULL`, `RESTRICT`).
- **`UNIQUE`** — distinct values, but allows NULL.
- **`NOT NULL`** — no NULLs.

`ALTER TABLE` adds or drops columns and constraints; `DROP TABLE` removes the table and its data.

### DML

```sql
SELECT StudentID, FirstName, GPA
FROM   Students
WHERE  Major = 'CompSci' AND GPA > 3.0
ORDER BY LastName, FirstName;

INSERT INTO Students (StudentID, FirstName, LastName, Major, GPA, DeptID)
VALUES (101, 'Alice', 'Smith', 'CompSci', 3.8, 10);

UPDATE Students SET Major = 'Computer Science' WHERE Major = 'CompSci';

DELETE FROM Students WHERE StudentID = 101;
```

`INSERT … SELECT` bulk-loads from a query. `WHERE` is `σ`; `ORDER BY` has no algebra equivalent (relations are unordered).

### Joins, subqueries, aggregation

```sql
SELECT S.FirstName, D.DeptName
FROM   Students S
INNER JOIN Departments D ON S.DeptID = D.DeptID
WHERE  S.GPA > 3.5;
```

`INNER`, `LEFT`, `RIGHT`, and `FULL OUTER` joins mirror the algebra. **Subqueries** nest a query inside another:

```sql
-- in WHERE: set membership or a comparison
SELECT FirstName FROM Students
WHERE DeptID IN (SELECT DeptID FROM Departments WHERE Location = 'Building A');

SELECT FirstName, GPA FROM Students
WHERE GPA > (SELECT AVG(GPA) FROM Students);

-- in FROM: a derived table
SELECT T.Major, T.AvgGPA
FROM  (SELECT Major, AVG(GPA) AS AvgGPA FROM Students GROUP BY Major) T
WHERE T.AvgGPA > 3.2;

-- in SELECT: a correlated scalar subquery
SELECT StudentID, FirstName,
       (SELECT COUNT(*) FROM Enrollments E WHERE E.StudentID = S.StudentID) AS Courses
FROM Students S;
```

A **correlated** subquery references the outer query and re-runs per outer row. **Aggregation** groups rows and summarises:

```sql
SELECT Major, COUNT(*) AS NumStudents, AVG(GPA) AS AverageGPA
FROM   Students
WHERE  GPA IS NOT NULL   -- filters rows, before grouping
GROUP BY Major
HAVING COUNT(*) > 10;    -- filters groups, after aggregation
```

`WHERE` runs before `GROUP BY`, `HAVING` after — the one distinction people trip on.

## PL/SQL and procedural extensions

Pure SQL can't loop or branch. Vendors add a procedural layer — Oracle's **PL/SQL**, SQL Server's **T-SQL**, PostgreSQL's **PL/pgSQL** — with variables, `IF`/`ELSE`, `FOR`/`WHILE` loops, and exception handling.

```sql
DECLARE
  v_gpa students.gpa%TYPE;
  v_id  students.studentid%TYPE := 101;
BEGIN
  SELECT gpa INTO v_gpa FROM students WHERE studentid = v_id;
  IF v_gpa >= 3.5 THEN
    DBMS_OUTPUT.PUT_LINE('Student ' || v_id || ' is on the Dean''s List.');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Student ' || v_id || ' GPA: ' || v_gpa);
  END IF;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('Student ' || v_id || ' not found.');
END;
```

It bundles into **stored procedures** (may return nothing), **functions** (must return a value), and **triggers** (code that fires automatically on `INSERT`/`UPDATE`/`DELETE`). **Cursors** walk a result set row by row — *implicit* for single-row DML, *explicit* for multi-row queries. The payoff is fewer client round trips, precompiled logic, and granting permissions on a procedure instead of the underlying tables.

## The one idea to keep

Algebra and calculus are the same query from two directions — a recipe versus a description — and they bound what any relational language can express. SQL is the description form with a real syntax, and its declarative nature is the whole point: you specify the result, and the optimizer, using algebra equivalences, chooses among many plans that would all produce it. PL/SQL is the escape hatch for the logic that doesn't fit that model.
