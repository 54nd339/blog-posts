---
title: Structuring Intelligence - Knowledge Representation and Planning in AI
description: Delve into how AI systems represent knowledge about the world through ontological engineering, categories, and actions, and how they devise plans to achieve goals using state-space search and logical approaches.
date: 2023-06-26
draft: false
slug: /pensieve/ai-ml/knowledge-representation
tags:
  - AI/ML
  - CS Basics
---

Hello, AI explorers! We've previously journeyed through the minds of intelligent agents and the search algorithms they use. But how do these agents actually *understand* the world around them? And once they understand it, how do they figure out a sequence of steps to achieve their objectives? That's where **Knowledge Representation (KR)** and **Planning** come into play – two cornerstone concepts in Artificial Intelligence.

Today, we'll explore how AI structures knowledge, from defining categories and objects to understanding actions and events. We'll also touch upon reasoning with incomplete information. Then, we'll switch gears to see how AI agents plan their actions, transforming a problem into a sequence of manageable steps.

## Knowledge Representation: Structuring AI's Understanding
For an AI agent to act intelligently, it needs a way to store and reason about information about its environment and the tasks it needs to perform. Knowledge Representation is concerned with how to formally represent information in a way that a computer system can utilize to solve complex tasks.

### Ontological Engineering: Crafting the Vocabulary of Knowledge

**Ontological engineering** is the process of developing formal representations of knowledge. It involves defining a set of concepts and categories in a specific domain and the relationships between them. Think of it as creating a structured vocabulary and a set of rules for a particular area of knowledge. This allows for shared understanding and reusability of knowledge. Semantic nets and frame systems are early examples of such engineering.

### Categories and Objects: Classifying the World

A fundamental aspect of KR is the ability to represent **categories** (or classes) and **objects** (or instances).
* **Objects** are individual items in the world (e.g., "Tom the cat," "My specific laptop").
* **Categories** are groups of objects that share common properties (e.g., "Cats," "Laptops").

Relationships between objects and categories, and between categories themselves, are crucial:
* An `is_a` link often denotes that an object is an instance of a class (e.g., "Tom `is_a` Cat").
* An `a_kind_of` (ako) link usually signifies that one category is a subclass of another (e.g., "Cat `a_kind_of` Mammal").

**Frames** provide a structured way to represent objects and categories. A frame for an object or category has **slots** that represent its attributes or properties. These slots can be filled with specific values or pointers to other objects (fillers). For instance, a `Cat` frame might have slots like `color`, `sound_made`, and `likes_cream`.

A key feature here is **inheritance**: objects or subclasses inherit properties from their superclasses. If `Mammal` has the property `has_fur`, and `Cat` is a kind of `Mammal`, then `Tom` (who is a `Cat`) inherits the `has_fur` property.

### Actions, Situations, and Events: Representing Dynamics

Representing what happens in the world – actions, situations, and events – is vital for an agent that needs to interact with or reason about a dynamic environment.
One way to represent actions in KR systems like semantic networks is through **reification**, where an action or event itself is treated as an object. This "action object" can then have various roles or cases associated with it, describing who did what to whom, where, when, and how. Common cases include:
* **Agent/Subject**: The entity performing the action (e.g., *John* in "John gave the book to Mary").
* **Object/Patient**: The entity to which the action is done (e.g., *the book*).
* **Recipient/Dative**: The entity receiving the object or effect of the action (e.g., *Mary*).
* **Instrumental**: The means by which an action is done (e.g., *post* in "John sent the book by post").
* **Locative**: The place where an action occurs (e.g., *school* in "John gave the book to Mary at school").

**Scripts** are a related concept, representing stereotypical sequences of events or actions. For example, a "restaurant script" might detail the typical events: entering, being seated, ordering, eating, paying, and leaving.

### Mental Events and Mental Objects: A Glimpse into AI's Mind?

Representing an agent's internal "mental" state – its beliefs, desires, intentions, knowledge – is a complex but crucial aspect of KR, especially for sophisticated agents. While the provided texts don't delve deeply into specific formalisms for "mental events and mental objects," the knowledge base (KB) of a logical agent serves as a representation of what the agent knows or believes to be true. Model-based reflex agents also maintain an internal state which is their representation of unobserved aspects of the world. The challenge lies in how this knowledge leads to intentions and goal-directed behavior.

### The Internet Shopping World: A KR Example

Let's consider an **internet shopping world** to illustrate KR concepts:
* **Categories**: `Product`, `Customer`, `Order`, `ShoppingCart`, `PaymentMethod`.
* **Objects**: `LaptopX200` (an instance of `Product`), `Customer123` (an instance of `Customer`).
* **Attributes (Slots & Fillers)**: `LaptopX200` might have slots `price: $999`, `brand: "TechGiant"`, `screen_size: "14-inch"`. `Customer123` might have `shipping_address: "123 AI Lane"`.
* **Actions/Events**: `addToCart(Customer, Product)`, `placeOrder(Customer, ShoppingCart)`, `makePayment(Order, PaymentMethod)`. These can be reified, having attributes like `timestamp`, `status`.
* **Situations**: The state of a customer's shopping cart at a point in time, or the status of an order.

### Reasoning Systems for Categories

