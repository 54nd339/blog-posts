---
title: Artificial Intelligence - Rational Agents and Their Environments
description: What "AI" actually names, the four ways people have defined it, and the agent framework the modern field is built on - percepts and actions, rationality versus omniscience, PEAS task descriptions, the properties of an environment, and the five kinds of agent program.
date: 2023-01-12
draft: false
slug: /artificial-intelligence/ai
tags:
  - Artificial Intelligence
  - Search
  - Machine Learning
---

An **agent** is anything that senses its surroundings and acts on them: a thermostat, a chess program, a person, a Mars rover. Artificial intelligence, stripped to one sentence, is the engineering of agents that pick *good* actions — actions that, given what the agent knows and perceives, can be expected to achieve its goals.

That framing hides a lot of history and a lot of choices. This post is the map: how the field has defined "intelligent", where its ideas came from, and the vocabulary — percepts, rationality, PEAS, environment types, agent architectures — that the rest of the AI posts assume you have.

## Four definitions, two axes

People have defined AI along two axes: whether a system should **think** or **act**, and whether the standard is **human** performance or **rational** performance. That gives four cells.

- **Thinking humanly** — cognitive modelling: build programs whose internal steps match what a human brain actually does, and check them against psychology experiments.
- **Acting humanly** — the Turing Test: a machine passes if an interrogator conversing with it by teleprinter can't reliably tell it from a person.
- **Thinking rationally** — the "laws of thought": formalise correct reasoning, tracing back to Aristotle's syllogisms and modern logic.
- **Acting rationally** — do the thing expected to best achieve your goals given your information. This is the definition modern AI mostly works from, because it's the most general: correct reasoning is one way to act rationally, but reflexes with no reasoning at all can be rational too.

A separate distinction: **weak AI** aims for machines that *act* intelligently, whatever is going on inside; **strong AI** aims for machines that genuinely *are* minds. Almost all working AI is weak AI, and the acting-rationally standard is why — it asks for behaviour, not consciousness.

## Where the ideas came from

AI didn't appear from nothing. Each contributing field handed it a piece:

- **Philosophy** — logic, rules of inference, the idea that a mind could be a physical system operating by rules.
- **Mathematics** — formal proof, computability and its limits, complexity (tractable vs intractable), and probability for reasoning under uncertainty.
- **Economics** — utility and decision theory: how to choose when outcomes are uncertain and preferences must be traded off.
- **Neuroscience** — evidence about the physical substrate: how neurons and brains actually process information.
- **Psychology** — perception, motor control, and the experimental method for studying cognition.
- **Control theory** — designing systems that steer toward an objective over time using feedback.
- **Linguistics** — knowledge representation and the structure of grammar.
- **Computer engineering** — the fast machines any of this runs on.

A rough timeline of the field itself:

| Period | What happened |
| --- | --- |
| 1943 | McCulloch and Pitts model a neuron as a Boolean circuit |
| 1950 | Turing's "Computing Machinery and Intelligence" proposes the imitation game |
| 1956 | The Dartmouth workshop names the field "Artificial Intelligence" |
| 1950s–60s | Early programs: Samuel's checkers, Newell and Simon's Logic Theorist; Robinson's resolution (1965) |
| 1966–73 | Reality check on computational complexity; neural-net research stalls |
| 1969–79 | Knowledge-based ("expert") systems |
| 1980s | AI becomes an industry |
| 1986 on | Neural networks revive (backpropagation) |
| 1987 on | AI becomes more experimental and statistical; machine learning rises |
| 1995 on | Focus shifts to reasoning under uncertainty and the agent view |

## What it does now

AI stopped being a demo and became infrastructure:

- **Games** — Deep Blue beat Kasparov in 1997; Watson won *Jeopardy!* in 2011.
- **Autonomous vehicles and craft** — early highway self-driving ("No Hands Across America" drove ~98% of a coast-to-coast route autonomously); NASA runs on-board autonomous planning for spacecraft.
- **Logistics** — the 1991 Gulf War used AI planning for a deployment of up to 50,000 vehicles, cargo, and people.
- **Diagnosis and imaging** — clinical decision support, computer-aided reading of medical images.
- **Everywhere else** — fraud detection, algorithmic trading, scheduling, spam filtering, speech interfaces, machine translation, and control loops (fuzzy-logic controllers in car transmissions, appliances).

## Agents: perceive, decide, act

An agent perceives its environment through **sensors** and acts through **actuators**. A person's sensors are eyes and ears; actuators are hands, legs, voice. A software agent's sensors might be network packets or file contents; its actuators, screen output or outbound packets.

Some vocabulary that recurs everywhere:

