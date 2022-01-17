---
title: Regular Languages - Automata, Expressions, and Grammars
description: The smallest interesting class of formal languages, seen four ways - deterministic and nondeterministic finite automata, regular expressions, and regular grammars - plus why they are all equivalent, what they are closed under, and how the pumping lemma and Myhill-Nerode prove a language is not regular.
date: 2022-01-17
draft: false
slug: /automata/regular-languages
tags:
  - Theory of Computation
  - Automata
  - Formal Languages
---

A machine with a fixed, finite amount of memory can still recognise infinitely many strings — "does this input have an even number of `1`s?" needs exactly one bit of state, no matter how long the input runs. The languages such machines accept are the **regular languages**, and they are the floor of the whole theory of computation: the simplest class worth a name.

What makes them worth studying is that four completely different-looking tools — a machine that reads left to right, a machine that guesses, an algebraic notation, and a restricted grammar — all pick out *exactly* this class. This post builds each of the four, shows why they coincide, and then works through the two questions you actually ask about a language: what survives combining regular languages, and how you prove a language is *not* one.

## The vocabulary

An **alphabet** $\Sigma$ is a finite non-empty set of symbols ($\{a,b\}$, or $\{0,1,\dots,9\}$). A **string** is a finite sequence of them; the **empty string** $\lambda$ has length zero. Concatenation just glues strings ($wv$), and $\lambda$ is its identity: $\lambda w = w\lambda = w$. The reversal $w^R$ writes $w$ backwards, and $(uv)^R = v^R u^R$. Powers repeat: $w^0 = \lambda$, $w^{n} = w\,w^{n-1}$.

$\Sigma^*$ is the set of *all* finite strings over $\Sigma$ (including $\lambda$); it is always infinite when $\Sigma$ is non-empty. $\Sigma^+ = \Sigma^* - \{\lambda\}$.

A **language** is any subset of $\Sigma^*$ — nothing more. It can be finite ($\{ab, ba\}$) or infinite ($\{a^n b^n : n \ge 0\}$). Because languages are sets, the set operations apply directly: union, intersection, difference, and complement $\overline{L} = \Sigma^* - L$ (always taken relative to a fixed $\Sigma$). Three operations are specific to languages:

- **Concatenation:** $L_1 L_2 = \{uv : u \in L_1,\; v \in L_2\}$.
- **Kleene star:** $L^* = \bigcup_{i \ge 0} L^i$ where $L^0 = \{\lambda\}$ — zero or more strings from $L$, run together.
- **Positive closure:** $L^+ = \bigcup_{i \ge 1} L^i$, so $L^* = L^+ \cup \{\lambda\}$.

## Deterministic finite automata

A **deterministic finite accepter** (DFA) reads its input once, left to right, and after every symbol it is in exactly one of finitely many states.

Formally it is a 5-tuple $M = (Q, \Sigma, \delta, q_0, F)$: a finite state set $Q$, the alphabet $\Sigma$, a **transition function** $\delta : Q \times \Sigma \rightarrow Q$ that is *total* (defined for every state-symbol pair), a start state $q_0 \in Q$, and a set of accepting states $F \subseteq Q$. It starts in $q_0$; on symbol $a_i$ from state $q$ it moves to $\delta(q, a_i)$; after the last symbol it **accepts** if the current state is in $F$ and **rejects** otherwise.

To talk about the state reached after a whole string, extend $\delta$ to $\delta^* : Q \times \Sigma^* \rightarrow Q$ by recursion on the string: $\delta^*(q, \lambda) = q$, and $\delta^*(q, wa) = \delta(\delta^*(q, w),\, a)$. Then $\delta^*(q_i, w) = q_j$ exactly when the transition graph has a walk labelled $w$ from $q_i$ to $q_j$, and the language of the machine is

$$L(M) = \{w \in \Sigma^* : \delta^*(q_0, w) \in F\}.$$

A language is **regular** if it is $L(M)$ for some DFA $M$. Building one is a matter of deciding what the machine has to *remember*: to accept strings that avoid the substring `001`, three states track "how much of `001` I've just seen" ($\lambda$, `0`, `00`), a fourth is a non-accepting trap you fall into once `001` appears, and every reachable non-trap state is accepting.

