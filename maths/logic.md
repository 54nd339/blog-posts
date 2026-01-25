---
title: The Foundations of Computation and Structure - An Introduction to Discrete Mathematics
description: Delve into the core pillars of discrete mathematics, exploring the formal reasoning of logic, the science of algorithms, and the abstract elegance of algebraic structures. This post covers propositional and predicate logic, proof techniques, algorithm complexity, and the hierarchy of algebraic systems from groups to fields.
date: 2023-03-07
draft: false
slug: /pensieve/maths/logic
tags:
  - Mathematics
  - Logic
---

While much of classical mathematics deals with continuous quantities, the world of modern science, particularly computer science, is built upon discrete structures—objects that are distinct and separable. Discrete mathematics is the branch that studies these structures. Its two major pillars are **logic**, which provides the rules for rigorous reasoning and formal proof, and **abstract algebra**, which examines sets equipped with operations, uncovering the fundamental anatomy of mathematical systems. This exploration provides a comprehensive journey through these fields, from the syntax of propositions and the strategy of proofs to the powerful abstractions of groups, rings, and fields.

---
## The Foundations of Reasoning - Mathematical Logic and Algorithms

Logic is the bedrock of mathematics and computer science. It provides the formal language for making precise statements and the rules for deducing new truths from existing ones.

### Propositional Logic: The Algebra of Truth
Propositional logic deals with propositions and the compound statements formed by connecting them.

* **Proposition and Truth Values:** A **proposition** is a declarative sentence that is either **true (T)** or **false (F)**, but not both. These are its **truth values**.
* **Connectives (Logical Operators):** We combine propositions using connectives:
    1.  **Negation (NOT, ¬):** Reverses the truth value.
    2.  **Conjunction (AND, ∧):** $P \land Q$ is true only if both $P$ and $Q$ are true.
    3.  **Disjunction (OR, ∨):** $P \lor Q$ is true if at least one of $P$ or $Q$ is true.
    4.  **Implication (Conditional, →):** $P \to Q$ ("if P, then Q") is false only when $P$ is true and $Q$ is false.
    5.  **Biconditional (↔):** $P \leftrightarrow Q$ ("P if and only if Q") is true only when $P$ and $Q$ have the same truth value.

* **Logical Equivalence of Compound Statements:** Two compound statements are **logically equivalent** if they have the same truth value for all possible truth values of their constituent propositions. This is often demonstrated using truth tables. For example, De Morgan's Law states $\neg(P \land Q) \equiv \neg P \lor \neg Q$.

* **Rules of Inference:** These are templates for constructing valid arguments. Given a set of true premises, a rule of inference allows us to deduce a true conclusion.
    * **Modus Ponens:** If $P$ and $P \to Q$ are true, then $Q$ is true.
    * **Modus Tollens:** If $\neg Q$ and $P \to Q$ are true, then $\neg P$ is true.
    * **Hypothetical Syllogism:** If $P \to Q$ and $Q \to R$ are true, then $P \to R$ is true.

### Predicates and Quantifiers
Propositional logic is limited because it cannot express statements about properties of objects or generalizations. **Predicate logic** extends it.
* **Predicates:** A predicate is a property or relation that can be affirmed of an object. It is a statement involving variables, like $P(x)$: "$x$ is greater than 3". It becomes a proposition once the variable $x$ is assigned a value.
* **Quantifiers:** Quantifiers express the extent to which a predicate is true over a range of elements.
    * **Universal Quantifier ($\forall$):** "For all" or "for every". $\forall x P(x)$ asserts that $P(x)$ is true for every $x$ in the domain.
    * **Existential Quantifier ($\exists$):** "There exists" or "for some". $\exists x P(x)$ asserts that there is at least one $x$ in the domain for which $P(x)$ is true.
* **Nested Quantifiers:** Quantifiers can be nested, and their order is crucial. For example, $\forall x \exists y (x+y=0)$ (for every number $x$, there is an additive inverse $y$) is true for integers, but $\exists y \forall x (x+y=0)$ (there is a single number $y$ that is the additive inverse for all numbers $x$) is false.

### Proof Methods & Strategy
A mathematical proof is a rigorous argument that establishes the truth of a statement.
* **Direct Proof:** To prove $p \to q$, assume $p$ is true and use rules of inference, axioms, and logical equivalences to deduce that $q$ must be true.
* **Proof by Contraposition (Indirect):** To prove $p \to q$, prove its logically equivalent contrapositive, $\neg q \to \neg p$. Assume $\neg q$ is true and deduce that $\neg p$ must be true.
* **Proof by Contradiction (Indirect):** To prove a statement $p$, assume its negation $\neg p$ is true. Then, deduce a contradiction (a statement of the form $r \land \neg r$). This shows that the assumption $\neg p$ must be false, so $p$ must be true.

