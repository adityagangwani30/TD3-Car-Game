# Experiment Design

This document explains the experimental setup used in this project.

---

## Factorial Grid

The experiments use a **full factorial design** combining:

- **4 reward modes:** R1 (basic), R2 (shaped), R3 (modified), R4 (tuned)
- **3 noise levels:** N1 (0.00), N2 (0.02), N3 (0.05)
- **2 algorithms:** TD3, DDPG

This produces **12 configurations per algorithm** (4 × 3 = 12), and **24 total configurations** across both algorithms.

Each configuration is trained with **3 independent random seeds** (0, 42, 123) for statistical validity, resulting in **72 total training runs**.

### The 12 Experiment Grid (per algorithm)

| | N1 (0.00) | N2 (0.02) | N3 (0.05) |
|---|---|---|---|
| **R1** (basic) | R1_N1 | R1_N2 | R1_N3 |
| **R2** (shaped) | R2_N1 | R2_N2 | R2_N3 |
| **R3** (modified) | R3_N1 | R3_N2 | R3_N3 |
| **R4** (tuned) | R4_N1 | R4_N2 | R4_N3 |

---

## Training Parameters

All experiments use the same hyperparameters (defined in `config.py`):

| Parameter | Value |
|-----------|-------|
| Episodes per run | 2,000 |
| Max steps per episode | 300 |
| Replay buffer capacity | 200,000 |
| Batch size | 256 |
| Training starts after | 5,000 steps |
| Network architecture | 2 hidden layers, 64 units each |
| Actor learning rate | 3 × 10⁻⁴ |
| Critic learning rate | 3 × 10⁻⁴ |
| Discount factor (γ) | 0.99 |
| Soft update rate (τ) | 0.005 |
| Exploration noise | 0.1, decaying at 0.9999/episode |
| Random seeds | 0, 42, 123 |

Keeping all settings fixed except the experimental variables (reward mode, noise level, algorithm) is essential for **controlled comparison**. Any observed performance differences can then be attributed to those variables rather than to confounding hyperparameter differences.

---

## Metrics

Each experiment is evaluated using multiple metrics:

### 1. Rolling Reward (window = 100)

The rolling average of total episode reward over the last 100 episodes. Captures learning progress and asymptotic performance.

### 2. Crash Rate

Percentage of episodes where the agent terminates by going off-track. Lower is better — this is a safety metric.

### 3. Lap Completion Rate

Percentage of episodes where the agent completes at least one full lap. Higher is better — this confirms task success.

### 4. Stability (Reward Variance)

Standard deviation of rewards within a run, averaged across seeds. Lower variance indicates more consistent behavior.

### 5. Convergence Episode

The episode at which the agent's rolling reward stabilizes (estimated computationally). Earlier convergence suggests faster learning.

---

## Why This Is a Factorial Study

A factorial study tests **all combinations** of experimental factors, not just individual factors in isolation. This design reveals:

- **Main effects:** Does changing the reward mode affect performance regardless of noise? Does noise affect performance regardless of reward?
- **Interaction effects:** Does the effect of noise depend on which reward mode is used? (Answer: yes — moderate noise helps R3 but not R1)
- **Non-monotonic relationships:** Performance does not simply improve with more shaping or decrease with more noise

A single-factor study (e.g., testing only reward modes at one noise level) would miss these interactions entirely.

---

## Why Fixed Settings Matter

Keeping hyperparameters identical across experiments ensures that:

- Performance differences are due to reward/noise/algorithm, not tuning
- Results are comparable across configurations
- The study answers "How do these factors affect performance?" rather than "What is the best possible tuning for each configuration?"

The trade-off: a configuration might perform better with custom-tuned hyperparameters. But custom tuning per configuration would confound the analysis — you could not tell whether a performance difference was due to the factor of interest or the tuning.

---

## Stochasticity and Interpretation

### Sources of Randomness

Reinforcement learning results are inherently stochastic due to:

- Neural network weight initialization
- Exploration noise during action selection
- Random mini-batch sampling from the replay buffer
- Environment dynamics (sensor noise, starting conditions)

### Current Study Design

The current study trains **3 seeds per configuration** to reduce the impact of random variation. Results are aggregated as mean ± standard deviation across seeds.

### Interpretation Guidelines

- Due to stochasticity, exact values may vary across runs
- Focus on **trends and relative rankings** rather than exact numbers
- Large standard deviations indicate that the configuration is sensitive to initialization
- The current study should be interpreted **descriptively** — as characterizing behavior patterns across the factorial grid

---

## Further Reading

- [reward_system.md](reward_system.md) — Detailed explanation of R1–R4
- [observation_noise.md](observation_noise.md) — Detailed explanation of N1–N3
- [results_interpretation.md](results_interpretation.md) — How to read the generated results
- [reproducibility.md](reproducibility.md) — Reproducibility details and limitations
- **Implementation:** [`run_experiments.py`](../run_experiments.py), [`config.py`](../config.py)
