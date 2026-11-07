---
title: "Origins of Reinforcement Learning"
topic: reinforcement-learning
date: 2026-11-07
tags: [reinforcement-learning, history, psychology, optimal-control, dynamic-programming, terminology]
source: agent session 2026-11-07
papers:
  - "Minsky (1961) — Steps Toward Artificial Intelligence. Proc. IRE 49(1):8–30. DOI: 10.1109/JRPROC.1961.287775"
  - "Samuel (1959) — Some Studies in Machine Learning Using the Game of Checkers. IBM J. Res. Dev. 3(3)"
  - "Sutton (1984) — Temporal Credit Assignment in Reinforcement Learning. PhD thesis, Univ. of Massachusetts Amherst (COINS TR 84-2). http://incompleteideas.net/papers/Sutton-PhD-thesis.pdf"
  - "Sutton (1988) — Learning to Predict by the Methods of Temporal Differences. Machine Learning 3:9–44"
  - "Sutton (1992) — Introduction: The Challenge of Reinforcement Learning. Machine Learning 8. DOI: 10.1007/BF00992695"
  - "Kaelbling, Littman & Moore (1996) — Reinforcement Learning: A Survey. J. Artificial Intelligence Research 4:237–285 — [arXiv:cs/9605103](https://arxiv.org/abs/cs/9605103)"
  - "Williams (1992) — Simple statistical gradient-following algorithms for connectionist reinforcement learning. Machine Learning 8:229–256"
  - "Silver et al. (2016) — Mastering the game of Go with deep neural networks and tree search. Nature 529:484–489. DOI: 10.1038/nature16961"
  - "Tsetlin (1961) — On the behavior of finite automata in random media. Avtomatika i Telemekhanika 22:1345–1354"
  - "Mendel & McLaren (1970) — Reinforcement learning control and pattern recognition systems. In Mendel & Fu (eds.), Adaptive, Learning and Pattern Recognition Systems"
  - "Klopf (1982) — The Hedonistic Neuron: A Theory of Memory, Learning, and Intelligence. Hemisphere"
  - "Sutton & Barto (2018) — Reinforcement Learning: An Introduction, 2nd ed. MIT Press"
---

# Origins of Reinforcement Learning

## The Question

Did deep learning "borrow" the term *reinforcement learning*? The short answer is no. The *word* was borrowed twice, but the *mathematics* has been continuous since the 1980s:

