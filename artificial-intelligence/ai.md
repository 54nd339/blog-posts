---
title: Unveiling Artificial Intelligence - From Concepts to Intelligent Agents
description: A deep dive into the world of Artificial Intelligence, exploring its foundations, history, current state, and the fascinating realm of intelligent agents, their rationality, and interaction with diverse environments.
date: 2023-06-22
draft: false
slug: /pensieve/ai-ml
tags:
  - AI/ML
  - CS Basics
---

Hey there, AI enthusiasts!  Ever wondered what makes a machine "intelligent"? Or how a robot perceives its surroundings and makes decisions? You've come to the right place! In this post, we'll embark on an exciting journey into the heart of Artificial Intelligence (AI), unraveling its core concepts, tracing its history, and exploring its current cutting-edge applications. Then, we'll dive deeper into the fascinating world of **Intelligent Agents** – the entities that bring AI to life. So, grab your thinking caps, and let's get started!

## Decoding Artificial Intelligence: An Overview

At its core, **Artificial Intelligence (AI)** is a branch of science and engineering focused on understanding intelligent behavior and creating machines that can exhibit such behavior. Think of it as the art and science of making computers do things that, at the moment, humans do better. Intelligence itself is the ability to learn, understand, and deal with new situations, or to apply knowledge and think abstractly.

AI systems are generally enabled by algorithms, restricted by constraints, and aimed at modeling perception, thinking, and action. A classic example often used to illustrate an AI problem is finding the maximum or minimum number formed by a certain number of binary bits. While straightforward for a few bits, the number of combinations (the solution space) grows exponentially, necessitating heuristic approaches – a hallmark of many AI problems.

AI can be viewed along two dimensions: whether it **thinks** or **acts**, and whether it does so **humanly** or **rationally**. This gives us four categories:
* **Thinking Humanly**: This involves cognitive modeling, trying to understand the internal activities of the human brain.
* **Acting Humanly**: This is famously exemplified by the Turing Test, where a machine's ability to exhibit intelligent behavior is judged by its capacity to fool a human interrogator into believing it is also human.
* **Thinking Rationally**: This approach, with roots in Aristotle's logic, focuses on formalizing correct argument and thought processes.
* **Acting Rationally**: This is about "doing the right thing" – selecting actions expected to maximize goal achievement given available information. This is often the preferred definition in modern AI.

A distinction is also made between:
* **Weak AI**: Aims at building machines that *act* intelligently without necessarily *being* intelligent.
* **Strong AI**: Aims at building machines that are genuinely persons, a goal we are still far from achieving.

To achieve its goals, AI generally requires interaction with the real world, reasoning, planning, decision-making, and the ability to learn and adapt to change. The overarching goals of AI can be seen as both scientific (to understand human intelligence) and engineering-focused (to build intelligent agents solving real-world problems). This includes replicating human intelligence and solving knowledge-intensive tasks.

---
## The Pillars of AI: Its Foundations

AI is not a field that sprang out of a vacuum. It stands on the shoulders of giants from various disciplines. Key influences include:

* **Philosophy**: Provided logic, methods of reasoning, the concept of the mind as a physical system, and foundations for learning, language, and rationality.
* **Mathematics**: Offered formal representation, proof algorithms, computation, and theories of (un)decidability, (in)tractability, and probability.
* **Economics**: Contributed concepts of utility and decision theory.
* **Neuroscience**: Studies the physical substrate for mental activity.
* **Psychology**: Explores phenomena of perception, motor control, and experimental techniques.
* **Computer Engineering**: Focuses on building fast computers – the hardware AI runs on.
* **Control Theory**: Deals with designing systems that maximize an objective function over time.
* **Linguistics**: Provides theories of knowledge representation and grammar.

---
## A Journey Through Time: The History of AI

The quest for artificial intelligence has a rich history, marked by significant milestones:

* **1943**: McCulloch & Pitts developed the Boolean circuit model of the brain, a foundational concept.
* **1950**: Alan Turing published "Computing Machinery and Intelligence," introducing the Turing Test.
* **1950s**: This decade saw the "cognitive revolution" and the emergence of early AI programs, including Samuel's checkers program and Newell & Simon's Logic Theorist.
* **1956**: The Dartmouth meeting officially adopted the term "Artificial Intelligence".
* **1965**: Robinson's complete algorithm for logical reasoning was a significant step forward.
* **1966-1973**: AI encountered the challenge of computational complexity, and neural network research saw a decline.
* **1969-1979**: Early development of knowledge-based systems took place.
* **1980s**: AI began to transition into an industry.
* **1986 onwards**: Neural networks experienced a resurgence in popularity.
* **1987 onwards**: AI increasingly became a science, with the rise of Machine Learning (ML).
* **1995 onwards**: The focus shifted towards understanding uncertainty and the emergence of intelligent agents.