### Mathematical Induction
This is a powerful proof technique for establishing that a property $P(n)$ holds for all natural numbers $n$ (or all integers $n \ge n_0$).
* **Principle of Mathematical Induction:**
    1.  **Basis Step:** Prove that $P(1)$ (or $P(n_0)$) is true.
    2.  **Inductive Step:** Prove that for every positive integer $k$, if $P(k)$ is true (this is the **inductive hypothesis**), then $P(k+1)$ must also be true.
    These two steps establish the truth of $P(n)$ for all integers $n \ge 1$.
* **Strong Induction:** The inductive step is modified: instead of assuming only $P(k)$ is true, we assume $P(j)$ is true for all integers $j$ from the base case up to $k$. This stronger hypothesis is sometimes needed to prove the $P(k+1)$ case.
* **Well-Ordering Principle:** This states that every non-empty set of positive integers has a least element. It is an axiom equivalent to the principle of mathematical induction.

### Algorithms and Complexity
* **Algorithm:** An algorithm is a finite sequence of well-defined, computer-implementable instructions to solve a class of problems or to perform a computation.
* **Growth of Functions & Complexity:** **Algorithmic complexity** measures the amount of resources (typically time or memory) an algorithm requires to run, as a function of the size of the input. We use **Big-O notation** to describe the upper bound on this growth. A function $f(n)$ is $O(g(n))$ if there exist positive constants $C$ and $k$ such that $|f(n)| \le C|g(n)|$ for all $n \ge k$.
* **Recursive Algorithms:** A recursive algorithm is one that solves a problem by calling itself with smaller instances of the same problem. This approach often mirrors proofs by mathematical induction. A classic example is the factorial function: `Factorial(n) = n * Factorial(n-1)`, with the base case `Factorial(0) = 1`.

---
## The Anatomy of Structure - Algebraic Systems

Algebraic structures are sets equipped with one or more binary operations that satisfy certain axioms. They abstract the properties of familiar number systems like the integers and real numbers.

### Lattices and Boolean Algebra
* **Lattice:** A **lattice** is a partially ordered set in which every two elements have a unique **supremum** (least upper bound, or join, denoted by $\lor$) and a unique **infimum** (greatest lower bound, or meet, denoted by $\land$).
* **Boolean Algebra:** A **Boolean algebra** is a special type of algebraic structure that models logical operations. It is a complemented, distributive lattice. Its properties are fundamental to digital logic circuit design and set theory.

### A Hierarchy of Algebraic Structures
We can build up structures by adding axioms for a binary operation $*$ on a set $S$.

1.  **Semigroup:** A set $S$ with an **associative** binary operation $*$. That is, for all $a,b,c \in S$, $(a*b)*c = a*(b*c)$.
    *Example:* The set of positive integers under addition $(\mathbb{Z}^+, +)$.

2.  **Monoid:** A semigroup that has an **identity element** $e \in S$ such that for every element $a \in S$, $e*a = a*e = a$.
    *Example:* The set of non-negative integers under addition $(\mathbb{N}_0, +)$, where $0$ is the identity.

3.  **Group:** A monoid in which every element has an **inverse**. For each $a \in S$, there exists an element $a^{-1} \in S$ such that $a*a^{-1} = a^{-1}*a = e$. A group $(G,*)$ must satisfy four axioms:
    * **Closure:** If $a, b \in G$, then $a*b \in G$.
    * **Associativity:** $(a*b)*c = a*(b*c)$.
    * **Identity Element:** There exists $e \in G$ such that $a*e=e*a=a$.
    * **Inverse Element:** For each $a \in G$, there exists $a^{-1} \in G$ such that $a*a^{-1}=a^{-1}*a=e$.
    *Example:* The set of integers under addition $(\mathbb{Z}, +)$.

4.  **Abelian Group:** A group in which the binary operation is also **commutative**. That is, for all $a,b \in G$, $a*b=b*a$.
    *Example:* $(\mathbb{Z}, +)$ is an Abelian group. The set of non-zero rational numbers under multiplication $(\mathbb{Q}-\{0\}, \cdot)$ is also an Abelian group.

