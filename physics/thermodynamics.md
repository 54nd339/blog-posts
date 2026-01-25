---
title: Thermodynamics - The Universe's Energy Rules & Why You Can't Win (But Can Break Even!)
description: Ever wondered why your coffee cools but never heats up on its own? Or why you can’t build a perfect engine? This post unravels entropy, heat transfer, and why time only moves forward. Click in—before your curiosity dissipates as wasted energy!
date: 2023-02-07
draft: false
slug: /pensieve/physics/thermodynamics
tags:
  - Physics
  - Thermodynamics
---

Hello, heat hackers and energy explorers! Ever wondered why your ice cream melts on a hot day but never spontaneously refreezes? Or why, no matter how clever your Rube Goldberg machine, you can't seem to get more energy out than you put in? These aren't just kitchen conundrums or engineering headaches; they're deep truths about how the universe operates, governed by the powerful laws of **Thermodynamics**.

Thermodynamics is the grand science of energy – its transformations, its movements, and its limitations. It deals with concepts like heat, work, temperature, and the rather mysterious but all-important entropy. It explains everything from how a car engine works to why stars shine, and even why time seems to flow in only one direction. So, let's get fired up and explore the fundamental rules that keep the universe's energy accounts in order!

---
## The Foundation: Thermal Equilibrium and Temperature – Getting Cozy

Before we talk about energy moving around, we need to understand when things are "thermally settled."

* **Thermal Equilibrium:** Imagine two objects. If you put them in contact, and no net heat flows between them, they are said to be in **thermal equilibrium**. It's like they've reached a mutual understanding of "hotness." They are at the same temperature. If System A is in thermal equilibrium with System B, we can write this as $A \leftrightarrow B$.
* **Temperature (T): The Official Hotness/Coldness Scorecard**
    **Temperature** is a fundamental property that tells us how hot or cold something is. More technically, it's a measure related to the average kinetic energy of the particles within a system.
    * **Celsius (°C):** Based on water freezing at 0°C and boiling at 100°C (at standard pressure).
    * **Fahrenheit (°F):** Water freezes at 32°F and boils at 212°F. Commonly used in the US.
    * **Kelvin (K):** The **absolute temperature scale** used in science. $0 \text{ K}$ is **absolute zero**, the coldest possible temperature where, classically, all thermal motion would cease. No negative Kelvin temperatures!
    **Conversions:**
    $T(\text{K}) = T(^\circ\text{C}) + 273.15$
    $T(^\circ\text{F}) = \frac{9}{5}T(^\circ\text{C}) + 32$

---
## The Zeroth Law: The Social Network of Temperature

This law might seem obvious, but it's crucial for actually defining and measuring temperature.
**Zeroth Law of Thermodynamics:** If two thermodynamic systems are each in thermal equilibrium with a third system, then they are in thermal equilibrium with each other.
$$ \text{If } A \leftrightarrow C \quad \text{and} \quad B \leftrightarrow C, \quad \text{then } A \leftrightarrow B $$
This law allows us to define temperature scales and use thermometers. If your thermometer (System C) is in equilibrium with your coffee (System A) and reads 70°C, and then it's in equilibrium with your soup (System B) and also reads 70°C, then your coffee and soup are at the same temperature and in thermal equilibrium with each other.

---
## Energy on the Move: Heat and Work – The Dynamic Duo

Energy can be transferred between a system and its surroundings in two primary ways:

* **Heat (Q): Energy Transfer Due to Temperature Difference**
    **Heat** is thermal energy transferred from a hotter system to a colder system *because* of their temperature difference. It's energy in transit. If there's no temperature difference, there's no net heat flow (in the absence of other work).
    The SI unit of heat is the **Joule (J)**.
    *Convention:* $Q > 0$ when heat is added *to* the system; $Q < 0$ when heat leaves the system.

