---
title: Logic in AI - From Propositional Reasoning to First-Order Knowledge
description: An exploration of how AI agents use logic to reason about the world, starting with knowledge-based agents, the Wumpus World, propositional logic, and advancing to the more expressive First-Order Logic for complex knowledge representation and engineering.
date: 2023-06-25
draft: false
slug: /pensieve/ai-ml/logical-agents
tags:
  - AI/ML
  - CS Basics
---

Hey everyone! We've previously touched upon how AI agents perceive environments and search for solutions. But how do these agents truly "think" or "reason" about the information they gather? That's where logic comes into play, forming the backbone of many intelligent systems. Today, we'll explore **Logical Agents**, see them in action in the infamous **Wumpus World**, understand the fundamentals of **Propositional Logic**, and then see why we often need to step up to the more powerful **First-Order Logic**.

## Knowledge-Based Agents: The Brains of the Operation
At the heart of many logical AI systems are **knowledge-based agents**. These agents are composed of two main parts:
1.  A **Knowledge Base (KB)**: This is a set of sentences expressed in a formal language, representing what the agent knows about the world. A KB typically consists of facts (statements of truth), procedural rules (well-defined, invariant rules), and heuristics (procedures for when invariant rules aren't available).
2.  An **Inference Mechanism (or Inference Engine)**: This allows the agent to deduce new sentences from the existing KB and use these new sentences to decide what actions to take.

These agents operate on a **declarative approach**: you TELL them what they need to know, and then they can ASK themselves (using their inference mechanism) what to do. The key operations are:
* **TELL**: To add new sentences (knowledge) to the KB.
* **ASK**: To query what is known by the KB (what follows from the KB).

An agent can be viewed at the **knowledge level** (what it knows) or the **implementation level** (the data structures and algorithms).

## The Wumpus World: A Perilous Playground for Logical Agents

To understand how logical agents work, AI researchers often use illustrative environments. One classic example is the **Wumpus World**.

**PEAS Description**:
* **Performance Measure**: +1000 for finding gold, -1000 for death (falling into a pit or eaten by the Wumpus), -1 per step taken, -10 for using the arrow.
* **Environment**: A grid of squares. Squares adjacent to the Wumpus are smelly. Squares adjacent to a pit are breezy. Glitter appears in a square if gold is present. Shooting an arrow kills the Wumpus if the agent is facing it (uses the only arrow). Grabbing picks up gold if in the same square; releasing drops it.
* **Actuators**: Turn Left, Turn Right, Forward, Grab, Release, Shoot.
* **Sensors**: Stench, Breeze, Glitter, Bump (if an agent walks into a wall), Scream (if the Wumpus is killed).

**Environment Characterization**:
* **Partially Observable**: The agent only perceives its local square.
* **Deterministic**: Outcomes of actions are exactly specified.
* **Sequential**: Actions have long-term consequences.
* **Static**: The Wumpus and pits do not move.
* **Discrete**: Percepts and actions are distinct.
* **Single-agent?** Yes, the Wumpus is more like a natural feature.

**Exploring the Wumpus World**:
An agent starts, usually in [1,1]. It uses its percepts to infer the state of neighboring squares. For instance:
* If the agent is in [1,1] and perceives no stench or breeze, it can infer that its adjacent squares [1,2] and [2,1] are safe (no pit, no Wumpus).
* If it moves to [2,1] and perceives a breeze, it infers there's a pit in an adjacent square ([1,1] - known safe, [2,2], or [3,1]). If it then moves to [1,2] (known safe) and perceives no breeze, it can deduce the pit must be in [3,1].

![Wumpus World Exploration](https://www.researchgate.net/profile/Paul-Bello-5/publication/2430434/figure/fig1/AS:669983520595969@1536747927730/A-Typical-Wumpus-World.png)

This simple example shows how an agent combines percepts with its knowledge (rules about breezes and stenches) to make safe moves and achieve its goal.

---
## Logic: The Formal Language of Reasoning
To enable this kind of reasoning, agents need a formal language. **Logics** are precisely that: formal languages for representing information such that conclusions can be drawn.

Key aspects of any logic:
* **Syntax**: Defines the rules for forming well-formed sentences in the language. For example, in arithmetic, `x + 2 >= y` is a sentence, but `x2+y > {}` is not.
* **Semantics**: Defines the "meaning" of sentences by specifying the conditions under which a sentence is true in a given "world" or **model**. A model is a formally structured world with respect to which truth can be evaluated. We say `m` is a model of a sentence `α` if `α` is true in `m`.

**Entailment (KB ╞ α)**: This is a crucial concept. A knowledge base KB **entails** a sentence α if and only if α is true in all worlds (models) where KB is true. For example, if KB contains "The Giants won" and "The Reds won," it entails "Either the Giants won or the Reds won". Entailment is about what logically follows from the known facts.

**Inference (KB ├i α)**: This means that sentence α can be derived from KB using a specific inference procedure `i`.
* **Soundness**: An inference procedure `i` is sound if whenever KB ├i α, it's also true that KB ╞ α (it only derives true things).
* **Completeness**: An inference procedure `i` is complete if whenever KB ╞ α, it's also true that KB ├i α (it can derive all true things).

---
## Propositional Logic (Boolean Logic): The Basics

**Propositional logic** is the simplest form of logic and serves as a good starting point. It assumes the world contains facts that are either true or false.

### Syntax of Propositional Logic

* **Proposition Symbols**: $P_1, P_2$, etc., represent basic facts (e.g., $P_{1,1}$ could mean "there is a pit in square [1,1]"). These are sentences.
* **Logical Connectives**:
    * `¬S`: Negation (NOT S) - true if S is false.
    * `S1 ∧ S2`: Conjunction (S1 AND S2) - true if S1 and S2 are both true.
    * `S1 ∨ S2`: Disjunction (S1 OR S2) - true if S1 or S2 (or both) are true.
    * `S1 ⇒ S2`: Implication (S1 IMPLIES S2) - true if S1 is false or S2 is true (false only when S1 is true and S2 is false).
    * `S1 ⇔ S2`: Biconditional (S1 IF AND ONLY IF S2) - true if S1 and S2 have the same truth value.
    *(All from )*

### Semantics of Propositional Logic

Each **model** in propositional logic assigns a truth value (true or false) to every proposition symbol. For instance, if we have symbols $P_{1,2}$, $P_{2,2}$, $P_{3,1}$, a model could be `{$P_{1,2}$ = false, $P_{2,2}$ = true, $P_{3,1}$ = false}`.
The truth of complex sentences is determined using truth tables for the connectives.

**Wumpus World Sentences in Propositional Logic**:
* Let $P_{i,j}$ be true if there is a pit in square `[i,j]`.
* Let $B_{i,j}$ be true if there is a breeze in square `[i,j]`.
* Observations: `¬P1,1` (no pit in [1,1]), `¬B1,1` (no breeze in [1,1]), `B2,1` (breeze in [2,1]).
* Rule: "Pits cause breezes in adjacent squares." For square [1,1]:
    $B_{1,1} \Leftrightarrow (P_{1,2} \lor P_{2,1})$
    For square [2,1]:
    $B_{2,1} \Leftrightarrow (P_{1,1} \lor P_{2,2} \lor P_{3,1})$

### Reasoning Patterns in Propositional Logic

* **Logical Equivalence**: Two sentences α and β are logically equivalent (α ≡ β) if they are true in the same set of models. This means α ╞ β and β ╞ α. Standard equivalences include De Morgan's laws, commutativity, associativity, etc..
* **Validity**: A sentence is valid if it is true in all possible models (e.g., `A ∨ ¬A`, `A ⇒ A`). The **Deduction Theorem** states: KB ╞ α if and only if (KB ⇒ α) is valid.
* **Satisfiability**: A sentence is satisfiable if it is true in at least one model (e.g., `A ∨ B`). A sentence is unsatisfiable if it's true in no models (e.g., `A ∧ ¬A`). Inference is connected to satisfiability: KB ╞ α if and only if (KB ∧ ¬α) is unsatisfiable.

### Effective Propositional Inference: Making Deductions

How do we derive new sentences from the KB?
* **Application of Inference Rules**: These are legitimate (sound) ways to generate new sentences from old ones. A proof is a sequence of such applications.
* **Model Checking**: Involves enumerating models (e.g., truth table enumeration, which is exponential) or using improved backtracking or heuristic search methods.

#### Resolution

A powerful inference rule for propositional logic, especially when sentences are in **Conjunctive Normal Form (CNF)**. CNF is a conjunction of clauses, where each clause is a disjunction of literals (a proposition symbol or its negation).
* **Resolution Rule**: Given two clauses $(l_1 \lor \dots \lor l_k)$ and $(m_1 \lor \dots \lor m_n)$, where $l_i$ and $m_j$ are complementary literals (e.g., P and ¬P), we can infer:
    $(l_1 \lor \dots \lor l_{i-1} \lor l_{i+1} \lor \dots \lor l_k \lor m_1 \lor \dots \lor m_{j-1} \lor m_{j+1} \lor \dots \lor m_n)$.
    Example: From $(P_{1,3} \lor P_{2,2})$ and $(\neg P_{2,2})$, infer $P_{1,3}$.
* Resolution is **sound and complete** for propositional logic.
* **Conversion to CNF**: Sentences can be converted to CNF using equivalences:
    1.  Eliminate `⇔`: replace α ⇔ β with (α ⇒ β) ∧ (β ⇒ α).
    2.  Eliminate `⇒`: replace α ⇒ β with ¬α ∨ β.
    3.  Move `¬` inwards using De Morgan's rules (e.g., ¬(α ∧ β) ≡ ¬α ∨ ¬β) and double-negation (¬¬α ≡ α).
    4.  Apply distributive law (e.g., α ∨ (β ∧ γ) ≡ (α ∨ β) ∧ (α ∨ γ)) to get CNF.
* **Resolution Algorithm**: Often used in a proof by contradiction: to show KB ╞ α, show that (KB ∧ ¬α) is unsatisfiable (leads to an empty clause, representing a contradiction).

#### Forward and Backward Chaining

These are inference algorithms often used with Horn clauses (clauses with at most one positive literal).

* **Forward Chaining (Data-Driven)**:
    * Starts with known facts in the KB.
    * Fires any rule whose premises (LHS) are satisfied.
    * Adds the rule's conclusion (RHS) to the KB.
    * Repeats until the query is found or no new sentences can be derived.
    * It's like deducing all possible consequences from the given data.
    * Forward chaining is sound and complete for Horn clauses.

* **Backward Chaining (Goal-Driven)**:
    * Starts with the query (the goal) and works backward.
    * Finds rules whose conclusion matches the current goal.
    * The premises of such rules become new sub-goals.
    * Continues until all sub-goals are satisfied by facts in the KB.
    * More focused than forward chaining if the goal is specific.
    * Avoids loops by checking if a new sub-goal is already on the goal stack.

### Agents Based on Propositional Logic

A simple knowledge-based agent can use propositional logic to represent its world and make decisions. For instance, in the Wumpus World, an agent's KB might contain rules like $B_{x,y} \Leftrightarrow (P_{x,y+1} \lor P_{x,y-1} \lor P_{x+1,y} \lor P_{x-1,y})$ (a square is breezy if and only if an adjacent square has a pit). By TELLing the KB its current percepts (e.g., `¬Breeze_1,1`, `Stench_1,2`), the agent can ASK the KB whether a certain square is safe or if an action is appropriate, using inference methods like resolution or chaining.

However, propositional logic has significant limitations in expressive power. It's hard to represent general statements like "all squares adjacent to a pit are breezy" without enumerating it for every square. This leads us to First-Order Logic.

---
## First-Order Logic (FOL): Describing a Richer World
Propositional logic assumes the world contains facts. **First-Order Logic (FOL)**, also known as predicate calculus, provides a much more expressive way to represent knowledge. FOL assumes the world contains:
* **Objects**: People, houses, numbers, colors, games, etc.
* **Relations**: Properties of objects or relationships between them (e.g., red, round, brother of, bigger than).
* **Functions**: Mappings from objects to objects (e.g., father of, one more than).

FOL has well-defined formal semantics and sound and complete inference rules, making it a powerful tool for AI. It allows symbolic structures to represent natural language statements, and inference rules can deduce new structures.

### Syntax and Semantics of First-Order Logic

#### Syntax: The Building Blocks
The basic elements of FOL syntax are:
* **Constants**: Represent specific objects (e.g., `KingJohn`, `2`, `NUS`).
* **Variables**: Represent unspecified objects (e.g., `x`, `y`, `a`).
* **Functions**: Symbols that map one or more objects to a single object (e.g., `Sqrt(x)`, `LeftLegOf(Richard)`). The number of arguments is its arity.
* **Predicates**: Symbols that represent relations among objects or properties of objects (e.g., `Brother(x,y)`, `IsRed(block)`, `>(age1, age2)`). They map objects to truth values (true/false). A 0-ary predicate is a proposition.
* **Terms**: A constant, a variable, or a function applied to terms (e.g., `KingJohn`, `x`, `Father(x)`). A term refers to an object.
* **Atomic Sentences (Atoms)**: Formed by a predicate symbol followed by a parenthesized list of terms (e.g., `Brother(KingJohn, RichardTheLionheart)`) or two terms connected by equality (e.g., `Father(John) = Henry`).
* **Connectives**: Same as in propositional logic: `¬`, `∧`, `∨`, `⇒`, `⇔`. Used to build complex sentences from atomic ones (e.g., `Sibling(KingJohn,Richard) ⇒ Sibling(Richard,KingJohn)`).
* **Equality**: `term1 = term2` is an atomic sentence.
* **Quantifiers**:
    * **Universal Quantifier (∀)**: "For all..." (e.g., `∀x At(x,NUS) ⇒ Smart(x)` means "Everyone at NUS is smart"). Typically, `⇒` is the main connective used with `∀`.
    * **Existential Quantifier (∃)**: "There exists..." (e.g., `∃x At(x,NUS) ∧ Smart(x)` means "Someone at NUS is smart"). Typically, `∧` is the main connective used with `∃`.
* **Auxiliary Symbols**: Parentheses `()`, brackets `[]`, etc., for punctuation.

#### Semantics: Meaning in Models
Sentences in FOL are true with respect to a **model** and an **interpretation**.
* A **model** contains:
    * A set of **objects** (the domain elements).
    * **Relations** among these objects (which predicates are true for which objects).
    * **Functional relations** (the results of functions applied to objects).
* An **interpretation** specifies what each symbol refers to:
    * Constant symbols refer to objects.
    * Predicate symbols refer to relations.
    * Function symbols refer to functional relations.
An atomic sentence `predicate(term1, ..., termn)` is true if the objects referred to by `term1, ..., termn` are in the relation referred to by `predicate`.

* **Universal Quantification (∀x P)**: True in a model if P is true with `x` being each possible object in the model. It's like a conjunction of instantiations: `P(obj1) ∧ P(obj2) ∧ ...`.
* **Existential Quantification (∃x P)**: True in a model if P is true with `x` being at least one object in the model. It's like a disjunction: `P(obj1) ∨ P(obj2) ∨ ...`.

**Properties of Quantifiers**:
* `∀x ∀y` is the same as `∀y ∀x`.
* `∃x ∃y` is the same as `∃y ∃x`.
* `∃x ∀y` is NOT the same as `∀y ∃x`.
    * `∃x ∀y Loves(x,y)`: "There is a person who loves everyone in the world."
    * `∀y ∃x Loves(x,y)`: "Everyone in the world is loved by at least one person."
* **Quantifier Duality**: `∀x P ≡ ¬∃x ¬P` and `∃x P ≡ ¬∀x ¬P`.

**Equality**: `term1 = term2` is true if both terms refer to the same object in the interpretation.

### Using First-Order Logic: Expressing Knowledge

FOL allows us to represent complex knowledge concisely.

**Kinship Domain Examples**:
* "Brothers are siblings": `∀x,y Brother(x,y) ⇔ Sibling(x,y)`
* "One's mother is one's female parent": `∀m,c Mother(c) = m ⇔ (Female(m) ∧ Parent(m,c))`
* "Sibling is symmetric": `∀x,y Sibling(x,y) ⇔ Sibling(y,x)`

**Set Domain Examples**:
* `∀s Set(s) ⇔ (s = {} ) ∨ (∃x,s2 Set(s2) ∧ s = {x|s2})` (A set is empty or an element adjoined to another set)
* `∀x,s1,s2 x ∈ (s1 ∩ s2) ⇔ (x ∈ s1 ∧ x ∈ s2)` (Definition of intersection)

**General Knowledge Examples**:
* "All Pompeians were Romans": `∀x (Pompeian(x) ⇒ Roman(x))`.
* "Marcus was a man": `Man(Marcus)`.
* "Marcus tried to assassinate Caesar": `TryAssassinate(Marcus, Caesar)`. (Ignoring tense for simplicity).
* "Every house has an owner": `∀h House(h) ⇒ ∃o Owner(o,h)`.

**Interacting with FOL KBs**:
Similar to propositional logic, agents use TELL and ASK.
* `TELL(KB, Percept([Smell,Breeze,None],5))` adds a percept.
* `ASK(KB, ∃a BestAction(a,5))` queries for the best action.
The answer might be a **substitution** (or binding list) for variables, like `{a/Shoot}`. If `S = Smarter(x,y)` and `σ = {x/Hillary, y/Bill}`, then `Sσ = Smarter(Hillary,Bill)`.

**Wumpus World in FOL**:
* Perception: `∀t,s,b Percept([s,b,Glitter],t) ⇒ Glitter(t)`
* Reflex: `∀t Glitter(t) ⇒ BestAction(Grab,t)`
* Deducing properties: `∀x,y,a,b Adjacent([x,y],[a,b]) ⇔ [a,b] ∈ {[x+1,y], [x-1,y],[x,y+1],[x,y-1]}`
* Causal rule for breezes: `∀r Pit(r) ⇒ [∀s Adjacent(r,s) ⇒ Breezy(s)]`

### Knowledge Engineering in First-Order Logic ️

Building a knowledge base in FOL is a process called **knowledge engineering**. It generally involves these steps:
1.  **Identify the task**: Understand the problem the agent needs to solve.
2.  **Assemble the relevant knowledge**: Gather information about the domain.
3.  **Decide on a vocabulary**: Choose appropriate constants, predicates, and functions.
4.  **Encode general knowledge about the domain**: Write axioms that capture the rules and relationships.
5.  **Encode a description of the specific problem instance**: Add facts relevant to the particular scenario.
6.  **Pose queries to the inference procedure**: Test the KB and get answers.
7.  **Debug the knowledge base**: Refine axioms and facts based on query results.

---
## Conclusion: The Power of Logical Representation

Logical agents, equipped with knowledge bases and inference mechanisms, demonstrate a powerful way for AI to reason about and interact with the world. Propositional logic provides the basic tools for this reasoning, with methods like resolution and chaining allowing for effective inference. However, its limited expressiveness often necessitates the move to First-Order Logic, which allows us to talk about objects, their properties, and relations between them in a much more nuanced and general way. FOL, coupled with systematic knowledge engineering, enables the creation of sophisticated AI systems capable of complex reasoning and problem-solving.