1. **Psychology → computing (1950s–60s).** "Reinforcement" (and "learning") came from animal-learning psychology. This borrowing changed the concept: a description of how behavior changes became an optimization problem.
2. **Classical RL → deep RL (2013–).** This was *not* a borrowing. Deep RL kept the framework (MDPs, value functions, Bellman equations, TD learning, policy gradients) and replaced the function approximator: tables and linear features gave way to neural networks. AlphaGo ([Silver et al. 2016](https://www.nature.com/articles/nature16961)), for example, trains its policy network with Williams's (1992) REINFORCE.
3. **Deep RL → LLM fine-tuning (2020–).** The term is often used *loosely* here. Reward-filtered fine-tuning and DPO get called "RL" even though they have no environment dynamics, no exploration and no value function. The setting is close to a single-step contextual bandit.

## Two Roots

Modern RL is the merger of two lines of work that developed separately for decades.

### Root 1: Trial-and-error learning (psychology → AI)

| When | Who | Contribution |
|---|---|---|
| 1898 / 1911 | Thorndike | **Law of effect**: responses followed by satisfaction are strengthened ("stamped in"); those followed by discomfort are weakened. The idea without the word. |
| 1927 | Pavlov, *Conditioned Reflexes* (English translation by G. V. Anrep, Oxford University Press) | "Reinforcement" in the conditioning sense: pairing with the unconditioned stimulus, which a conditioned reflex needs in order to persist. This is the translation through which the English-speaking world learned Pavlov's term. |
| 1930s–50s | Skinner | **Operant conditioning**: behavior shaped by its consequences, with reinforcement schedules. Minsky (1961) takes the term "operant" directly from Skinner's *Science and Human Behavior* (1953). |
| 1951 | Bush & Mosteller | "A Mathematical Model for Simple Learning" (*Psych. Rev.* 58): **linear-operator learning models**, in which choice probabilities are nudged a fixed fraction toward 0 or 1 after reward or non-reward. This is mathematical psychology, and it is the update rule Minsky (1961) uses. |
| 1951 | Minsky | **SNARC** (Stochastic Neural Analog Reinforcement Calculator): a vacuum-tube machine that learned mazes by strengthening the connections that had led to success. Arguably the first RL machine. (Not mentioned in the 1961 paper.) |
| 1954 | Farley & Clark | "Simulation of self-organizing systems by digital computer": early trial-and-error learning in simulated neural nets. Minsky lists it as an important omission from his survey. |
| 1954 | Minsky | PhD thesis (Princeton, mathematics): *Theory of Neural-Analog Reinforcement Systems and Its Application to the Brain-Model Problem*. Minsky's own 1961 bibliography lists it informally as "Neural Nets and the Brain Model Problem." The title has "reinforcement systems," not "reinforcement learning." |
| 1961 | Minsky | *Steps Toward Artificial Intelligence*. Earliest prominent printed use of **"reinforcement learning"**, including in a section heading: [Sutton (1984)](http://incompleteideas.net/papers/Sutton-PhD-thesis.pdf) quotes §III-D as **"the basic credit-assignment problem for complex reinforcement learning systems"**. The phrase is used but never defined. What §III-A defines is "reinforcement" alone (a *reinforcement process* and *operator*). [Sutton (1992)](https://link.springer.com/content/pdf/10.1007/BF00992695.pdf) credits Minsky with coining the term. See the summary below. |
| 1968 | Michie & Chambers | **BOXES**: a tabular trial-and-error learner for pole balancing (cart-pole). |
| 1972 / 1982 | A. H. Klopf | "Brain function and adaptive systems — A heterostatic theory," Air Force Cambridge Research Laboratories report AFCRL-72-0164 (1972), and *The Hedonistic Neuron: A Theory of Memory, Learning, and Intelligence* (Hemisphere, 1982). **Heterostasis**: neurons, and organisms, as *maximizers* of a signal, against the homeostatic, equilibrium-seeking view of cybernetics (Ashby). Klopf argued that adaptive-systems research had lost the "hedonic" aspect: the drive to *get* something from the environment, which had been displaced by supervised pattern classification. He also proposed synaptic "eligibility," the origin of **eligibility traces**. As an Air Force researcher, he helped bring about the Amherst project whose first report (Barto & Sutton 1981, AFWAL-TR-81-1070) was an "initial assessment" of his goal-seeking components. |
| 1972 | Rescorla & Wagner | Error-driven model of classical conditioning, which TD learning later generalized across time. |

**Prediction versus control.** Psychology's two kinds of conditioning map onto RL's basic split:

| Psychology | What is learned | RL counterpart |
|---|---|---|
| **Classical (Pavlovian) conditioning** | Predicting reward from cues, regardless of action | **Prediction**: learning $V(s)$ (TD learning was first proposed as a model of this) |
| **Operant (Skinnerian) conditioning** | Choosing actions because of their consequences | **Control**: learning the policy $\pi(a \mid s)$ |

### A third strand: learning control and learning automata (1960s)

In parallel with the AI and psychology line, control engineers and Soviet cyberneticists built **learning controllers**. These were finite-state machines that keep a table of action probabilities for each "control situation" and adjust them by reward and penalty. By the early 1960s this was an established, international field, with no AI or psychology sources in its reference lists.

Two representative works: Tsetlin (1961), on finite automata in random media, began the Soviet **learning-automata** tradition. Mendel & McLaren (1970), "Reinforcement learning control and pattern recognition systems," is the source Sutton (1984) cites for the term's use in control engineering.


### Root 2: Optimal control and dynamic programming (mathematics)

| When | Who | Contribution |
|---|---|---|
| 1950s | Bellman | **Dynamic programming**, the **Bellman equation**, and discrete-time stochastic optimal control, i.e. the **Markov decision process** (MDP) *(Bellman 1957)*. |
| 1960 | Howard | **Policy iteration** for MDPs. |
| 1950s–60s | Pontryagin, Kalman, and others | Continuous-time optimal control: the maximum principle and the Hamilton–Jacobi–Bellman equation. |

This root contributes the *mathematics*: value functions, the Bellman fixed point, and policies as feedback laws. It does not learn from experience; it assumes the dynamics are known.

### The bridge: Samuel's checkers (1959)

Samuel's checkers program learned an evaluation function (a value function) by adjusting earlier position estimates toward later ones. That is essentially **temporal-difference learning**, about 30 years before it was formalized. Sutton & Barto call it the earliest machine-learning work they see as directly relevant to RL. The paper's title also helped popularize the phrase **"machine learning"**.

## Minsky (1961): The Founding Survey

The paper's §III "Learning Systems" contains in embryonic form nearly every central RL idea:

| Minsky (1961) | Modern RL |
|---|---|
| The "Trainer," who detects success but "need not know how to solve problems" | Reward function: evaluative rather than instructive feedback |
| Bush–Mosteller operator $`p_{n+1} = q\thinspace p_n + (1-q)`$, an exponentially weighted average | Learning-rate update; the same form as TD(0) |
| "how often one should play a strategy different from the estimated optimum" | Exploration versus exploitation |
| Mesa phenomenon | Sparse rewards, plateaus |
| Operant machine plus a "Pavlovian" device $U$ predicting reinforcement | Actor-critic (policy plus value function) |
| Warning that chained secondary reinforcers may let "values drift" | Instability of bootstrapping; reward hacking |
| Predictor $P$; a premium for novel predictions | World models; curiosity |
| Samuel's "Delta" | A TD error with a search-based target |
| §III-D, "The Basic Credit-Assignment Problem for Complex Reinforcement Learning Systems" (original heading, as quoted by Sutton 1984) | Named outright |


## The First Explicit Definition: Sutton (1984)

Minsky named the problem but never defined the field. The first explicit definition appears in Sutton's thesis (Ch. I, p. 2). It is built in two steps:

> "For the purposes of this dissertation, a *learning system* is defined as a system that uses information gained during one interaction with its *environment* to improve its performance in subsequent interactions (after Smith, 1980). A *reinforcement-learning system* is a learning system that learns under the influence of reinforcement, where *reinforcement* is feedback from the environment that assigns credit to the learning system's actions but does not assign credit to their internal structure or indicate what actions would have been better."

He immediately contrasts it with supervised learning:

> "Supervised learning pattern classification … is not reinforcement learning because the environment, in this case called a 'teacher,' indicates what actions should have been taken and thus implicitly assigns credit to particular internal decisions."

The same chapter splits Minsky's credit-assignment problem in two: **temporal** (which *actions* deserve credit for an outcome) and **structural** (which *internal decisions* deserve credit for an action). RL addresses the temporal half; backpropagation later addressed the structural half.


## Dormancy and Revival (1970s–1980s)

Sutton (1992) describes trial-and-error learning as **"largely forgotten"** for a period. Much of what was called "reinforcement learning" in the 1960s–70s drifted into supervised pattern classification (learning from labeled examples rather than evaluative feedback).

The revival culminated in [Sutton & Barto (1998)](http://incompleteideas.net/book/the-book-2nd.html), *Reinforcement Learning: An Introduction*. It is still the standard first citation for the term, and its definition, learning to maximize a numerical reward signal, is the one the field uses today.

