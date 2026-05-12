# Training and Evaluation

This document explains how to train, evaluate, and run experiments using the actual commands available in this project.

---

## Types of Runs

| Run Type | Script | Purpose |
|----------|--------|---------|
| Single training run | `main.py` | Train one agent with default settings |
| Batch experiment run | `run_experiments.py` | Run all 12 configurations for one algorithm |
| Evaluation/demo run | `main.py` | Evaluate a trained agent or run a demo |
| Plot generation | `plot_metrics.py` | Generate plots from existing logs |
| Model comparison | `eval_models.py` | Compare multiple saved checkpoints |

---

## Single Training Run

Train a single agent with default settings:

```bash
# Train TD3 (2,000 episodes, with GUI rendering)
python main.py --algo td3 --mode train

# Train DDPG in headless mode (faster, no display)
python main.py --algo ddpg --mode train --headless

# Custom episode and step counts
python main.py --algo td3 --mode train --max-episodes 1000 --max-steps 300 --headless

# Resume training from latest checkpoint
python main.py --algo td3 --mode train --resume

# Resume from a specific checkpoint
python main.py --algo td3 --mode train --checkpoint models/td3/td3_ep500.pth
```

A single training run uses the default reward mode (`shaped` / R2) and the default sensor noise from `config.py`. It trains from scratch unless `--resume` or `--checkpoint` is specified.

---

## Batch Experiment Run

Run all 12 configurations (4 reward × 3 noise) for one algorithm:

```bash
# Run all TD3 experiments
python run_experiments.py --algo td3 --headless

# Run all DDPG experiments
python run_experiments.py --algo ddpg --headless

# Run a specific seed only
python run_experiments.py --algo td3 --seed 42 --headless

# Resume interrupted experiments (skips completed runs)
python run_experiments.py --algo td3 --resume --headless

# Run only the first N experiments (useful for testing)
python run_experiments.py --algo td3 --max-experiments 3 --headless

# Start from a specific experiment index (for batching)
python run_experiments.py --algo td3 --start-index 6 --headless

# Custom episode count
python run_experiments.py --algo td3 --max-episodes 500 --headless
```

The batch runner:
- Iterates through all 12 experiments defined in `config.EXPERIMENTS`
- For each experiment, trains with seeds [0, 42, 123] (unless `--seed` overrides)
- Creates isolated output directories per (algo, experiment, seed)
- With `--resume`, skips experiments where logs already show completion

---

## Evaluation and Demo

Evaluate a trained model without exploration noise:

```bash
# Evaluate TD3 with best checkpoint (10 episodes)
python main.py --algo td3 --mode eval --render

# Evaluate with a specific checkpoint
python main.py --algo td3 --mode eval --checkpoint models/td3/td3_best.pth --render

# Custom number of evaluation episodes
python main.py --algo td3 --mode eval --eval-episodes 20 --render

# Run interactive demo (2 episodes, always rendered)
python main.py --algo td3 --mode demo
```

Evaluation mode:
- Loads the best available checkpoint (or a specified one)
- Runs deterministic actions (no exploration noise)
- Reports average reward, crash rate, and lap completion

### Multi-Model Comparison

```bash
# Evaluate a specific model file
python eval_models.py --model td3_best.pth --episodes 10

# Use pattern matching to evaluate multiple models
python eval_models.py --model "td3_ep*.pth" --episodes 5

# Headless evaluation
python eval_models.py --headless --episodes 10
```

---

## Plot Generation

Generate plots from existing training logs:

```bash
# Generate individual plots for one algorithm
python plot_metrics.py --algo td3

# Generate TD3 vs DDPG comparison plots + grouped noise-level plots
python plot_metrics.py --compare-algos

# Custom smoothing window
python plot_metrics.py --algo td3 --window 50

# Specific experiments only
python plot_metrics.py --algo td3 --experiments R1_N1 R2_N1 R3_N1

# Custom output directories
python plot_metrics.py --compare-algos --comparison-output results/custom_plots
```

Plot generation reads from `logs/` and writes to `results/`:

| Output | Directory |
|--------|-----------|
| Individual per-experiment plots | `results/plots/{algo}/individual/{experiment}/` |
| TD3 vs DDPG comparisons | `results/plots/comparison/` |
| Grouped by noise level | `results/grouped/` |

---

## Where Outputs Are Saved

### Logs

```
logs/{algo}/{experiment_tag}/seed_{seed}/training_log.jsonl
```

Example: `logs/td3/R1_N1/seed_0/training_log.jsonl`

Each JSONL file contains one JSON object per episode with reward, crash, lap, speed, and noise data.

### Models

```
models/{algo}/{experiment_tag}/seed_{seed}/{algo}_best.pth
```

Checkpoint types:
- `{algo}_best.pth` — Best single-episode reward
- `{algo}_best_avg100.pth` — Best 100-episode rolling average
- `{algo}_ep{N}.pth` — Periodic checkpoints every 100 episodes

### Results/Plots

```
results/plots/{algo}/individual/{experiment}/reward_vs_episodes.png
results/plots/comparison/{experiment}_reward_comparison.png
results/grouped/{algo}_{noise}_reward.png
```

---

## Training Behavior

### Fresh Start vs Resume

- **Fresh start** (default): Agent is initialized with random weights. Training starts from episode 1.
- **Resume** (`--resume`): The runner searches for the latest checkpoint in the model directory, loads weights, reads the log to determine the last completed episode, and continues from there. If no checkpoint is found, training starts from scratch with a warning.

### When Models Are Saved

During training:
- **Best model** is updated whenever a new highest single-episode reward is achieved
- **Best avg100 model** is updated whenever the 100-episode rolling average reaches a new high
- **Periodic checkpoints** are saved every 100 episodes

### Why Results Can Vary Between Runs

Even with the same configuration and seed, results may differ between runs due to:
- Different hardware (CPU vs GPU, different GPU models)
- Non-deterministic CUDA operations
- Python hash randomization
- Operating system threading differences

Setting `torch.backends.cudnn.deterministic = True` (done in `utils.py`) helps but does not guarantee bit-exact reproducibility.

---

## Further Reading

- [experiment_design.md](experiment_design.md) — The 12-experiment factorial grid
- [project_architecture.md](project_architecture.md) — How the code is organized
- [results_interpretation.md](results_interpretation.md) — How to read generated plots
- [reproducibility.md](reproducibility.md) — Why results vary
