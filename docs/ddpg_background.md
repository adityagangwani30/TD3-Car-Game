# DDPG — Deep Deterministic Policy Gradient (Background)

This document explains **DDPG** — the foundation that TD3 improves upon.

---

## What Is DDPG?

DDPG is an **actor-critic RL algorithm** for continuous action spaces, introduced by Lillicrap et al. (2015). It combines DQN-style techniques (experience replay, target networks) with the deterministic policy gradient theorem.

---

## Actor-Critic Structure

### Actor (Policy Network)

- Maps states directly to actions (deterministic — same state always yields same action before noise)
- In this project (`ddpg_agent.py`): 7-dim state → 2 hidden layers (64 units, ReLU) → 2-dim action (tanh output)

### Critic (Q-Network)

- Estimates Q(s, a) — expected cumulative reward for taking action a in state s
- Architecture: state+action concatenated (9 dims) → 2 hidden layers (64 units, ReLU) → scalar Q-value
- DDPG uses a **single** critic (unlike TD3's twin critics)

### How They Work Together

1. Actor proposes an action
2. Critic evaluates that action's Q-value
3. Actor is updated to maximize the critic's Q-value
4. Critic is updated using the Bellman equation

---

## Deterministic Policy Gradient

Standard policy gradients sample from action distributions (high variance). The DPG theorem shows deterministic policies have a simpler gradient:

```
∇θ J ≈ E[ ∇a Q(s,a) · ∇θ μ(s) ]
```

In code (`ddpg_agent.py`):
```python
actor_loss = -self.critic(state, self.actor(state)).mean()
```

---

## Key Mechanisms

- **Experience Replay:** Same buffer as TD3 — 200,000 capacity, batch size 256, training starts after 5,000 transitions
- **Target Networks:** Soft-updated via Polyak averaging (τ = 0.005)
- **Exploration Noise:** Gaussian noise added to actions (initial 0.1, decay 0.9999/episode)

---

## Problems with DDPG

### 1. Overestimation Bias

A single critic tends to overestimate Q-values. The actor then exploits these overestimates, creating a feedback loop that can cause Q-values to diverge. In the car environment, this manifests as overconfident aggressive driving.

### 2. Instability

DDPG updates the policy at **every training step** (same frequency as critic). Inaccurate Q-estimates propagate directly into policy changes, causing noisy learning curves and sudden performance collapses.

### 3. Sensitivity to Hyperparameters

Small changes in learning rates, network size, exploration noise, or buffer size can cause DDPG to fail entirely — making reliable deployment difficult across different reward configurations.

---

## How TD3 Fixes These

| DDPG Problem | TD3 Solution | Mechanism |
|---|---|---|
| Overestimation bias | Twin critics | Minimum of two independent Q-estimates |
| Policy instability | Delayed policy updates | Actor updated every 2 critic updates |
| Critic exploitation | Target policy smoothing | Clipped noise added to target actions |

---

## DDPG vs TD3 in This Project

Both use **identical hyperparameters** for fair comparison:

| Aspect | DDPG | TD3 |
|--------|------|-----|
| Critics | 1 | 2 (twin) |
| Policy update freq | Every step | Every 2nd step |
| Target action noise | None | Clipped Gaussian |
| Architecture | 2×64 hidden | 2×64 hidden |
| Soft update τ | 0.005 | 0.005 |
| Learning rates | 3×10⁻⁴ | 3×10⁻⁴ |
| Gradient clipping | Max norm 1.0 | Max norm 1.0 |

---

## Further Reading

- **DDPG paper:** [Continuous Control with Deep RL (Lillicrap et al., 2015)](https://arxiv.org/abs/1509.02971)
- **TD3 paper:** [Addressing Function Approximation Error (Fujimoto et al., 2018)](https://arxiv.org/abs/1802.09477)
- **TD3 in this project:** [td3_explanation.md](td3_explanation.md)
- **Implementation:** [`ddpg_agent.py`](../ddpg_agent.py)