The primary reasoning mechanism for categories is **inheritance**. If "Electronics" is a category with a property `requires_power: true`, and `Laptop` is a subclass of `Electronics`, then an instance `MyLaptop` inherits `requires_power: true`. This allows for efficient storage and inference of properties.

### Reasoning with Default Information: Handling Exceptions

Often, properties are typical rather than universal. For example, "birds fly." However, penguins are birds, and they don't fly. This is handled by **default reasoning** and **overrides**.
* A class can have a **default value** for an attribute (e.g., `Birds` class, `can_fly: true`).
* A subclass or instance can **override** this default value (e.g., `Penguin` subclass, `can_fly: false`).
This makes the knowledge representation more flexible and realistic. A challenge arises with **multiple inheritance** if an object inherits conflicting default properties from different superclasses (the "Nixon diamond" problem, where Nixon was both a Quaker (typically pacifist) and a Republican (typically not pacifist)). The order of inheritance or more specific rules might be needed to resolve such conflicts.

### Truth Maintenance Systems (TMS)

When an agent's beliefs change due to new information, a KB might become inconsistent. **Truth Maintenance Systems (TMS)** are designed to manage dependencies between facts in a KB and help maintain consistency when new information is added or old information is retracted. While the provided documents don't detail TMS, they are an important area in KR for handling dynamic belief systems.

---
## Planning: Charting the Course of Action ️

Once an agent can represent knowledge about its world, it often needs to **plan** – to find a sequence of actions that will lead it from an initial state to a desired goal state. We'll focus on **classical planning environments**, which are fully observable, deterministic, finite, static, and discrete.

### The Planning Problem Defined

A planning problem is typically described by:
1.  An **Initial State**: A description of the starting world, often as a conjunction of positive literals (e.g., `On(A,Table) ∧ Clear(A)`).
2.  A **Goal**: A partially specified state that the agent wants to achieve, also a conjunction of positive literals (e.g., `On(A,B) ∧ On(B,C)`).
3.  A set of **Actions**: Each action is specified by:
    * **Preconditions**: A conjunction of positive literals that must be true before the action can be executed.
    * **Effects**: A conjunction of literals describing how the state changes when the action is executed (some literals become true, some false).
    The STRIPS (Stanford Research Institute Problem Solver) representation is a common way to define actions. For example:
    `Action(Move(block, from, to)`
    ` PRECOND: On(block, from) ∧ Clear(block) ∧ Clear(to)`
    ` EFFECT: On(block, to) ∧ Clear(from) ∧ ¬On(block, from) ∧ ¬Clear(to))`
A **solution** is a sequence of actions that, when executed from the initial state, results in a state that satisfies the goal.

### Planning with State-Space Search

Planning can be approached as a state-space search problem.
* **States**: The configurations of the world.
* **Actions**: The operators that transform one state into another.
* **Initial State**: The starting configuration.
* **Goal Test**: Checks if a state satisfies the goal description.

The search algorithm explores paths from the initial state to find one that reaches a goal state. The actions along this path constitute the plan.

**Totally Ordered vs. Partially Ordered Planning**:
* **Totally ordered plans** are strict linear sequences of actions. The blocks-world example of stacking A on B, and B on C, might yield a plan like: `[Move(B, Table, C), Move(A, Table, B)]`.
* **Partially ordered planning** allows actions to be in the plan without specifying their exact order relative to all other actions, as long as causal links are respected. For example, when putting on shoes and socks, the plan for the right foot (`[RightSock, RightShoe]`) and the plan for the left foot (`[LeftSock, LeftShoe]`) can be determined somewhat independently, and then interleaved in several valid ways (e.g., RightSock, LeftSock, RightShoe, LeftShoe OR RightSock, RightShoe, LeftSock, LeftShoe).

### Planning Graphs

Planning graphs are a powerful data structure used in some advanced planning algorithms (like GraphPlan) to efficiently estimate the reachability of literals and actions from an initial state. They help in:
* Determining if a goal is possibly achievable.
* Providing better heuristic estimates for guiding search in state-space planners.
* Detecting mutex (mutually exclusive) relationships between literals or actions at the same level of the graph, which helps prune the search space.

### Planning with Propositional Logic

Another approach to planning involves translating the planning problem into a **propositional logic satisfiability problem (SAT)**.
* **States** are represented by a set of propositions that are true in that state.
* **Actions** are represented by rules that define how propositions change. For an action A to be executed at time t, its preconditions must be true at t, and its effects will be true at t+1.
* The **initial state** is a set of true propositions at time 0.
* The **goal** is a set of propositions that must be true at some future time step T.

The problem then becomes finding a sequence of actions (i.e., a satisfying assignment to action variables over time steps) such that the goal propositions are true at time T, given the initial state and the action descriptions, all expressed as logical sentences. If a satisfying assignment is found, it corresponds to a valid plan. This allows powerful SAT solvers to be used for planning.

---
## Conclusion: The Intelligent Framework

Knowledge Representation and Planning are the twin pillars that allow AI systems to not just perceive but to *understand* and *act purposefully* in their environments. By structuring information into meaningful categories, objects, and relations, and by devising sequences of actions to achieve goals, AI agents move beyond simple reflexes to exhibit more sophisticated forms of intelligence. From the logical deductions in the Wumpus World to complex real-world scheduling and decision-making, KR and Planning are fundamental to the AI endeavor.