---
## AI Today: The State of the Art

Modern AI has achieved remarkable feats and is pervasively integrated into our lives:

* **Game Playing**: Deep Blue defeated world chess champion Garry Kasparov in 1997. IBM's Watson famously won on Jeopardy!.
* **Autonomous Systems**: Self-driving cars have made significant progress, with early examples like "No hands across America" driving autonomously for 98% of the journey. NASA uses on-board autonomous planning for spacecraft operations.
* **Logistics and Planning**: AI was used in the 1991 Gulf War for logistics planning involving up to 50,000 vehicles, cargo, and people.
* **Problem Solving**: AI has proved mathematical conjectures that were unsolved for decades and can solve crossword puzzles better than most humans (e.g., Proverb).
* **Diverse Applications**: AI is used across various fields like medical diagnosis, stock trading, robot control, law, and scientific discovery. Thousands of AI applications are embedded in the infrastructure of virtually every industry.
    * **Finance**: Banks use AI for organizing operations, investment, property management, and fraud detection. Automated online assistants handle customer support for financial services.
    * **Healthcare**: AI systems help in organizing hospital bed schedules, staff rotations, and providing medical information. They are also used in clinical decision support for diagnosis and computer-aided interpretation of medical images.
    * **Heavy Industry**: Robots are common for dangerous or repetitive jobs, with Japan being a leader in their use and production.
    * **Customer Service**: Automated online assistants and call center technologies leverage AI, particularly Natural Language Processing (NLP) and speech recognition.
    * **Transportation**: Fuzzy logic controllers are used in automatic gearboxes for automobiles.
    * **Telecommunications**: Heuristic search is used for workforce management.
    * **Toys and Games**: AI has been mass-produced for educational and leisure toys since the 1990s, like Tamagotchis and Furby, and is extensively used in video games.
    * **Music**: AI is used in composition, performance, music theory, and sound processing.
    * **Aviation**: The Air Operations Division (AOD) uses AI for rule-based expert systems, combat and training simulators, mission management aids, and tactical decision-making support. AI is also used in aircraft design, like the AIDA system.
    * **News and Publishing**: Companies like Narrative Science use AI to generate news reports and summaries from statistical data.

It's clear that AI is no longer just a futuristic concept; it's a powerful tool shaping our present and future!

---
## Meet the Actors: Intelligent Agents

Now that we have a grasp of AI, let's zoom in on the entities that actually *do* the "intelligent" work: **agents**.

An **agent** is anything that can be viewed as perceiving its environment through **sensors** and acting upon that environment through **actuators**. This cycle of perceiving, thinking, and acting is fundamental to an agent's operation. Agents come in various forms:

* **Human Agents**: We perceive using our eyes, ears, and other organs (sensors), and act using our hands, legs, and vocal tract (actuators).
* **Robotic Agents**: These can have cameras, infrared range finders, or Natural Language Processing (NLP) capabilities as sensors, and various motors as actuators.
* **Software Agents**: They might take keystrokes or file contents as sensory input and act by displaying information on a screen or sending network packets.

An **intelligent agent** is an autonomous entity that uses its sensors and actuators to achieve specific goals within its environment. A thermostat is a simple example. These agents might also learn from their environment to better achieve their goals.

There are four main rules that generally define an AI agent:
1.  It must be able to perceive its environment.
2.  Observations must be used to make decisions.
3.  Decisions should result in an action.
4.  The action taken must be a rational action.

### Agent Terminology ️

Let's clarify some key terms associated with agents:

* **Performance Measure**: This criterion determines how successful an agent is. For a vacuum-cleaner agent, this could be the amount of dirt cleaned, time taken, or electricity consumed.
* **Behavior of Agent**: The actions an agent performs after any given sequence of percepts.
* **Percept**: An agent's perceptual input at a given instance.
* **Percept Sequence**: The complete history of everything an agent has perceived up to a certain point.
* **Agent Function**: This is a map from the percept sequence to an action ($f: P* \rightarrow A$), essentially describing the agent's behavior.

For example, a vacuum-cleaner agent might perceive its location and whether that location is dirty (e.g., `[A, Dirty]`). Its actions could be to move `Left`, `Right`, `Remove` dirt, or do `NoOp` (no operation).

---
## The Quest for Rationality

