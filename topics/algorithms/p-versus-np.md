---
title: P versus NP
topic: algorithms
tags: [complexity-theory, NP-completeness, reductions, SAT, computability]
---

# P versus NP

*A conceptual map of computational complexity vocabulary, with emphasis on the asymmetries
that make the field tick.*

---

## 1. The vocabulary stack

Everything is built from these rungs, bottom to top:

- **Alphabet** $\Sigma$ — a finite set of symbols.
- **String** — a finite sequence of symbols; the form an input takes on a Turing
  machine's tape.
- **Language** — a *set of strings*. **In this field, "language" = problem.** It has
  nothing to do with programming languages; the word is inherited from formal-language
  theory, where a "language" is literally a subset of $\Sigma^*$.
- **Class** (P, NP, …) — a *set of languages*, i.e. a collection of problems grouped by
  how much resource it takes to decide them.

### Key reframing: "language" = problem, not algorithm

A **language is the question, frozen as the set of "yes" inputs.** It is static; there
is no computation inside it. The *machine / algorithm* is the separate thing that
*decides* the language.

- **Language** = the problem = the set of yes-instances. (e.g. "all graphs containing a
  Hamiltonian cycle")
- **Machine** = a candidate solver.
- $L \in \mathrm{P}$ = "there exists a polynomial-time solver for $L$."

A loop is a feature of the **solver** (a finite control reusing itself over an unbounded
tape) — file it under "how the machine works," never under "what the language is."

### A decision problem is specifically yes/no

A language is a *set* — you're either in it or not — so it models **decision problems**
only. Optimization problems ("find the shortest tour") are converted to decision form
("is there a tour of length $\le k$?") to fit. The two versions are polynomially
equivalent for this theory's purposes.

---

## 2. Why strings? (The string is coupled to the *machine*, not the *problem*)

The string encoding felt unnecessary — and that instinct is correct. The string is
coupled to the **Turing machine** (its tape holds a sequence of symbols), and the
problem is coupled to the machine only because we *chose* that machine as our yardstick.

Two things the string is genuinely providing:

1. **A notion of input size $n$** — the whole theory is asymptotic ($O(n^k)$), so you
   need a size parameter to take a limit in. String length supplies it.
2. **Finite describability / countability** — instances must be presentable to a machine
   as finite data.

Everything else about the encoding is **bookkeeping**, because the theory is invariant
under polynomial transformations and all sane encodings are polynomially
interconvertible.

### The one place encoding genuinely bites: unary vs. binary

Encode integer $N$ in **binary** → size $\sim \log N$.
Encode it in **unary** ($N$ tally marks) → size $N$ itself.

These are **not** polynomially equivalent. An algorithm polynomial in the *unary* size
can be exponential in the *binary* size. This is exactly the
**polynomial vs. pseudo-polynomial** distinction (e.g. dynamic-programming knapsack is
polynomial in the numeric *value* but exponential in the *bit-length* — "weakly
NP-complete"). You can *see* the gap as physical tape consumption.

---

## 3. The classes P and NP

Both P and NP are **sets of languages** (sets of problems). They differ *only* in which
machine — which yardstick — is allowed in the defining clause:

$$
\mathrm{P} = \{\thinspace L : \text{some \textbf{deterministic} poly-time machine decides } L \thinspace\}
$$

$$
\mathrm{NP} = \{\thinspace L : \text{some \textbf{nondeterministic} poly-time machine decides } L \thinspace\}
$$

Neither class *is* a machine. The machine appears *inside* the defining clause as the
sorting tool, never as the class itself.

### NP, the verifier definition (the honest one)

$$
x \in L \iff \exists\thinspace w,\ |w| \le p(|x|),\ V(x,w)\ \text{accepts}
$$

Reading, glyph by glyph:

- $x$ — the input instance.
- $x \in L$ — "$x$ is a yes-instance."
- $\iff$ — if and only if (both directions).
- $\exists\thinspace w$ — there exists a **witness / certificate** $w$. *This $\exists$ is the
  "N" in NP.*
- $|w| \le p(|x|)$ — the witness is **short** (polynomially bounded *length*).
- $V(x,w)$ accepts — a deterministic **poly-time verifier** $V$ checks $x$ against $w$
  and says yes.
- The commas $\approx$ logical **AND** (the first comma, right after $\exists w$, really
  reads "such that").

**In words:** $x$ is a yes-instance iff there is a short witness that a fast deterministic
checker accepts. NP factors cleanly into **"guess a short witness" + "check it
deterministically fast."**

Both directions of $\iff$ matter: yes-instances *have* proofs (completeness), and the
verifier *cannot be fooled* by a no-instance (soundness).

### Two different polynomial leashes (don't merge them)

- $|w| \le p(|x|)$ — bounds the witness **length** (size of the proof).
- $V$ runs in poly time — bounds the verification **time**.

They're coupled (a fast verifier can only *read* a polynomially-long witness) but they
are different quantities: one is space-like (length), the other time-like (runtime).

---

## 4. "Nondeterministic" — the worst-named concept in the field

**Nondeterministic ≠ random, and ≠ quantum.** Four distinct machine models:

| Model | Branch rule | Read-out | Physical? |
|---|---|---|---|
| **Deterministic** | one legal move per step; single fixed path | the answer | yes (every real computer) |
| **Nondeterministic** | *all* legal moves at once; accept iff **∃** an accepting branch | free existential | **no — pure fiction** |
| **Probabilistic** (BPP) | coin flips; follow *one* sampled path | right with prob. $\ge 2/3$ | yes |
| **Quantum** (BQP) | all paths as **amplitudes** | **measure**: outcome $i$ w.p. $`\lvert\text{amp}_i\rvert^2`$ | yes (in principle) |

The decisive feature of nondeterminism is the **existential quantifier**, evaluated for
free: one good branch among exponentially many *counts*, with no penalty from the others.

- **Randomness** *averages* over branches → one good branch among exponentially many
  gives probability $\approx 0$. Useless. Different from $\exists$.
- **Quantum** carries amplitudes you must turn into probabilities by measurement; one
  tiny-amplitude branch shows up with probability $\approx 0$. So a quantum computer is a
  cousin of the **probabilistic** model, *not* the nondeterministic one. (Quantum
  algorithms work by **interference** — arranging wrong answers to cancel *before*
  measuring — not by reading off a free $\exists$.)

**Mental rename:** the "N" in NP is an **$\exists$**, not a die and not a qubit. Read
"nondeterministic machine" as "**guess-and-check machine**."

**The nondeterministic machine does not exist** and is not meant to — it is a
*definitional device*, a deliberately-unphysical "upper fiction" used to define the class
of guess-and-check problems cleanly. Its non-existence is intrinsic, not an engineering
gap. (The field is full of such lenses: oracle machines, alternating machines, circuit
families.)

---

## 5. P ⊆ NP, and the actual open question

$\mathrm{P} \subseteq \mathrm{NP}$ is **trivial**: if you can *solve* fast, you can
*check* fast — build a verifier that ignores the witness and just runs the solver. So
**solving ⟹ checking** (the easy, proven direction).

**P vs NP** asks the converse: **does checking-fast imply solving-fast?**

$$
\mathrm{P} \overset{?}{=} \mathrm{NP}
\quad\Longleftrightarrow\quad
\text{"is the inclusion } \mathrm{P}\subseteq\mathrm{NP}\text{ strict?"}
\quad\Longleftrightarrow\quad
\text{"does efficient verifiability imply efficient solvability?"}
$$

Three phrasings of one question. The set view asks whether $\mathrm{NP}\setminus\mathrm{P}$
is empty; the capability view asks whether the witness is ever *truly load-bearing* —
whether there's ever a real gap between **recognizing** a solution and **finding** one.

> **The watcher (verifier) is provably in P. Whether the *finder* is in P is the open
> conjecture — almost everyone bets NO (P ≠ NP), but nobody can prove it.**

---

## 6. "Polynomial = tractable" is a proxy, not a guarantee

Polynomials can be large: $n^{100}$ is polynomial and useless; $2^{0.0001 n}$ is
exponential and often fine. P is a **robustness** abstraction (invariant across machine
models), *not* a feasibility guarantee. It deliberately throws away the degree and
constants — exactly the information that decides real-world feasibility.

- A proof that $\mathrm{P}=\mathrm{NP}$ via an $n^{100}$ algorithm would change almost
  nothing practical.
- Conversely, NP-complete problems are solved at industrial scale daily, because
  worst-case hardness says nothing about the instances you actually meet.

**Why the proxy survives:** empirically, when a *natural* problem is shown to be in P at
all, the algorithm almost always turns out to be low-degree. The $n^{100}$ monsters are
nearly always artificial. Finer subfields (*fine-grained* complexity, *parameterized*
complexity) exist precisely because "polynomial vs. not" is too coarse once you're inside
P.

---

## 7. Reductions and the meaning of "hard"

### Karp (many-one, poly-time) reduction

$$
L_1 \le_p L_2 \quad:\Longleftrightarrow\quad
\exists\thinspace \text{poly-time } f \text{ such that } \forall x,\ \big(x \in L_1 \iff f(x) \in L_2\big)
$$

$f$ is a **bridge** that rewrites $`L_1`$-questions as $`L_2`$-questions *without changing the
answer*. The defined object is $f$; $`L_1, L_2`$ are given. With the $\exists$ out front,
the whole statement is a **property of the pair** $`(L_1, L_2)`$ — it holds for some pairs,
not all. The symbol $`\le_p`$ is a genuine, *selective* relation.

**"Hard" means solving-cost, compared via reductions — nothing to do with $|x|$.**

$$
L_1 \le_p L_2 \quad\text{reads}\quad
\text{"a fast solver for } L_2 \text{ yields a fast solver for } L_1\text{"}
\quad=\quad
\text{"} L_1 \text{ is no harder than } L_2\text{."}
$$

Why the theory is built on reductions rather than absolute runtimes: we mostly **can't**
prove absolute lower bounds (nobody can prove SAT needs exponential time). Reductions let
us *compare* difficulty even when both absolute difficulties are unknown. Hardness here is
**relative**, measured **up to polynomial factors**. For a concrete example, see
[Maximum Common Induced Subgraph](maximum-common-induced-subgraph.md), which reduces to
Maximum Clique via the modular product — itself an NP-hard problem.

### Two senses of "universal" in a reduction

1. **$\forall x$ (works on every instance)** — *forced*. A reduction must preserve answers
   on *all* inputs, or it can't faithfully transport a solver.
2. **Universal over the whole class NP** (every $L\in\mathrm{NP}$ reduces to $A$) — a
   *definitional choice*, and it's what defines **NP-hardness**. That a *single* problem
   can be universal for all of NP is **not** obvious a priori — it's the content of
   **Cook–Levin** (SAT is such a problem). A theorem, not a fiat.

**Choice of reduction = a resolution dial.** The reduction must be *weaker* than the
resource you're studying or it dissolves the distinction:
- poly-time reductions → separate P from NP;
- log-space reductions → study structure *inside* P (P-completeness);
- weaker still (AC⁰) → circuit-complexity distinctions.

---

## 8. NP-hard vs. NP-complete (the relationship people get backwards)

Two ingredients:
- **in NP** = has short checkable certificates — a *ceiling*.
- **NP-hard** = everything in NP reduces to it — a *floor* (at least as hard as all of NP).

$$
\boxed{\ \textbf{NP-complete} \thickspace=\thickspace \textbf{NP-hard} \thickspace\cap\thickspace \textbf{NP}\ }
$$

- **NP-hard** = the floor *alone*. No ceiling. Can be much harder than NP — even
  undecidable (the halting problem is NP-hard).
- **NP-complete** = floor **and** ceiling. Pinned at exactly NP's level — the *maximal
  elements of NP*, the hardest problems still possessing short certificates.

**Crucial:** "complete" does **not** mean "more/harder." NP-complete is a **subset** of
NP-hard (it adds the *restriction* "must also be in NP"). The word "complete" is the
logician's sense — *a representative element that captures the whole class via
reductions*. Solve one NP-complete problem in P and **all** of NP collapses into P
($\mathrm{P}=\mathrm{NP}$).

In algorithms practice, "this is NP-hard" is a **routing decision**: *stop looking for an
efficient exact algorithm (it can't exist unless P = NP); pivot* to approximation,
heuristics, exponential-but-clever exact methods, parameterized algorithms, or tractable
special cases. The "probably not polynomial" reading rides on the **assumption** P ≠ NP.

### The NP-complete frontier: edge or region?

- **Edge**, in the order-theoretic sense — the *maximal elements* of NP under $`\le_p`$, a
  single $`\le_p`$-equivalence class (all NP-complete problems inter-reduce, so at poly-time
  resolution they're "one point").
- **Region**, honestly — that frontier is densely populated by *thousands of structurally
  different* problems that only *look* like one point because poly-time reductions are too
  coarse to separate them. Zoom in (finer reductions, actual exponents, average-case
  behavior) and they spread out. The diagram draws a region partly for ink, partly because
  it's not wrong.

---

## 9. The NP-hard "overhang" (NP-hard but not in NP)

No single famous name for the whole set; it **stratifies** into named tiers, going up:

- $`\Sigma_k^p`$ / $`\Pi_k^p`$-complete ($k\ge 2$) — higher levels of the **polynomial
  hierarchy** (alternating $\exists\forall$ quantifiers). e.g. "is this the *optimal*
  tour?" ($\exists$ tour $\forall$ other tours…).
- **PSPACE-complete** — polynomial *space*, possibly exponential time. e.g. **QBF**
  (fully-quantified Boolean formulas), optimal play in generalized chess/Go.
- **EXPTIME-complete** — *provably* requires exponential time (one of the rare
  unconditional lower bounds, via the time hierarchy theorem).
- **Undecidable** — the top. The **halting problem**.

When pointing at the overhang, name the *specific tier* — that's the information "NP-hard
but not in NP" throws away.

---

## 10. The halting problem (the ceiling of the overhang)

**Question:** given a program $P$ and input $x$, does $P(x)$ eventually **halt**, or run
**forever**? (Turing, 1936 — predates complexity theory; about *computability*, not
speed.)

You can *run* $P(x)$ and confirm halting when it happens — but no finite time upgrades
"hasn't halted yet" to "never will." That gap is never closeable in general.

**Proof (diagonalization / self-reference):** Suppose a decider $H(P,x)$ exists (always
halts; says "halts" or "loops" correctly). Build

> $D(Q)$: run $H(Q,Q)$; if it says "$Q(Q)$ halts," **loop forever**; if "loops," **halt.**

Now ask: **what does $D(D)$ do?**
- If $D(D)$ **halts** → $H$ said it loops → contradiction.
- If $D(D)$ **loops** → $H$ said it halts → contradiction.

Either way, contradiction. So $H$ cannot exist — the halting problem is **undecidable**.

**Why it tops the overhang:** it is **not in NP** (the "loops forever" case has *no finite
certificate* — there's nothing of bounded size proving a computation never ends), and it's
NP-hard (everything decidable reduces into it), so it sits infinitely above NP — beyond
computability entirely. The collapse of $\mathrm{P}=\mathrm{NP}$ can't touch it:
undecidable is undecidable at any speed.

**Reach (Rice's theorem):** *any* non-trivial semantic property of program behavior is
undecidable in general. Hence perfect static analysis, perfect compiler optimization,
perfect verification, perfect antivirus are impossible *in principle*. Real tools are
approximations dancing around an undecidable core (sound-but-incomplete, or
complete-but-unsound, never both).

**Lineage:** one of the three 1930s self-reference earthquakes — Gödel incompleteness
(1931), Turing halting (1936), Church $\lambda$-calculus (1936). Turing's paper *defined*
the notion of "computation" (the Turing machine) as the vehicle for this very proof.

---

## 11. SAT — the keystone problem

**SAT (satisfiability):** given a Boolean formula over variables that are true/false,
combined with AND ($\wedge$), OR ($\vee$), NOT ($\neg$) — **is there an assignment making
the whole formula true?**

- **In NP:** the satisfying assignment is a perfect short certificate — plug it in,
  evaluate every clause in linear time. *Finding* it seems to need search over $2^n$
  assignments; *checking* a proposed one is instant.
- **NP-complete (Cook–Levin, 1971):** the *first* problem proven NP-complete. The proof
  encodes an *arbitrary* nondeterministic poly-time machine's computation as a SAT formula
  that is satisfiable iff the machine accepts (tape cells / states / time-steps become
  Boolean variables; legal-transition rules become clauses). So **logic is expressive
  enough to simulate computation itself**, making SAT a universal target every NP problem
  reduces to.

### CNF, and the $k$ in $k$-SAT

**Conjunctive Normal Form (CNF):** an AND of **clauses**, each clause an OR of
**literals** (a variable or its negation). To satisfy: **every clause must have at least
one true literal.**

The $k$ in **$k$-SAT** counts **literals per clause** — *not* which connectives are
allowed (all three are always available). It is a **clause width**, not a connective count.
(The "which connectives form a complete basis" question — AND+NOT, NAND alone, etc. — is a
*different* topic.)

$$
\textbf{2-SAT} \in \mathrm{P}
\qquad\qquad
\textbf{3-SAT} \text{ is NP-complete}
$$

A single increment in clause width — 2 to 3 — vaults from tractable to NP-complete.

### Why width 2 is easy and width 3 is hard

A **2-literal** clause $(a \vee b)$ is logically an **implication**:
$(a \vee b) \equiv (\neg a \Rightarrow b) \equiv (\neg b \Rightarrow a)$. A 2-SAT instance
is therefore a tangle of *deterministic forcings* — build the **implication graph**;
the formula is unsatisfiable iff some $x$ and $\neg x$ land in the same strongly-connected
component. SCCs are linear-time → 2-SAT is in P.

A **3-literal** clause $(a \vee b \vee c)$ does **not** collapse to a single implication:
knowing one literal is false only says "at least one of the *other two* is true" — a
**branch** with no forced move. That branching is exactly the combinatorial freedom that
lets 3-SAT encode arbitrary computation, and it kills the implication-graph trick.

### Can a 3-clause be converted to 2-clauses? — No, and the "no" *is* P ≠ NP

- **Logical equivalence (same variables):** impossible. $(a\vee b\vee c)$ is false on
  **exactly one** of 8 assignments (FFF). Any 2-literal clause is false on **two**
  assignments. A conjunction of 2-clauses has a false-set that is a *union* of size-2
  blocks — you can never carve out a false-set of size exactly 1. Truth tables can't match.
- **Equisatisfiability (with helper variables):** the auxiliary-variable "chain" trick
  *does* break wide clauses down — but it bottoms out **at width 3** (this is *how* you
  reduce general SAT to 3-SAT). It **cannot** descend to width 2, because a poly-time
  conversion 3-SAT $`\le_p`$ 2-SAT would put an NP-complete problem in P, i.e. **prove
  $\mathrm{P}=\mathrm{NP}$.**

The undescendable final step from width 3 to width 2 is P ≠ NP in a very small disguise.

**Practical twist** (ties to §6): despite worst-case NP-completeness, modern **SAT
solvers** routinely handle millions of variables (hardware verification, chip design,
planning) by exploiting structure in *real* instances. The poster child for "NP-complete
in theory, conquered in practice."

---

## 12. Inclusive-OR, precisely (a spot that quietly causes confusion)

$$
a \vee b \vee c \thickspace=\thickspace \text{"at least one of } a,b,c \text{ is true"} \thickspace=\thickspace \text{"not all three false."}
$$

- **Inclusive**, not exclusive: happy with one, two, or three true. Only objects to
  *zero* true.
- **True on 7 of 8 assignments**; false on exactly one (FFF) — a single forbidden corner
  of the cube. (This is *why* it resists 2-clause decomposition: one forbidden corner
  can't be built from clauses that each forbid two.)
- **Unordered, simultaneous, static** — *not* a left-to-right sequence. The logical
  $\vee$ has no "first/then." This differs from a programming short-circuit `||`, which
  *is* a sequence of control-flow steps. The two agree on the *answer* but differ in
  *structure* — and that's where the (mistaken) "surely I can chain it into a series"
  intuition comes from.

---

## 13. Verification vs. solving — `all`/`any`, and the two loops

Evaluating a CNF formula on a *given* assignment is exactly nested `all`/`any`:

```python
satisfied = all(any(literal_is_true for literal in clause)   # inner: cheap, linear
                for clause in formula)
```

This is the **verifier** — the cheap half of NP. But the assignment isn't given; *that's
what you search for.* Two very different loops:

```python
for assignment in all_2_to_the_n_assignments:        # OUTER loop — exponential — "solving"
    if all(any(lit for lit in clause)                # INNER check — cheap — "verification"
           for clause in formula):
        return SAT, assignment
return UNSAT
```

- **Inner loop** (`all`/`any` over literals, assignment fixed) = verification. In P.
- **Outer loop** (over all $2^n$ assignments) = solving. Exponential by brute force.

**P vs NP is exactly: can the outer loop be replaced by something cleverer than trying all
$2^n$?** For 2-SAT, yes (deterministic constraint propagation via the implication graph).
For 3-SAT, nobody knows how to avoid the search. The nondeterministic machine "skips" the
outer loop by *guessing* the assignment for free ($\exists$), then runs the cheap inner
verifier. P vs NP asks whether a *real* machine can skip it too.

---

## 14. Two great asymmetries ("the watcher is P; the inverse is not")

The verifier — the **watcher** — only ever watches **one** path. It is in P. The hardness
always lives in the quantifier the watcher can't cheaply discharge. *Two* distinct
"inverses" arise, and **both are open**:

| Easy (provably) | Hard (conjectured) | The open question |
|---|---|---|
| **check** a given witness | **find** a witness (discharge the $\exists$) | **P vs NP** |
| certify a **yes** (one passing witness) | certify a **no** (exhaust *all* witnesses — a $\forall$) | **NP vs co-NP** |

- $\mathbf{co\text{-}NP} = \{\thinspace L : \bar L \in \mathrm{NP} \thinspace\}$ — problems with short
  *disqualifying* certificates (e.g. proving a formula **unsatisfiable**). Verifying a
  *yes* for SAT is cheap (one assignment); verifying a *no* has no obvious short
  certificate. $\mathrm{P} \subseteq \mathrm{NP} \cap \mathrm{co\text{-}NP}$.
- The **polynomial hierarchy** generalizes this with alternating $\exists/\forall$:
  $`\mathrm{NP}=\Sigma_1^p`$, $`\mathrm{co\text{-}NP}=\Pi_1^p`$. $\mathrm{P}=\mathrm{NP}$ would
  collapse the entire hierarchy.

Same underlying flavor throughout: **verification is one-sided and cheap; the
complementary task — finding the $\exists$, or exhausting the $\forall$ — appears to lack
that cheap structure.**

---

## 15. Complexity as a statement about physics (the frontier wondering)

Computational complexity is secretly a statement about **how much physical resource**
(time, space, energy, precision) it takes to extract an answer from the world.

- **(Strong / Extended) Church–Turing thesis:** anything physically computable is
  computable with at most polynomial overhead. "Nature never solves a super-polynomial
  problem for free."
- **Quantum mechanics may already break the *classical* version:** an $n$-particle state
  lives in a $2^n$-dimensional Hilbert space, so classical simulation appears to need
  exponential resources. If $\mathrm{BQP} \supsetneq \mathrm{P}$ (believed), the universe
  itself is a counterexample to "all physics is classically poly-time simulable." (This is
  Feynman's 1982 argument that birthed quantum computing.) The natural repair: physics is
  efficiently simulable *by a quantum computer* — a quantum-flavored Church–Turing thesis.
- **But Nature is still not an $\exists$-oracle:** the believed picture is
  $\mathrm{BQP} \not\supseteq \mathrm{NP\text{-}complete}$ — quantum computers are
  *not* expected to crack NP-complete problems. (Grover's algorithm gives only a
  *quadratic* speedup for unstructured search — nowhere near the exponential needed.)
- **"Let Nature solve it" always hides an exponential somewhere.** Soap films (Steiner
  trees), spin-glass / annealing relaxation, protein folding — each proposal to "let a
  physical system settle into the answer" turns out to get stuck in local minima, or pay
  an exponential cost in relaxation time / precision / energy. The free $\exists$ keeps
  turning out to be unphysical.

**QFT footnote.** Apparent hardness from a *bad method* (divergent perturbation series —
QED's series has zero radius of convergence, Dyson 1952; factorially many diagrams) is
often defeasible: a cleverer non-perturbative method removes it. But *structural* hardness
is real: simulating quantum field theories is **BQP-complete** in regimes
(Jordan–Lee–Preskill 2012), and ground-state problems reach **QMA-complete** (the quantum
analog of NP-complete). **QCD** is the sharpest physical example of *robust* hardness:
perturbation theory is simply inapplicable at hadronic scales (coupling is $O(1)$); the
non-perturbative replacement (lattice QCD) hits the **fermion sign problem** at finite
density / real time, and a *general* solution to the sign problem is **NP-hard**
(Troyer–Wiese 2005). The genuinely hard QFT regimes sit at the **quantum** tier, which is
exactly why people now build quantum simulators for them.

**The epistemology to keep:** observed cost may be an artifact of *method* (cleverness can
dissolve it), but we can *sometimes* prove — conditionally, relative to class separations
— that hardness is *structural* and cleverness within a model cannot dissolve it. The art
is telling which regime you're in.

---

## 16. The diagram

![P / NP / NP-complete / NP-hard — Euler diagram, both panels](../../images/p-versus-np-complexity-diagram.svg)

*[Behnam Esfahbod](https://commons.wikimedia.org/wiki/File:P_np_np-complete_np-hard.svg), [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/)*

*Reading the two panels:*

- **Left panel ($\mathrm{P} \ne \mathrm{NP}$, the conjectured world):** P sits *strictly*
  inside NP; NP-complete is a band at the *top edge* of NP (in NP, disjoint from P);
  NP-hard includes that band **and extends upward past NP** (the overhang — undecidable /
  PSPACE-hard, etc.). NP-complete $=$ NP-hard $\cap$ NP, the part of the NP-hard region
  dipping down into NP.
- **Right panel ($\mathrm{P} = \mathrm{NP}$):** P, NP, and NP-complete **merge into one
  blob**. NP-hard still *contains* that merged blob **and still over-tops it** with the
  overhang — because $\mathrm{P}=\mathrm{NP}$ says nothing about problems outside NP. The
  halting problem doesn't become tractable.
- **Invariant across both panels:** **NP-hard over-tops NP no matter how P-vs-NP
  resolves.** The two panels are not two facts — they are the **two candidate answers** to
  the open question, with the persistent overhang the part that's the same in both.

---

## Compact glossary

- **Language** = problem (set of yes-instance strings). Decision (yes/no) only.
- **P** = solvable fast (deterministic poly-time).
- **NP** = checkable fast / has short certificates (= "guess-and-check"; the **N** is an
  **$\exists$**, not randomness, not quantum).
- **co-NP** = the *no*-side has short certificates.
- **Reduction $`\le_p`$** = an answer-preserving poly-time bridge $f$; "$`L_1`$ no harder than
  $`L_2`$."
- **NP-hard** = floor: everything in NP reduces to it (may be far harder than NP, even
  undecidable).
- **NP-complete** = NP-hard **∩** NP = the maximal elements of NP. ("Complete" = captures
  the class; it is the *narrower*, more demanding label.)
- **Verifier / watcher** = the cheap, in-P check of one witness. The hardness is always in
  the **finding** ($\exists$) or the **exhausting** ($\forall$).
