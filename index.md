# Index

All research entries, organized by paper publication date.

<!-- Within each year, entries are sorted chronologically. Foundational entries with no single publication date go at the top. -->

## Foundations

- [KL Divergence](topics/generative-models/kl-divergence.md) — definition, mode-covering vs mode-seeking asymmetry, equivalence to MLE, measure-theoretic foundations, and role as the ELBO gap in VAEs
- [Sequential Monte Carlo](topics/generative-models/sequential-monte-carlo.md) — SMC fundamentals, AESMC, and FK steering of diffusion models; particle resampling as inference-time reward alignment

## 2022

- [Progressive Distillation](topics/generative-models/progressive-distillation.md) — Salimans & Ho (Feb 2022); iterative teacher-student halving to compress diffusion sampling from thousands of steps to 4; introduces v-prediction; precursor to flow map matching
- [Stochastic Interpolants](topics/generative-models/stochastic-interpolants.md) — Albergo & Vanden-Eijnden (Sep 2022 / Mar 2023); unifying framework for flows and diffusions via $I_t = \alpha_t I_0 + \beta_t I_1$

## 2025

- [Feynman-Kac Steering](topics/generative-models/feynman-kac-steering.md) — Singhal et al. (Jan 2025); inference-time reward alignment for diffusion models via FK-IPS particle resampling, no fine-tuning required