A core concept in AI is **rationality**. An agent should strive to "do the right thing" based on what it perceives and the actions it can perform. The "right action" is the one that will cause the agent to be most successful, as defined by its performance measure.

Rationality at any given time depends on several factors:
* The agent's prior knowledge of the environment.
* The actions the agent can perform.
* The agent's percept sequence up to that moment.
* The performance measure defining success.

A **rational agent** is one that, for each possible percept sequence, selects an action that is expected to maximize its performance measure, given the evidence from the percept sequence and any built-in knowledge. It's about making the *right* decisions to achieve goals. In AI, especially in reinforcement learning, rational actions are crucial as they often lead to positive rewards, while wrong actions might incur penalties.

It's important to distinguish rationality from **omniscience**. An omniscient agent knows the actual outcome of its actions, which is impossible in reality. Rationality is about making the best decision given what is known. Rational agents also benefit from **information gathering** (acting to modify future percepts for useful information) and the ability to **learn and adapt**. An agent is **autonomous** if its behavior is determined by its own experience and learning.

### PEAS: Specifying the Task Environment
To design a rational agent, we need to specify its task environment. This is commonly done using the **PEAS** framework:

* **P**erformance Measure: How is success defined? (e.g., for an automated taxi: safe, fast, legal, comfortable trip, maximize profits).
* **E**nvironment: Where does the agent operate? (e.g., for a taxi: roads, other traffic, pedestrians, customers).
* **A**ctuators: How does the agent act upon the environment? (e.g., for a taxi: steering wheel, accelerator, brake, signal, horn).
* **S**ensors: How does the agent perceive the environment? (e.g., for a taxi: cameras, sonar, speedometer, GPS, odometer, engine sensors, keyboard).

Let's look at a couple more examples:

| Agent                   | Performance Measure                          | Environment             | Actuators                                      | Sensors                                           |
| :---------------------- | :------------------------------------------- | :---------------------- | :--------------------------------------------- | :------------------------------------------------ |
| Medical Diagnosis System | Healthy patient, minimized costs, lawsuits  | Patient, hospital, staff  | Screen display (questions, tests, diagnoses)  | Keyboard (entry of symptoms, patient's answers)  |
| Part-Picking Robot      | Percentage of parts in correct bins  | Conveyor belt with parts, bins  | Jointed arm and hand          | Camera, joint angle sensors  |
| Vacuum Cleaner          | Cleanness, efficiency, battery life, security  | Room, table, wood floor, carpet, obstacles  | Wheels, brushes, vacuum extractor     | Camera, dirt detection sensor, cliff sensor, bump sensor  |

---
## The Nature of the Environment
The environment an agent operates in significantly influences its design. Environments can be characterized along several dimensions:

* **Fully Observable vs. Partially Observable**: If an agent's sensors give it access to the complete state of the environment at each point in time, it's fully observable. Otherwise, it's partially observable (e.g., a vacuum cleaner with only a local dirt sensor cannot know if other squares are clean). Fully observable environments are easier as the agent doesn't need to maintain an internal state to track the world's history. An environment where an agent has no sensors is called unobservable.
* **Deterministic vs. Stochastic**: If the next state of the environment is completely determined by the current state and the agent's action, it's deterministic. Stochastic environments are random and cannot be completely determined by the agent. Strategic environments are deterministic except for the actions of other agents. In a deterministic, fully observable environment, uncertainty is not a concern.
* **Episodic vs. Sequential**: In an episodic environment, the agent's experience is divided into atomic "episodes" (perceiving and then acting), and the choice of action in each episode only depends on the episode itself. In sequential environments, the current decision can affect all future decisions.
* **Static vs. Dynamic**: If the environment can change while the agent is deliberating, it's dynamic. Static environments are easier because the agent doesn't need to constantly monitor the world while deciding on an action. Taxi driving is dynamic; crossword puzzles are static. An environment is semi-dynamic if it doesn't change with time, but the agent's performance score does.
* **Discrete vs. Continuous**: An environment with a finite number of percepts and actions is discrete (e.g., a chess game). A self-driving car operates in a continuous environment.
* **Single-agent vs. Multi-agent**: If only one agent is operating, it's a single-agent environment. If multiple agents are present, it's a multi-agent environment, which presents different design challenges.
* **Known vs. Unknown**: This refers to the agent's state of knowledge. In a known environment, the outcomes of all actions are known to the agent. In an unknown environment, the agent must learn how it works to perform actions. A known environment can be partially observable, and an unknown one can be fully observable.
* **Accessible vs. Inaccessible**: If an agent can get complete and accurate information about the environment's state, it's accessible (e.g., an empty room where temperature defines the state). Information about a global event on Earth is an example of an inaccessible environment.

The real world is typically partially observable, stochastic, sequential, dynamic, continuous, and multi-agent. Some programs, however, operate in entirely artificial environments confined to keyboard inputs, databases, and screen outputs. The Turing Test environment is a famous example of such an artificial setting.

---
## The Blueprint of an Agent: Structure and Types

The structure of an intelligent agent can be viewed as:
**Agent = Architecture + Agent Program**

* **Architecture**: The machinery that an agent executes on (e.g., the physical robot, the computer).
* **Agent Program**: An implementation of an agent function, which maps percept sequences to actions. The agent program runs on the architecture to produce the agent function *f*.

Agents have internal data structures that are updated with new percepts, and decision-making procedures operate on these structures to generate actions.

There are several basic types of intelligent agents, in increasing order of sophistication:

### 1. Simple Reflex Agents
These agents choose actions based *only* on the current percept. They work by finding a rule whose condition matches the current situation and then performing the associated action (condition-action rules or if-then rules). They are rational only if a correct decision can be made purely on the current percept, meaning their environment is typically fully observable.

*Flow*: Sensors -> What the world is like now? -> Condition-Action Rule -> What action I need to do? -> Effectors.

### 2. Model-Based Reflex Agents
These agents can handle partially observable environments. They maintain an **internal state** to keep track of the parts of the world they can't see directly, based on percept history. They use a **model** of the world (knowledge about "how things happen") to update their internal state and choose actions. The state changes based on actions performed or new percepts.

*Flow*: Sensors -> How is the world like now? -> State (How world evolves, What my actions do) -> Condition-Action Rule -> What actions I need to do? -> Effectors.

### 3. Goal-Based Agents
Knowing the current state isn't always enough; agents often need **goals** that describe desirable situations. Goal-based agents choose actions to achieve these goals. This approach is more flexible than reflex agents because the knowledge supporting decisions is explicitly modeled and can be modified. They may involve search and planning to find action sequences to achieve their goals.

*Flow*: Sensors -> How is the world like now? -> State (How world evolves, What my actions do) -> What happens if I do action A? -> Goals -> What actions I need to do? -> Effectors.

### 4. Utility-Based Agents
Goals alone are not always sufficient, especially when there are conflicting goals or uncertainty about achieving them. **Utility-based agents** choose actions based on a preference (utility) for each state. A **utility function** maps a state to a real number representing the degree of "happiness" or desirability. This allows for rational decisions even with conflicting goals or when weighing the likelihood of success against the importance of a goal. They use a model of the world and a utility function to choose actions that lead to the best expected utility.

*Flow*: Sensors -> How is the world like now? -> State (How world evolves, What my actions do) -> What happens if I do action A? -> Utility (How happy I am by doing action A?) -> What actions I need to do? -> Effectors.

### 5. Learning Agents
A key aspect of intelligence is learning. A **learning agent** can improve its performance over time through experience. This allows agents to operate in unknown environments and become more competent than their initial knowledge allows. Learning agents typically have four conceptual components:

* **Learning Element**: Responsible for making improvements.
* **Performance Element**: Responsible for selecting external actions (this is what we previously considered the whole agent).
* **Critic**: Provides feedback to the learning element on how well the agent is doing with respect to a fixed performance standard.
* **Problem Generator**: Suggests actions that will lead to new and informative experiences, allowing the agent to explore and learn.

*Flow*: Environment -> Sensors -> Performance Element (acts) -> Effectors. Critic (observes rewards/penalties based on performance standard) -> Learning Element (updates performance element based on critic's feedback and problem generator's suggestions for new experiences) -> Problem Generator (suggests new actions).

---
## Key Takeaways

* **AI** is about creating machines that can behave intelligently, often by perceiving their environment, thinking or reasoning, and then acting.
* AI has a rich **history** drawing from philosophy, mathematics, neuroscience, and more, and its applications are now widespread.
* **Intelligent Agents** are the entities that perceive and act. Their design is heavily influenced by their **rationality** and the **nature of the environment** they operate in.
* Agents can range from simple **reflex agents** to more complex **model-based, goal-based, utility-based, and learning agents**, each with different capabilities for handling uncertainty and achieving objectives.

This introduction has just scratched the surface of the vast and exciting field of AI and intelligent agents. There's so much more to explore, from the intricacies of machine learning algorithms to the ethical considerations of advanced AI. Stay curious, and keep learning! 