* **Work (W): Energy Transfer via Macroscopic Forces**
    **Work** is energy transferred when a force acts over a distance. Examples include a gas expanding and pushing a piston, or a stirrer doing work on a liquid.
    The SI unit of work is also the **Joule (J)**.
    *Convention (Common in Physics/Chemistry for the First Law $Q = \Delta U + W_{by\_sys}$ or $\Delta U = Q - W_{by\_sys}$):
        $W > 0$ when work is done *by* the system *on* the surroundings (e.g., gas expands).
        $W < 0$ when work is done *on* the system *by* the surroundings (e.g., gas is compressed).
    *(Note: The original content for this blog stated for the First Law: "$W$ is the work done on the system (positive when work is done on the system)". If this convention is used, then for an expanding gas, $W$ would be negative. We will be careful with the context.)* Let's use the convention: **$W$ is work done *by* the system.**

---
## The Internal Energy: The System's Hidden Reservoir

* **Internal Energy (U):** This is the sum of all the microscopic energies within a system – the kinetic energies of its molecules (translation, rotation, vibration) and the potential energies associated with intermolecular forces and bonds. It's a state function, meaning it depends only on the current state of the system (like its T, P, V), not on how it got there. For an ideal gas, $U$ depends only on temperature.

---
## The First Law: Energy Can't Be Created or Destroyed (Just Transformed!)

