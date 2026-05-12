# Observation Noise

This document explains the observation noise experiments in this project — what noise means, how it is applied, and why the results are non-monotonic.

---

## What Observation Noise Means

In this project, **observation noise** refers to Gaussian noise added to the agent's sensor readings during environment interaction. The agent receives a noisy version of its true sensor distances, simulating imperfect perception — as would occur in a real-world autonomous vehicle with noisy LIDAR or distance sensors.

The noise is applied to the **three raycasting sensor channels** (indices 4, 5, 6 of the 7-dimensional state vector). The other four state channels (position x, position y, speed, heading) remain clean.

---

## Which State Channels Receive Noise

| Index | Feature | Noise Applied? |
|-------|---------|----------------|
| 0 | Position X | No |
| 1 | Position Y | No |
| 2 | Speed | No |
| 3 | Heading | No |
| 4 | Sensor Left (−45°) | **Yes** |
| 5 | Sensor Front (0°) | **Yes** |
| 6 | Sensor Right (+45°) | **Yes** |

The noise is added in `car.py` during the `cast_sensors()` method:

```python
if has_noise:
    noise = np.random.normal(0, noise_std)
    normalized_dist += noise
    # Clip to [0.0, 1.0]
```

After noise is added, the value is clipped to [0.0, 1.0] to maintain valid sensor readings.

---

## Noise Levels

Three noise levels are tested, labeled N1–N3:

| Label | Standard Deviation | Interpretation |
|-------|--------------------|----------------|
| **N1** | 0.00 | Perfect sensors — no noise, clean observations |
| **N2** | 0.02 | Mild noise — realistic perception jitter |
| **N3** | 0.05 | Heavy noise — significant sensory uncertainty |

These noise levels are defined in `config.py`:

```python
EXPERIMENT_SENSOR_NOISE_LEVELS = (0.0, 0.02, 0.05)
```

Since sensor distances are normalized to [0, 1], a noise std of 0.02 means typical perturbations of ±2% of the full sensor range, and 0.05 means ±5%.

---

## Why Noise Is Added Only to Sensors

Position, speed, and heading represent the car's internal state — values that a real vehicle would know precisely from odometry and internal sensors. However, distance to track boundaries (measured by external sensors like LIDAR or cameras) is inherently noisy in real-world applications due to:

- Sensor measurement error
- Environmental interference (rain, dust, reflections)
- Quantization and processing delays

By adding noise only to the sensor channels, the experiments simulate a realistic scenario: the agent has accurate self-knowledge but imperfect perception of its surroundings.

---

## How Moderate Noise Can Help

This is the most counter-intuitive finding: **moderate noise (N2) can sometimes improve performance** compared to no noise (N1).

### Conditional Regularization

Moderate noise acts as a form of **implicit regularization** during training:

- The agent is forced to learn policies that do not rely on exact sensor values
- The policy becomes more robust to small variations in input
- This is similar to how dropout or data augmentation in supervised learning can improve generalization

However, this regularization effect is **conditional** — it helps only when:

1. The **reward shaping is sufficiently informative** (e.g., R3 with speed scaling and stability bonuses)
2. The reward signal can guide learning despite the noise

With minimal reward shaping (R1), the agent receives so little guidance that noise only makes learning harder, not better.

### Why It Is Not Always Beneficial

Noise is a double-edged sword:

- Too little noise (N1) may allow the agent to overfit to exact sensor values, creating brittle policies
- Moderate noise (N2) can regularize learning and produce more robust policies
- Too much noise (N3) overwhelms the signal, making it difficult for the agent to distinguish between safe and dangerous states

---

## Why High Noise Suppresses Learning

At N3 (std = 0.05), the sensor readings are perturbed by up to ±5% of their range at every time step. This creates several problems:

1. **Credit assignment difficulty:** The agent cannot reliably determine which actions led to good or bad outcomes because the perceived state is significantly different from the true state
2. **Q-value noise amplification:** The critic's Q-estimates become noisy because they are trained on state-action pairs where the state is corrupted, leading to inaccurate value estimates
3. **Exploration-exploitation confusion:** The agent cannot reliably tell when it is near a track boundary, leading to more frequent accidental crashes and less effective exploration

---

## Non-Monotonic Results

A key finding in this project is that **performance does not simply decrease as noise increases**. The relationship between noise level and agent performance is non-monotonic:

```
Performance
    ▲
    │     ●  (N2 — moderate noise)
    │   ●    (N1 — no noise)
    │
    │          ● (N3 — high noise)
    │
    └─────────────────► Noise Level
```

This non-monotonic pattern holds primarily for **well-shaped reward modes** (R3, and to some extent R2 and R4):

- **R3 + N2** has shown strong performance in recent runs — better than R3 + N1 in some metrics
- **R1 + N2** does not show the same benefit because R1's minimal reward signal cannot guide learning through the noise

The takeaway: **moderate noise helps only when reward shaping is informative enough to compensate for the added uncertainty.** Noise and reward design interact — they cannot be evaluated independently.

---

## Summary

| Level | Effect | When It Helps | When It Hurts |
|-------|--------|---------------|---------------|
| N1 (0.00) | Perfect perception | Always a valid baseline | May allow overfitting to exact values |
| N2 (0.02) | Mild regularization | With informative rewards (R2–R4) | With minimal rewards (R1) |
| N3 (0.05) | Strong noise | Rarely beneficial | Almost always degrades performance |

---

## Further Reading

- [reward_system.md](reward_system.md) — How reward modes interact with noise
- [experiment_design.md](experiment_design.md) — The full factorial grid of noise × reward
- [results_interpretation.md](results_interpretation.md) — How to read the results
- **Implementation:** [`car.py`](../car.py) — `cast_sensors()` method
