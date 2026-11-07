# Index

<!-- ML/paper entries are organized by publication year; algorithm entries go in ## Algorithms; RL concept/history entries go in ## Reinforcement Learning; foundational ML concepts go in ## Foundations. -->

## Algorithms

<!-- Entries sorted alphabetically by concept name. -->

- [Graph Products](topics/algorithms/graph-products.md) — Cartesian, tensor, strong, lexicographic, and modular products; edge predicates, edge counts, adjacency matrix decompositions, and the $P_3$ worked example
- [Graph Theory Glossary](topics/algorithms/graph-theory-glossary.md) — definitions of core graph-theoretic terms: graph, adjacency, degree, path, bipartite, adjacency matrix, and product-specific vocabulary
- [Maximum Common Induced Subgraph](topics/algorithms/maximum-common-induced-subgraph.md) — reduction to maximum clique via the modular product; Bron–Kerbosch algorithm; complexity and practical seed-ordering heuristics
- [P versus NP](topics/algorithms/p-versus-np.md) — complexity classes, reductions, NP-completeness, the halting problem, SAT, and the physics of hardness; includes glossary

## Reinforcement Learning

- [Origins of Reinforcement Learning](topics/reinforcement-learning/rl-origins.md) — two roots (psychology and optimal control) plus learning automata; Minsky (1961) names the problem, Sutton (1984) defines the field
- [Reinforcement Learning Glossary](topics/reinforcement-learning/rl-glossary.md) — core RL terms; grows with each RL article

## Foundations

- [KL Divergence](topics/generative-models/kl-divergence.md) — definition, mode-covering vs mode-seeking asymmetry, equivalence to MLE, measure-theoretic foundations, and role as the ELBO gap in VAEs
- [Sequential Monte Carlo](topics/generative-models/sequential-monte-carlo.md) — SMC fundamentals, AESMC, and FK steering of diffusion models; particle resampling as inference-time reward alignment

## 2022

- [Progressive Distillation](topics/generative-models/progressive-distillation.md) — Salimans & Ho (Feb 2022); iterative teacher-student halving to compress diffusion sampling from thousands of steps to 4; introduces v-prediction; precursor to flow map matching
- [Stochastic Interpolants](topics/generative-models/stochastic-interpolants.md) — Albergo & Vanden-Eijnden (Sep 2022 / Mar 2023); unifying framework for flows and diffusions via $I_t = \alpha_t I_0 + \beta_t I_1$

## 2024

- [Flow Map Matching](topics/generative-models/flow-map-matching.md) — Boffi, Albergo & Vanden-Eijnden (Jun 2024); unifying framework for fast diffusion inference; shows progressive distillation, consistency models, and CTMs are special cases of learning the two-time flow map $`F_{s,t}`$

## 2025

- [Feynman-Kac Steering](topics/generative-models/feynman-kac-steering.md) — Singhal et al. (Jan 2025); inference-time reward alignment for diffusion models via FK-IPS particle resampling, no fine-tuning required

## 2026

- [TimeGS](topics/generative-models/time-gs.md) — Wang et al. (Mar 2026); time series forecasting via 2D period reshaping and a fixed-dictionary Gaussian decoder; critical reading including LatentTSF detour