### Deeper into Group Theory
* **Properties of Groups:** From the axioms, we can derive properties like the uniqueness of the identity element and the uniqueness of the inverse for each element.
* **Cyclic Groups:** A group $G$ is **cyclic** if it can be generated by a single element $a$, called the **generator**. This means every element in $G$ is a power of $a$ (or a multiple of $a$ if the operation is addition). We write $G = \langle a \rangle$.
* **Subgroup:** A subset $H$ of a group $G$ that is itself a group under the same operation as $G$.
* **Cosets:** For a subgroup $H$ of $G$, a **left coset** of $H$ with respect to an element $g \in G$ is the set $gH = \{gh \mid h \in H\}$. Right cosets $Hg$ are defined similarly. Cosets partition the group $G$.
* **Lagrange’s Theorem:** A fundamental theorem stating that for any finite group $G$, the order (number of elements) of any subgroup $H$ of $G$ must divide the order of $G$. The number of distinct cosets, called the index $[G:H]$, is $|G|/|H|$.
* **Normal Subgroup:** A subgroup $H$ of $G$ is **normal** if its left and right cosets coincide for all elements in $G$ (i.e., $gH = Hg$ for all $g \in G$). Normal subgroups are important because they allow for the construction of quotient groups.
* **Homomorphism and Isomorphism:**
    * **Homomorphism:** A map $\phi$ between two groups $(G, *)$ and $(G', \star)$ is a homomorphism if it preserves the group operation: $\phi(a * b) = \phi(a) \star \phi(b)$.
    * **Isomorphism:** An isomorphism is a homomorphism that is also a bijection (one-to-one and onto). Two groups are isomorphic if they are structurally identical, just with different labels for their elements.

### Structures with Two Operations: Rings and Fields
These structures, like the integers or real numbers, have two binary operations (typically addition and multiplication).

* **Ring ($R,+, \cdot$):** A set with two binary operations, addition and multiplication, such that:
    1.  $(R,+)$ is an Abelian group.
    2.  Multiplication is associative.
    3.  The distributive laws hold: $a(b+c) = ab+ac$ and $(b+c)a = ba+ca$.
    *Example:* The set of integers $(\mathbb{Z}, +, \cdot)$.

* **Integral Domain:** A **commutative ring** with a multiplicative identity ($1 \neq 0$) that has no **zero-divisors**. No zero-divisors means if $ab=0$, then either $a=0$ or $b=0$.
    *Example:* The integers $(\mathbb{Z}, +, \cdot)$ form an integral domain.

* **Field ($F, +, \cdot$):** A **commutative ring** with a multiplicative identity ($1 \neq 0$) in which every **non-zero element has a multiplicative inverse**.
    A field is a set that forms an Abelian group under addition, and its non-zero elements form an Abelian group under multiplication, with distributivity connecting the two operations.
    *Examples:* The rational numbers $(\mathbb{Q}, +, \cdot)$, the real numbers $(\mathbb{R}, +, \cdot)$, and the complex numbers $(\mathbb{C}, +, \cdot)$ are all fields.

---
## The Mathematics of Communication - Information Theory

Information theory is a mathematical discipline that deals with the quantification, storage, and communication of information. It was founded by Claude Shannon in his seminal 1948 paper, "A Mathematical Theory of Communication."

### Key Concepts
* **Information Entropy (Shannon Entropy):** The central concept is **entropy**, which, in this context, measures the average level of "information," "surprise," or "uncertainty" inherent in a random variable's possible outcomes. For a discrete random variable $X$ with possible outcomes $x_1, \ldots, x_n$ and probabilities $p(x_i)$:
    $$ H(X) = -\sum_{i=1}^n p(x_i) \log_b p(x_i) $$
    * The base of the logarithm, $b$, determines the units. If $b=2$, the unit is **bits**. If $b=e$, the unit is **nats**.
    * A distribution with high entropy is very uncertain (e.g., a fair coin toss), while a distribution with low entropy is very predictable (e.g., a two-headed coin toss).
* **Channel Capacity:** This represents the tight upper bound on the rate at which information can be reliably transmitted over a communication channel. It is a property of the channel itself, not the information being sent.
* **Shannon's Noisy-Channel Coding Theorem:** This is a fundamental theorem of information theory. It states that for any given communication channel with a certain level of noise, it is possible to communicate discrete data nearly error-free up to a computable maximum rate, the **channel capacity**. To achieve this, one needs to use appropriate error-correcting codes. It provides a theoretical limit to reliable communication, fundamentally linking error correction to the concept of entropy.

---
## Key Takeaways: The Foundations of Formal Reasoning and Structure

The disciplines of formal logic and abstract algebra provide the essential framework for rigorous thought in mathematics, computer science, and philosophy.

* **Logic as the Basis of Reasoning:** Propositional and predicate logic provide a formal language for expressing statements and arguments. Rules of inference allow for the step-by-step deduction of new truths, while proof strategies like mathematical induction provide powerful methods for establishing facts about infinite sets.
* **Algorithms and Complexity:** Logic underpins the design of algorithms, and the study of their complexity using notations like Big-O allows us to analyze and compare their efficiency.
* **Algebraic Structures Generalize Arithmetic:** By defining sets with operations that follow specific axioms, we can study abstract structures like semigroups, monoids, groups, rings, and fields.
* **Group Theory as a Study of Symmetry:** Groups are one of the most important algebraic structures, capturing the essence of symmetry. Key concepts like subgroups, cosets, and Lagrange's Theorem reveal their deep internal structure.
* **A Hierarchy of Structure:** From the simple associative property of a semigroup to the rich structure of a field (which supports all standard arithmetic), these algebraic systems form a hierarchy that allows us to understand the properties needed for different mathematical contexts.
* **Information Theory:** Provides a mathematical framework for quantifying information and uncertainty through the concept of entropy, and sets the fundamental limits of data compression and reliable communication.

These foundational areas of discrete mathematics are not just abstract constructions; they are the tools that enable us to build proofs, design algorithms, and understand the fundamental nature of mathematical structure itself.