- **Percept** — the agent's sensory input at one instant.
- **Percept sequence** — the full history of everything it has perceived.
- **Agent function** — the map from percept sequence to action, $f : P^* \rightarrow A$. It's the mathematical description of the agent's behaviour.
- **Agent program** — the concrete implementation of $f$ that runs on the hardware.
- **Performance measure** — the external criterion for success. For a vacuum robot: dirt removed, time taken, energy used. Choosing this well matters: reward "dirt collected" and a clever agent dumps dirt out to suck it up again.

## Rationality is not omniscience

A **rational agent** selects, for each percept sequence, an action expected to maximise its performance measure, given the evidence from that sequence plus whatever knowledge is built in.

Four things determine what's rational at a given moment: the performance measure, the agent's prior knowledge of the environment, the actions available, and the percept sequence so far.

Rationality is bounded by what the agent can know. An **omniscient** agent knows the actual outcome of every action — impossible in practice. Crossing a road after checking both ways is rational even if a meteor then flattens you; the meteor doesn't make the decision irrational. Two habits push a rational agent further: **information gathering** (acting to improve future percepts — turning your head before crossing), and **learning** from experience. An agent whose behaviour is shaped by its own experience rather than only its designer's assumptions is **autonomous**.

## PEAS: pinning down the task

Before designing an agent you specify its task environment with four things — **P**erformance measure, **E**nvironment, **A**ctuators, **S**ensors:

| Agent | Performance | Environment | Actuators | Sensors |
| --- | --- | --- | --- | --- |
| Automated taxi | Safe, fast, legal, comfortable, profitable | Roads, traffic, pedestrians, passengers | Steering, accelerator, brake, horn, signals | Cameras, sonar, GPS, speedometer, engine sensors |
| Medical diagnosis | Healthy patient, low cost, few lawsuits | Patient, hospital, staff | Screen (questions, tests, diagnoses) | Keyboard entry of symptoms and answers |
| Part-picking robot | Fraction of parts in the right bin | Conveyor, parts, bins | Jointed arm and gripper | Camera, joint-angle sensors |
| Vacuum cleaner | Cleanliness, efficiency, battery life | Floor, carpet, obstacles | Wheels, brushes, suction | Dirt sensor, cliff sensor, bump sensor |

## The environment shapes the agent

How hard the task is depends on the environment's properties, which come in pairs:

- **Fully vs partially observable** — do the sensors capture the complete state, or does the agent have to remember what it can't currently see? (A vacuum with only a local dirt sensor is partially observable.)
- **Deterministic vs stochastic** — is the next state fixed by the current state and the action, or is there randomness? An environment that's deterministic except for other agents is sometimes called *strategic*.
- **Episodic vs sequential** — is each decision independent (classify this part, then the next), or does the current action affect all later ones (chess)?
- **Static vs dynamic** — can the world change while the agent is deliberating? Crossword: static. Taxi driving: dynamic. *Semi-dynamic* if the world holds still but the clock is scoring you.
- **Discrete vs continuous** — finitely many percepts and actions (chess), or a continuum (steering a car).
- **Single-agent vs multi-agent** — is anyone else acting, cooperatively or competitively?
- **Known vs unknown** — does the agent know the rules (the outcomes of actions), or must it learn them? This is about the agent's knowledge, not about observability — the two are independent.

The real world is partially observable, stochastic, sequential, dynamic, continuous, and multi-agent — the hard corner of every axis. That's why so much of AI is about coping with less than full information.

## Five kinds of agent program

`Agent = architecture + program`: the architecture is the machine, the program is the implementation of the agent function. In rough order of sophistication:

1. **Simple reflex** — pick an action from the current percept alone, via condition-action rules ("if car-in-front-braking then brake"). Rational only when the right choice depends on nothing but the present percept.
2. **Model-based reflex** — keep an **internal state** from percept history, updated using a **model** of how the world evolves and how the agent's own actions change it. This handles partial observability.
3. **Goal-based** — carry an explicit **goal** and choose actions that move toward it, often via [search](/citadel/artificial-intelligence/search) or planning. More flexible than reflexes, because the goal is data you can change.
4. **Utility-based** — replace the binary goal with a **utility function** mapping each state to a real number. Now the agent can trade off conflicting goals and weigh likelihood of success against value, choosing the action with the best expected utility.
5. **Learning** — any of the above, wrapped so it improves with experience. Four parts: a **performance element** (the agent as described so far), a **learning element** that improves it, a **critic** that tells the learning element how it's doing against a fixed standard, and a **problem generator** that proposes exploratory actions.

## Where this goes

The rest of the AI series fills in these boxes. [Search](/citadel/artificial-intelligence/search) and [constraint satisfaction](/citadel/artificial-intelligence/csp) are how goal-based agents find action sequences. [Logical agents](/citadel/artificial-intelligence/logical-agents) and [knowledge representation](/citadel/artificial-intelligence/knowledge-rep) are how a model-based agent stores and reasons about the world. [Machine learning](/citadel/artificial-intelligence/ml) is how the learning element does its job. The through-line is always the same question: given what it perceives, what should the agent do?