## Nondeterministic finite automata

An **NFA** relaxes two rules: from a state, a symbol may lead to a *set* of next states (possibly empty), and the machine may also follow $\lambda$-transitions that consume no input. So $\delta : Q \times (\Sigma \cup \{\lambda\}) \rightarrow 2^Q$.

Read it as the machine exploring every choice at once. $\delta^*(q_i, w)$ is now the *set* of all states reachable from $q_i$ along paths labelled $w$ (with $\lambda$-steps interleaved freely), and the NFA **accepts** $w$ if *any* of those paths ends in $F$:

$$L(M) = \{w : \delta^*(q_0, w) \cap F \neq \emptyset\}.$$

Nondeterminism buys no new languages, but it buys smaller and clearer machines — a union of two languages is just two start-branches joined by $\lambda$-transitions — and it makes closure proofs almost free, as the next sections show.

### NFA and DFA accept the same languages

**Subset construction** turns any NFA $M_N$ into a DFA $M_D$ with $L(M_D) = L(M_N)$. Each DFA state is a *set* of NFA states — "all the places $M_N$ could be right now":

1. The DFA start state is the $\lambda$-closure of $\{q_0\}$ (that state plus everything reachable by $\lambda$-transitions).
2. $\delta_D(\{p_1,\dots,p_k\},\, a)$ = every NFA state reachable from some $p_i$ by reading $a$, then closed under $\lambda$-transitions.
3. A DFA state is accepting if it contains any $p_i \in F_N$.

There are at most $2^{|Q_N|}$ subsets, so the DFA can blow up exponentially — but usually only a handful of subsets are reachable. Here is the construction run on the fly as an acceptor, which is all you need to decide membership:

```python
def accepts(nfa, string):
    # nfa: {"start": s, "accept": set, "delta": {(state, symbol): set_of_states}}
    # symbol "" is the epsilon move
    def eps_closure(states):
        stack, seen = list(states), set(states)
        while stack:
            s = stack.pop()
            for nxt in nfa["delta"].get((s, ""), ()):
                if nxt not in seen:
                    seen.add(nxt)
                    stack.append(nxt)
        return seen

    current = eps_closure({nfa["start"]})
    for ch in string:
        nxt = set()
        for s in current:
            nxt |= nfa["delta"].get((s, ch), set())
        current = eps_closure(nxt)
    return bool(current & nfa["accept"])


# NFA for "ends in ab" over {a, b}: q0 loops on a,b; guesses the final "ab".
nfa = {
    "start": "q0",
    "accept": {"q2"},
    "delta": {
        ("q0", "a"): {"q0", "q1"}, ("q0", "b"): {"q0"},
        ("q1", "b"): {"q2"},
    },
}
print(accepts(nfa, "aabab"))  # True
print(accepts(nfa, "abba"))   # False
```

### Minimising a DFA

For a given regular language there is a *unique* smallest DFA (up to renaming states). Find it by merging states no string can tell apart. States $p, q$ are **indistinguishable** if for every $w$, $\delta^*(p, w) \in F \iff \delta^*(q, w) \in F$; otherwise some $w$ **distinguishes** them.

The `mark` procedure finds the distinguishable pairs:

1. Delete states unreachable from $q_0$.
2. Mark every pair $(p, q)$ with exactly one of them in $F$.
3. Repeat until nothing changes: for each unmarked pair $(p, q)$ and each $a \in \Sigma$, if $(\delta(p,a),\, \delta(q,a))$ is already marked, mark $(p, q)$.
4. Unmarked pairs are indistinguishable. Collapse each equivalence class to one state; a class is accepting if it holds any original accepting state, and transitions carry over class-wise.

## Regular expressions

A **regular expression** describes a regular language algebraically. The primitives are $\emptyset$, $\lambda$, and each $a \in \Sigma$; from expressions $r_1, r_2$ you build $(r_1 + r_2)$, $(r_1 \cdot r_2)$, and $(r_1^*)$, and nothing else is one. The language denoted is defined to match:

