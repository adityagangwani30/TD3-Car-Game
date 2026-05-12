# Reproducibility

This document discusses reproducibility in this reinforcement learning project — why results vary, what causes randomness, and how to improve confidence in findings.

---

## Why Reinforcement Learning Results Vary

RL results are inherently stochastic. Even with the same code, hyperparameters, and random seed, results may differ across runs due to hardware and software non-determinism. Between different seeds, variation is expected and can be substantial.

This is a known property of deep RL, not a bug. It means that:
- Single-run conclusions are unreliable
- Results must be interpreted as trends across multiple seeds
- Error bars and standard deviations are essential

---

## Sources of Randomness

### 1. Neural Network Initialization

Both the actor and critic networks are initialized with random weights (PyTorch default initialization). Different weight initializations lead to different optimization trajectories and potentially different final policies.

### 2. Exploration Noise

During training, Gaussian noise is added to the agent's actions for exploration:
```python
noise = np.random.normal(0, noise_scale, size=ACTION_DIM)
```

Different noise samples lead to different experiences being collected, which changes the contents of the replay buffer and therefore the training signal.

### 3. Replay Buffer Sampling

Mini-batches are sampled uniformly at random from the replay buffer:
```python
indices = np.random.randint(0, self.size, size=batch_size)
```

Different mini-batches expose the networks to different combinations of past experiences, affecting gradient updates.

### 4. Environment Interaction

Sensor noise (when N2 or N3 is used) adds random perturbations to observations at every time step. This changes the agent's perception of the environment and therefore its behavior and collected experiences.

### 5. Hardware Non-Determinism

Even with deterministic mode enabled (`torch.backends.cudnn.deterministic = True`), some operations may produce slightly different results across:
- CPU vs GPU execution
- Different GPU architectures
- Different CUDA versions
- Floating-point operation ordering

---

## How Seeds Are Used

This project sets random seeds across multiple libraries in `utils.py`:

```python
def set_global_seed(seed: int):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
```

The experiment runner uses three seeds: **0, 42, and 123**.

Seeds control:
- Neural network weight initialization
- Exploration noise generation
- Replay buffer sampling order
- Sensor noise generation

---

## Current Limitation

The current study trains **3 seeds per configuration**. While this is better than a single run, 3 seeds provide only a rough estimate of the mean and a noisy estimate of variance.

With 3 seeds:
- The sample mean may not closely approximate the true mean
- The standard deviation estimate has high uncertainty
- Outlier seeds can substantially shift the reported statistics

---

## Recommended Future Improvements

### Multi-Seed Experiments

For stronger statistical validity, future work should use:

- **5–10 seeds per configuration** — provides more reliable mean estimates
- **Mean and standard deviation reporting** — across all seeds for every metric
- **Statistical tests** — e.g., paired t-tests or Wilcoxon signed-rank tests to determine whether differences between configurations are statistically significant

### Additional Practices

- **Report confidence intervals** instead of or in addition to standard deviations
- **Use box plots or violin plots** to show the full distribution across seeds
- **Track computational cost** to enable cost-performance trade-off analysis
- **Log hardware specifications** to enable cross-platform comparison

---

## Reproducibility Statement

The following statement is suitable for inclusion in a README or research paper:

> *Due to the stochastic nature of reinforcement learning, exact values may vary across runs, but overall trends should be interpreted across reward-noise configurations.*

---

## What Is Reproducible

Despite the stochasticity, the following aspects of this project are fully reproducible:

| Aspect | How |
|--------|-----|
| Experiment grid | Defined in `config.py` as `EXPERIMENTS` |
| Hyperparameters | All in `config.py`, identical across experiments |
| Code | Version-controlled in Git |
| Logs | JSONL format with all episode data |
| Seeds | Fixed at 0, 42, 123 |
| Environment | Deterministic physics (given same inputs and sensor noise seed) |

---

## Further Reading

- [experiment_design.md](experiment_design.md) — How the experiments are structured
- [results_interpretation.md](results_interpretation.md) — How to interpret results given stochasticity
- **Implementation:** [`utils.py`](../utils.py) — `set_global_seed()`, [`config.py`](../config.py) — all hyperparameters
