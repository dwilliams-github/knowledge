---
title: Reinforcement Learning Glossary
topic: reinforcement-learning
tags: [reinforcement-learning, glossary]
---

# Reinforcement learning glossary

Core terms used across the reinforcement-learning section. Add new terms as articles introduce them.

**Agent, environment.** The learner and everything it interacts with. At each step the agent observes a state $s$, takes an action $a$, and receives a reward $r$.

**Policy.** The agent's behavior: a conditional distribution $\pi(a \mid s)$ over actions given the state. It is the thing being learned. (The goal is specified by the reward, not the policy.)

**Reward.** The scalar feedback $r$ after each step; the goal specification.

**Return.** The discounted sum of future rewards along one trajectory, $\sum_t \gamma^t r_t$, with discount factor $0 \le \gamma \le 1$.

**Value function.** Expected return under a policy: from a state, $V^\pi(s)$, or after taking an action, $Q^\pi(s,a)$.

**Advantage.** $A^\pi(s,a) = Q^\pi(s,a) - V^\pi(s)$: how much better an action is than the policy's average. Sutton's (1984) "ideal reinforcement signal."

**Reinforcement.** Feedback that assigns credit to the agent's actions but neither assigns credit to their internal structure nor says what would have been better (Sutton 1984).

**Evaluative vs. instructive feedback.** Evaluative feedback scores what was done (RL). Instructive feedback says what should have been done (supervised learning, with its "teacher").

**Credit assignment.** Deciding which decisions deserve credit for an outcome (Minsky 1961). *Temporal*: which actions, in time. *Structural*: which internal components or parameters.

**Prediction vs. control.** Learning to predict reward (a value function) versus learning to act (a policy). These correspond to classical and operant conditioning.

**Secondary reinforcement.** A signal that becomes reinforcing because it predicts reinforcement. In RL this is a learned value function used as a learning signal.

**Markov decision process (MDP).** States, actions, transition probabilities and rewards, where the next state depends only on the current state and action. The standard formal setting for RL.

**Bellman equation.** The recursive consistency condition for values, e.g. $V^\pi(s) = \mathbb{E}\left[r + \gamma V^\pi(s')\right]$. The basis of dynamic programming.

**Temporal-difference (TD) learning.** Updating a value estimate toward a later estimate. The TD error is $\delta = r + \gamma V(s') - V(s)$.

**Bootstrapping.** In RL, updating an estimate using another estimate, as in TD learning. (Not the statistical bootstrap, and not the older sense of a system improving its own methods.)

**Eligibility trace.** A decaying record of recently visited states or actions, so a current TD error can update them all. The $\lambda$ in TD($\lambda$); from Klopf's synaptic "eligibility."

**Actor-critic.** An architecture with a policy (actor) and a learned value function (critic) whose TD error trains the actor.

**Policy gradient.** Adjusting the policy parameters along $\mathbb{E}[\nabla \log \pi \cdot A]$. REINFORCE (Williams 1992) is the basic estimator.

**Exploration vs. exploitation.** Trading off trying uncertain actions to gain information against taking the best-known action.

**Bandit.** The one-step special case of RL: choose an action, receive a reward, with no state transitions. *Contextual* bandits condition on an observed context.