$$L(\emptyset) = \emptyset,\quad L(\lambda) = \{\lambda\},\quad L(a) = \{a\},$$
$$L(r_1 + r_2) = L(r_1) \cup L(r_2),\quad L(r_1 \cdot r_2) = L(r_1)\,L(r_2),\quad L(r_1^*) = (L(r_1))^*.$$

Precedence runs star, then concatenation, then union, and the concatenation dot is usually dropped. So $(0+1)^*00(0+1)^*$ is "contains `00`", and $(1+01)^*(0+\lambda)$ is "no two consecutive `0`s" — different expressions can denote the same language.

## The three views are one

**A language is regular if and only if it is accepted by some DFA, accepted by some NFA, or denoted by some regular expression.** Each direction is a construction:

- **Regexp → NFA.** Build tiny NFAs for $\emptyset$, $\lambda$, $a$, then wire them together with $\lambda$-transitions: union is two parallel branches, concatenation is series, star is a branch with a feedback loop back to its start.
- **NFA → DFA.** Subset construction, above.
- **DFA → regexp.** Treat the transition graph as a **generalised transition graph** whose edge labels are regular expressions, and rip out states one at a time. Removing state $q_k$ rewrites every remaining edge $q_p \to q_q$ as $r_{pq} + r_{pk}(r_{kk})^* r_{kq}$ — "go direct, or detour through $q_k$ any number of times". When only the start and one final state remain, the surviving label is the expression.

## What regular languages are closed under

Combine two regular languages and the result is almost always regular:

- **Union, concatenation, star** — immediate from the regular-expression definition.
- **Complement.** Given a DFA for $L$, swap accepting and non-accepting states: $M' = (Q, \Sigma, \delta, q_0, Q - F)$. This needs $\delta$ total and the machine deterministic, which is why you convert to a DFA first.
- **Intersection.** Run two DFAs in lockstep — the **product automaton** has states $Q_1 \times Q_2$, transition $\delta_\cap((p,q), a) = (\delta_1(p,a),\, \delta_2(q,a))$, and accepts when *both* components accept ($F_1 \times F_2$). Or get it free from union and complement by De Morgan.
- **Difference** is $L_1 \cap \overline{L_2}$, so it follows too.
- **Reversal.** Reverse every edge of an NFA for $L$ and swap the roles of start and accepting states.
- **Homomorphism.** If $h$ maps each symbol to a string, apply $h$ to a regular expression for $L$ symbol by symbol to get one for $h(L)$.
- **Right quotient** $L_1 / L_2 = \{x : xy \in L_1 \text{ for some } y \in L_2\}$. Take a DFA for $L_1$ and make state $q$ accepting exactly when some $y \in L_2$ drives $q$ to an original accepting state.

## The pumping lemma

Every DFA has finitely many states, so a long enough accepted string must repeat a state — and the loop between the two visits can be traversed any number of times.

**Pumping lemma.** For every regular $L$ there is a constant $m \ge 1$ such that every $w \in L$ with $|w| \ge m$ splits as $w = xyz$ with $|y| \ge 1$, $|xy| \le m$, and $xy^i z \in L$ for *every* $i \ge 0$ (including $i = 0$, giving $xz$).

Why: take $m$ = number of states in a DFA for $L$. Reading $w$ visits $|w| + 1 \ge m + 1$ states, so by pigeonhole some state $q_r$ repeats within the first $m$ steps. Let $x$ reach the first visit to $q_r$, $y$ be the loop $q_r \to q_r$ (non-empty, and within the first $m$ symbols so $|xy| \le m$), and $z$ the rest. The loop can be taken $i$ times without changing where $z$ lands.

The lemma only proves languages **not** regular, and it is a game against an adversary:

1. Assume $L$ is regular; the adversary hands you $m$.
2. **You** pick a specific $w \in L$ with $|w| \ge m$, chosen to box the adversary in.
3. The adversary picks *any* split $w = xyz$ with $|y| \ge 1$, $|xy| \le m$. You must beat every legal split.
4. **You** pick an $i \ge 0$ (usually $0$ or $2$) making $xy^i z \notin L$.
5. Contradiction — so $L$ is not regular.