This is essentially the **law of conservation of energy** applied to thermodynamic systems. It states that the change in a system's internal energy ($\Delta U$) is equal to the heat ($Q$) added *to* the system minus the work ($W$) done *by* the system.
$$ \Delta U = Q - W $$
Or, in differential form: $dU = dQ - dW$.
If using the convention where $W$ is work done *on* the system (let's call it $W_{on}$), then $dW_{on} = -dW_{by\_sys}$, so the first law becomes $\Delta U = Q + W_{on}$.
$$ \Delta U = Q - W_{by\_system} $$
This law means that energy is always accounted for; it just changes form or location.

---
## The Second Law: Chaos Reigns & Time Marches On! (The Arrow of Time)

The First Law tells us energy is conserved, but it doesn't tell us *why* some processes happen spontaneously (like ice melting in a warm room) while others don't (like a warm room spontaneously making ice cubes appear from water). That's the job of the Second Law!
It introduces the concept of **Entropy (S)**, which is a measure of the **disorder, randomness, or "spread-out-ness"** of energy in a system.

**Second Law of Thermodynamics (Common Statements):**
* "The total entropy of an isolated system can only increase over time or remain constant in ideal cases where the system is in a steady state or undergoing a reversible process."
* "Heat cannot spontaneously flow from a colder body to a hotter body without external work being done." (Clausius statement)
* "It is impossible to construct a device that operates in a cycle and produces no effect other than the transfer of heat from a cooler body to a hotter body." (Also Clausius)
* "It is impossible for any device that operates on a cycle to receive heat from a single reservoir and produce a net amount of work." (Kelvin-Planck statement – you can't have a 100% efficient heat engine that converts all heat into work in a cycle).

Mathematically, for any process occurring in an isolated system (or for the universe as a whole):
$$ \Delta S_{\text{universe}} = \Delta S_{\text{system}} + \Delta S_{\text{surroundings}} \ge 0 $$
* For a **spontaneous (irreversible) process**: $\Delta S_{\text{universe}} > 0$. (Disorder increases!)
* For a **reversible process** (an idealized, perfectly efficient process): $\Delta S_{\text{universe}} = 0$.
The Second Law gives us the "arrow of time" – natural processes tend to move towards states of higher entropy.

---
## The Third Law: The Ultimate Chill – Approaching Absolute Zero

This law deals with the behavior of systems at the theoretical limit of coldness, absolute zero (0 Kelvin or -273.15 °C).
**Third Law of Thermodynamics:** As the temperature of a system approaches absolute zero, the entropy of the system approaches a minimum or zero value (for a perfect crystal).
$$ \lim_{T \to 0K} S(T) = S_0 $$
For a perfect crystalline substance, $S_0 = 0$.
An important consequence is that **it is impossible to reach absolute zero in a finite number of steps.** You can get incredibly close, but never quite there!

---
## Paths of Transformation: Thermodynamic Processes – How Systems Change State

Thermodynamic systems (like a gas in a cylinder) can undergo various changes or **processes**.

* **Isothermal Process (Constant Temperature, $\Delta T = 0$):**
    Imagine a gas expanding or contracting slowly while in contact with a large heat reservoir that keeps its temperature constant.
    For an ideal gas, internal energy $U$ depends only on $T$. So, if $T$ is constant, $\Delta U = 0$.
    From the First Law ($\Delta U = Q - W$): $0 = Q - W \implies Q = W$.
    The heat absorbed by the gas equals the work done by the gas during expansion.
    Work done *by* an ideal gas during a reversible isothermal expansion from $V_1$ to $V_2$:
    $P = nRT/V$.
    $$ W = \int_{V_1}^{V_2} P dV = \int_{V_1}^{V_2} \frac{nRT}{V} dV = nRT \int_{V_1}^{V_2} \frac{dV}{V} $$
    $$ W = nRT \ln\left(\frac{V_2}{V_1}\right) $$

* **Isochoric (or Isovolumetric) Process (Constant Volume, $\Delta V = 0$):**
    Heating a gas in a rigid, sealed container. Since the volume doesn't change, no P-V work is done: $W = \int P dV = 0$.
    From the First Law: $\Delta U = Q - 0 \implies Q = \Delta U$.
    The heat added goes entirely into increasing the internal energy. For an ideal gas, $\Delta U = nC_v\Delta T$, so $Q = nC_v\Delta T$.

* **Isobaric Process (Constant Pressure, $\Delta P = 0$):**
    Heating a gas in a cylinder with a freely moving piston (so pressure stays constant).
    Work done *by* the gas: $W = \int_{V_1}^{V_2} P dV = P \int_{V_1}^{V_2} dV = P(V_2 - V_1) = P\Delta V$.
    From the First Law: $Q = \Delta U + W = \Delta U + P\Delta V$.
    This heat $Q$ at constant pressure is also given by $Q = nC_p\Delta T$.

    **Relationship Between Molar Heat Capacities ($C_p, C_v$) and $R$ for an Ideal Gas:**
    For an ideal gas, $U$ depends only on $T$, so $dU = nC_v dT$.
    In an isobaric process, $dQ = dU + P dV$.
    Also, $dQ = nC_p dT$ for an isobaric process.
    From $PV=nRT$, at constant $P$, $P dV = nR dT$.
    So, $nC_p dT = nC_v dT + nR dT$.
    Dividing by $n dT$ gives Mayer's relation:
    $$ C_p - C_v = R $$
    Since $U$ is a state function, for any process involving an ideal gas, $\Delta U = nC_v\Delta T$.

* **Adiabatic Process (No Heat Exchange, $Q=0$):**
    This happens when a system is perfectly insulated, or the process occurs so rapidly that there's no time for significant heat transfer.
    From the First Law: $\Delta U = 0 - W \implies \Delta U = -W$.
    Work done *by* the gas results in a decrease in its internal energy (and thus its temperature). Work done *on* the gas increases its internal energy (and temperature).
    For a reversible adiabatic process in an ideal gas, the following relationships hold:
    $$ PV^\gamma = \text{constant} $$
    $$ TV^{\gamma-1} = \text{constant} $$
    $$ P^{1-\gamma}T^\gamma = \text{constant} $$
    Where $\gamma = C_p/C_v$ is the **adiabatic index** (or heat capacity ratio).
    *Derivation of $PV^\gamma = \text{constant}$:*
    $dU = -dW_{by} \implies nC_v dT = -P dV$.
    From $PV=nRT$, $P dV + V dP = nR dT$. So $dT = (P dV + V dP)/(nR)$.
    Substitute $dT$: $nC_v \frac{P dV + V dP}{nR} + P dV = 0$.
    $\frac{C_v}{R}(P dV + V dP) + P dV = 0$.
    Multiply by $R$: $C_v(P dV + V dP) + RP dV = 0$.
    $(C_v+R)P dV + C_v V dP = 0$. Since $C_p = C_v+R$:
    $C_p P dV + C_v V dP = 0$.
    Divide by $C_v PV$: $\frac{C_p}{C_v}\frac{dV}{V} + \frac{dP}{P} = 0 \implies \gamma \frac{dV}{V} + \frac{dP}{P} = 0$.
    Integrating: $\gamma \ln V + \ln P = \text{constant} \implies \ln(V^\gamma P) = \text{constant} \implies PV^\gamma = \text{constant}$.
    Work done *by* the gas in an adiabatic process from state 1 to state 2:
    $W = \int_{V_1}^{V_2} P dV = \int_{V_1}^{V_2} \frac{\text{Const}}{V^\gamma} dV = \text{Const} \left[\frac{V^{1-\gamma}}{1-\gamma}\right]_{V_1}^{V_2}$
    Since Const $=P_1V_1^\gamma = P_2V_2^\gamma$:
    $$ W = \frac{P_2V_2^\gamma V_2^{1-\gamma} - P_1V_1^\gamma V_1^{1-\gamma}}{1-\gamma} = \frac{P_2V_2 - P_1V_1}{1-\gamma} = \frac{nR(T_2-T_1)}{1-\gamma} $$

* **Reversible Process:** An idealized process that can be reversed by an infinitesimal change in conditions, with the system passing through the same intermediate states in reverse. No net change in total entropy.
* **Irreversible Process:** All real-world spontaneous processes. They increase total entropy.
* **Cyclic Process:** A system undergoes a series of processes and returns to its initial state. For a cycle, $\Delta U_{cycle} = 0$. So, from First Law, $Q_{cycle} = W_{cycle}$ (net heat absorbed equals net work done).

* **Polytropic Process:** A generalized reversible process described by $PV^x = \text{constant}$, where $x$ is the polytropic index.
    Work done by gas: $W = \frac{P_2V_2 - P_1V_1}{1-x} = \frac{nR(T_2-T_1)}{1-x}$ (for $x \neq 1$).
    Change in internal energy (ideal gas): $\Delta U = nC_v\Delta T$.
    Heat transferred: $Q = \Delta U + W = nC_v\Delta T + \frac{nR(T_2-T_1)}{1-x}$.
    The molar heat capacity for a polytropic process ($C_x$) is $Q/(n\Delta T)$:
    $$ C_x = C_v + \frac{R}{1-x} $$

---
## Measuring Disorder: Entropy ($S$) – The Universe's Messiness Score

**Entropy ($S$)** is a thermodynamic property that can be thought of as a measure of a system's disorder, randomness, or the number of microscopic arrangements (microstates) that correspond to its macroscopic state.
For a **reversible process**, the infinitesimal change in entropy $dS$ is defined as:
$$ dS = \frac{dQ_{rev}}{T} $$
So, for a finite reversible process from state 1 to state 2: $\Delta S = \int_1^2 \frac{dQ_{rev}}{T}$.

**Entropy Changes for Ideal Gas in Reversible Processes:**
* **Isothermal Process ($T$=constant):** $\Delta U = 0 \implies dQ_{rev} = dW = P dV = (nRT/V)dV$.
    $$ \Delta S = \int \frac{nRT}{VT}dV = nR \int_{V_1}^{V_2} \frac{dV}{V} = nR \ln\left(\frac{V_2}{V_1}\right) $$
* **Isochoric Process ($V$=constant):** $dW=0 \implies dQ_{rev} = dU = nC_v dT$.
    $$ \Delta S = \int_{T_1}^{T_2} \frac{nC_v}{T}dT = nC_v \ln\left(\frac{T_2}{T_1}\right) $$
    Using $P_1V/T_1 = P_2V/T_2 \implies T_2/T_1 = P_2/P_1$: $\Delta S = nC_v \ln(P_2/P_1)$. Let's use $C_v = (\frac{f}{2})R$. It should be $C_v \ln(P_2/P_1)$ if it's actually isochoric.
* **Isobaric Process ($P$=constant):** $dQ_{rev} = nC_p dT$.
    $$ \Delta S = \int_{T_1}^{T_2} \frac{nC_p}{T}dT = nC_p \ln\left(\frac{T_2}{T_1}\right) $$
    Using $PV_1/T_1 = PV_2/T_2 \implies T_2/T_1 = V_2/V_1$: $\Delta S = nC_p \ln(V_2/V_1)$.
* **Adiabatic Process ($Q_{rev}=0$):**
    $$ \Delta S = 0 $$
    (A reversible adiabatic process is also called an isentropic process).

**General Entropy Change for an Ideal Gas:**
$dU = dQ_{rev} - P dV \implies dQ_{rev} = dU + P dV = nC_v dT + (nRT/V)dV$.
$dS = \frac{dQ_{rev}}{T} = nC_v \frac{dT}{T} + nR \frac{dV}{V}$.
Integrating:
$$ \Delta S = nC_v \ln\left(\frac{T_2}{T_1}\right) + nR \ln\left(\frac{V_2}{V_1}\right) $$
Using $R = C_p - C_v$ and ideal gas relations, this can also be written as:
$$ \Delta S = nC_p \ln\left(\frac{T_2}{T_1}\right) - nR \ln\left(\frac{P_2}{P_1}\right) $$

---
## Spontaneity & Free Energy: What Drives Change?

Entropy tells us about the universe's tendency towards disorder, but what about specific chemical reactions or processes under more constrained conditions (like constant temperature and pressure)? Two other thermodynamic potentials help here:

* **Gibbs Free Energy ($G$): Predicting Spontaneity at Constant T & P**
    The Gibbs Free Energy is defined as $G = H - TS$, where $H = U + PV$ is enthalpy.
    The change in Gibbs Free Energy for a process at constant temperature and pressure is:
    $$ \Delta G = \Delta H - T\Delta S $$
    The sign of $\Delta G$ tells us if a process is spontaneous under these conditions:
    * If $\Delta G < 0$: The process is spontaneous (exergonic).
    * If $\Delta G > 0$: The process is non-spontaneous (endergonic); reverse is spontaneous.
    * If $\Delta G = 0$: The system is at equilibrium.
    **Relationship Between $\Delta G$ and Equilibrium Constant ($K$):**
    For a reaction, $\Delta G^o = -RT \ln K_{eq}$ (where $\Delta G^o$ is standard free energy change).
    This leads to the van 't Hoff equation relating $K$ to temperature and enthalpy change $\Delta H^o$:
    $$ \ln K = -\frac{\Delta H^o}{RT} + \frac{\Delta S^o}{R} $$
    Integrating between two temperatures (assuming $\Delta H^o$ and $\Delta S^o$ are constant):
    $$ \ln\left(\frac{K_2}{K_1}\right) = -\frac{\Delta H^o}{R}\left(\frac{1}{T_2} - \frac{1}{T_1}\right) $$

* **Helmholtz Free Energy ($A$ or $F$): Spontaneity at Constant T & V**
    The Helmholtz Free Energy is defined as $A = U - TS$.
    The change in Helmholtz Free Energy for a process at constant temperature and volume is:
    $$ \Delta A = \Delta U - T\Delta S $$
    Under these conditions, a process is spontaneous if $\Delta A < 0$. $\Delta A$ represents the maximum work that can be extracted from a system in a reversible process at constant T and V.

---
## Sharing is Caring: Equipartition of Energy – How Molecules Divide Their Loot

The **Equipartition Theorem** from statistical mechanics gives us a simple way to estimate the average energy stored in each "mode" of motion for molecules in thermal equilibrium.
It states that for each **quadratic degree of freedom** (a way a molecule can store energy that depends on the square of a coordinate or momentum component), the average energy associated with that degree of freedom is $\frac{1}{2}k_B T$ per molecule (or $\frac{1}{2}RT$ per mole).
* A **degree of freedom ($f$)** is an independent way a molecule can possess energy.

| Molecule Type             | Degrees of Freedom ($f$)                                   | Avg. Energy/molecule ($U/N$) | Molar $C_v = (f/2)R$ | Molar $C_p = C_v+R$ | $\gamma = C_p/C_v$   |
| :------------------------ | :--------------------------------------------------------- | :--------------------------- | :------------------- | :------------------ | :------------------- |
| **Monatomic** (e.g., He, Ar) | 3 (translational: x, y, z)                                 | $\frac{3}{2}k_B T$           | $\frac{3}{2}R$       | $\frac{5}{2}R$      | $5/3 \approx 1.67$   |
| **Diatomic** (e.g., O$_2$, N$_2$) (moderate temp.) | 3 (trans) + 2 (rotational, about axes $\perp$ to bond) = 5 | $\frac{5}{2}k_B T$           | $\frac{5}{2}R$       | $\frac{7}{2}R$      | $7/5 = 1.40$         |
| **Polyatomic (linear)** (e.g. CO$_2$) (moderate temp.) | 3 (trans) + 2 (rot) = 5                                    | $\frac{5}{2}k_B T$           | $\frac{5}{2}R$       | $\frac{7}{2}R$      | $7/5 = 1.40$         |
| **Polyatomic (non-linear)** (e.g. H$_2$O) (moderate temp.) | 3 (trans) + 3 (rotational) = 6                             | $3k_B T$                     | $3R$                 | $4R$                | $4/3 \approx 1.33$   |
| **Diatomic/Polyatomic (high temp.)** | Add vibrational modes (2 per mode: KE+PE) e.g., diatomic adds 2, $f=7$ | $\frac{7}{2}k_B T$           | $\frac{7}{2}R$       | $\frac{9}{2}R$      | $9/7 \approx 1.29$   |

(Vibrational modes often "freeze out" at lower temperatures due to quantum effects).

---
## The Perfect Engine (That You Can't Build!): The Carnot Cycle

Sadi Carnot conceived a theoretical thermodynamic cycle that represents the most efficient possible cycle for converting a given amount of thermal energy into work, or vice-versa, operating between two heat reservoirs at temperatures $T_H$ (hot) and $T_L$ (cold). The **Carnot cycle** consists of four reversible processes acting on an ideal gas:

1.  **Isothermal Expansion (at $T_H$):** Gas absorbs heat $Q_H$ from the hot reservoir and expands, doing work $W_1$.
    $Q_H = W_1 = nRT_H \ln(V_2/V_1)$.
2.  **Adiabatic Expansion:** Gas expands further, doing work $W_2$, while its temperature drops from $T_H$ to $T_L$. No heat exchange.
    $W_2 = (P_2V_2 - P_3V_3)/(1-\gamma) = nR(T_H-T_L)/(\gamma-1)$.
3.  **Isothermal Compression (at $T_L$):** Gas is compressed, rejecting heat $Q_L$ to the cold reservoir. Work $W_3$ is done *on* the gas.
    $|Q_L| = |W_3| = nRT_L \ln(V_4/V_3)$. (Note $V_4 < V_3$, so $\ln(V_4/V_3)$ is negative, making $W_3$ negative if defined as work by gas, or positive if work on gas).
4.  **Adiabatic Compression:** Gas is compressed further, work $W_4$ is done *on* it, and its temperature rises from $T_L$ back to $T_H$. No heat exchange.
    $W_4 = nR(T_L-T_H)/(\gamma-1)$. (Work by gas is negative).

**Net Work Done ($W_{net}$):** The net work done *by* the engine in one cycle is $W_{net} = W_1 + W_2 + W_3 + W_4$. (Careful with signs for $W_3, W_4$).
More simply, $W_{net} = Q_H - |Q_L|$.
**Efficiency ($\eta$):** The thermal efficiency of a heat engine is (Net Work Output) / (Heat Input from Hot Reservoir).
$$ \eta = \frac{W_{net}}{Q_H} = \frac{Q_H - |Q_L|}{Q_H} = 1 - \frac{|Q_L|}{Q_H} $$
For a Carnot cycle using an ideal gas, it can be shown (using $TV^{\gamma-1}=\text{constant}$ for adiabatic steps to relate $V_2/V_1$ and $V_3/V_4$) that $|Q_L|/Q_H = T_L/T_H$.
Therefore, the **Carnot efficiency** is:
$$ \eta_{Carnot} = 1 - \frac{T_L}{T_H} $$
This is the maximum possible efficiency for *any* heat engine operating between temperatures $T_H$ and $T_L$. No real engine can exceed this due to irreversibilities.

---
## Beyond the Basics: The Symphony Continues...

Thermodynamics is a vast field with connections stretching throughout science and engineering.
* **Heat Engines, Refrigerators, Heat Pumps:** Practical devices whose operation and limits are defined by thermodynamic laws.
* **Entropy and Information Theory:** Deep conceptual links exist between thermodynamic entropy and the measure of information (or uncertainty) in a system.
* **Statistical Mechanics:** This branch provides the microscopic, statistical underpinning for the macroscopic laws of thermodynamics, connecting atomic/molecular behavior to bulk properties like temperature and entropy.
* **Chemical Thermodynamics:** Application to chemical reactions, equilibria, and spontaneity.
* **Astrophysics & Cosmology:** Thermodynamics governs stellar evolution, the cosmic microwave background, and the fate of the universe.

The study of thermodynamics continues to illuminate how energy shapes our universe and drives the processes within it. For further exploration, resources like Wikipedia provide a wealth of detailed information on advanced concepts and equations.

---
## Key Takeaways: Your Thermodynamics Pocket Dictionary

This journey through thermodynamics has revealed the fundamental rules governing energy, its transformation, and its flow. Here are the essential principles:

* **Zeroth Law:** Defines thermal equilibrium and allows for temperature measurement. If A=C and B=C, then A=B.
* **First Law ($\Delta U = Q - W_{by\_system}$):** Energy is conserved. The change in a system's internal energy is the heat added to it minus the work it does.
* **Second Law ($\Delta S_{universe} \ge 0$):** Total entropy (disorder) of an isolated system tends to increase. This dictates the direction of spontaneous processes and the "arrow of time."
* **Third Law ($S \to 0$ as $T \to 0K$):** Absolute zero is unattainable, and entropy approaches a minimum (zero for perfect crystals) at this limit.
* **Thermodynamic Processes:** Systems change state via Isothermal (const T), Isochoric (const V), Isobaric (const P), and Adiabatic (no Q) processes, each with specific energy and work relations (e.g., $PV^\gamma=$const for adiabatic).
* **Entropy ($dS = dQ_{rev}/T$):** A measure of disorder. Entropy changes can be calculated for various processes.
* **Free Energy (Gibbs $G=H-TS$, Helmholtz $A=U-TS$):** Potentials that determine the spontaneity of processes under different conditions (e.g., constant T&P for Gibbs). $\Delta G = -RT\ln K_{eq}$.
* **Equipartition of Energy:** In thermal equilibrium, energy is shared equally among a molecule's quadratic degrees of freedom ($\frac{1}{2}k_BT$ each). This determines specific heats ($C_p-C_v=R$).
* **Carnot Cycle:** The most efficient theoretical heat engine cycle operating between two temperatures, with efficiency $\eta = 1 - T_L/T_H$.

Thermodynamics is the ultimate energy accountant, ensuring the universe's books are always balanced, even if it means things inevitably get a bit messier over time!