Classic slips: choosing $m$ yourself, picking $w$ shorter than $m$, arguing against only one split instead of all of them, picking an $i$ that happens to stay in $L$, or trying to use the lemma to prove regularity.

For $L = \{a^n b^n : n \ge 0\}$: pick $w = a^m b^m$. Any valid split has $|xy| \le m$, so $y$ is all `a`s; pumping to $i = 2$ gives more `a`s than `b`s. Not regular.

## Myhill-Nerode

There is a characterisation that needs no machine. For a language $L$, say $x \equiv_L y$ if for every suffix $z$, $xz \in L \iff yz \in L$ — the two prefixes are interchangeable as far as $L$ can tell.

**Myhill-Nerode theorem.** $L$ is regular if and only if $\equiv_L$ has finitely many equivalence classes. That count is exactly the number of states in the minimal DFA.

This gives a second non-regularity proof — exhibit infinitely many classes. For $\{a^n b^n\}$, the prefixes $a^0, a^1, a^2, \dots$ are pairwise inequivalent (append $b^k$ to tell $a^j$ from $a^k$), so there are infinitely many classes and $L$ is not regular. It is also the theoretical basis for DFA minimisation.

## Decision questions are all easy

For regular languages given as DFAs, every standard question has an algorithm:

| Question | Method | Cost |
| --- | --- | --- |
| $w \in L$? | simulate the DFA on $w$ | $O(|w|)$ |
| $L = \emptyset$? | is any accepting state reachable from $q_0$? (BFS/DFS) | polynomial in states |
| $L$ infinite? | is there a reachable cycle that can still reach an accepting state? | polynomial in states |
| $L_1 = L_2$? | minimise both and check for isomorphism, or test $L_1 \triangle L_2 = \emptyset$ via product + emptiness | polynomial |
| $L_1 \subseteq L_2$? | is $L_1 \cap \overline{L_2} = \emptyset$? | polynomial |

The contrast with context-free languages, where equality and inclusion become *undecidable*, is one of the reasons the regular class is drawn where it is.

## Regular grammars

The fourth view is grammatical. A grammar $G = (V, T, S, P)$ has variables $V$, terminals $T$, a start variable $S$, and productions $P$. It is **right-linear** if every production is $A \rightarrow xB$ or $A \rightarrow x$ with $x \in T^*$ and $B \in V$ — at most one variable, always rightmost. **Left-linear** is the mirror image. A grammar that is entirely one or entirely the other is a **regular grammar**. For example $S \rightarrow abS \mid a$ generates $(ab)^*a$.

**A language is regular if and only if some regular grammar generates it.** Right-linear grammars and NFAs are almost notational variants:

- **Right-linear grammar → NFA.** One state per variable, plus a fresh final state $F$. For $A \rightarrow xB$, run a path labelled $x$ from state $A$ to state $B$ (adding intermediate states if $|x| > 1$); for $A \rightarrow x$, run it from $A$ to $F$; the start state is $S$.
- **DFA → right-linear grammar.** One variable per state, start variable $q_0$. For each transition $\delta(q_i, a) = q_k$ add $q_i \rightarrow a\,q_k$, and for each accepting state $q_f$ add $q_f \rightarrow \lambda$.

Left-linear grammars generate the same class — one way to see it is that reversing the right-hand sides of a left-linear grammar gives a right-linear grammar for $L^R$, and regular languages are closed under reversal.

## The one idea to keep

Every finite-memory recogniser gets you the same languages, and it does not matter whether you write that recogniser as a deterministic machine, a guessing machine, an algebraic expression, or a one-variable grammar — the four notations are interchangeable, and the pumping lemma and Myhill-Nerode are the two levers that pry a language out of the class when it needs unbounded counting. The next step up, [context-free languages](/citadel/automata/context-free-grammar), buys exactly that counting by bolting a stack onto the machine.
