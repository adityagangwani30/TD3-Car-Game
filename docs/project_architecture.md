# Project Architecture

This document explains the overall repository structure, code organization, and data flow.

---

## High-Level Flow

```
Environment (car.py + environment.py)
        │
        ▼
  TD3/DDPG Agent (td3_agent.py / ddpg_agent.py)
        │
        ▼
  Training Loop (train.py)
        │
        ▼
  Experiment Runner (run_experiments.py)  ──►  Logs (logs/)
        │                                       │
        ▼                                       ▼
  Models (models/)                     Plot Generator (plot_metrics.py)
                                                │
                                                ▼
                                        Results (results/plots/)
                                                │
                                                ▼
                                        Research Paper / Report
```

---

## Main Files and Their Roles

### Entry Points

| File | Role |
|------|------|
| `main.py` | Single-agent training, evaluation, and demo mode. Accepts `--algo {td3,ddpg}`, `--mode {train,eval,demo}`, and other CLI flags. |
| `run_experiments.py` | Batch runner for the full 4×3 experiment grid. Runs all 12 configurations sequentially for one algorithm with multiple seeds (0, 42, 123). |
| `eval_models.py` | Multi-model evaluation and comparison. Loads saved checkpoints and runs evaluation episodes. |
| `plot_metrics.py` | Generates publication-quality plots from training logs. Supports individual, comparison, and grouped visualizations. |

### Agent Implementations

| File | Role |
|------|------|
| `td3_agent.py` | TD3 agent: Actor, twin Critic, target networks, delayed updates, target smoothing. |
| `ddpg_agent.py` | DDPG agent: Actor, single Critic, target networks. Same API as TD3 for interchangeability. |
| `replay_buffer.py` | Circular experience replay buffer backed by pre-allocated NumPy arrays. Shared by both algorithms. |

### Environment

| File | Role |
|------|------|
| `environment.py` | Gym-style `CarRacingEnv` class. Handles reset, step, reward computation (R1–R4), rendering, and metrics integration. |
| `car.py` | Car physics (bicycle model), raycasting sensors, state vector construction, collision detection. |
| `lap_timer.py` | Finish-line crossing detection and lap timing logic. |

### Infrastructure

| File | Role |
|------|------|
| `config.py` | Central configuration — all hyperparameters, screen dimensions, physics constants, experiment grid, file paths. |
| `metrics_tracker.py` | Episode-level metrics logging to JSONL format. Tracks reward, speed, crashes, laps, and network stats. |
| `utils.py` | Helpers: seed setting, headless detection, Pygame initialization, asset generation, track mask loading. |

### Notebooks

| File | Role |
|------|------|
| `colab_demo_td3.ipynb` | Google Colab notebook for TD3 experiments. |
| `colab_demo_ddpg.ipynb` | Google Colab notebook for DDPG experiments. |
| `colab_demo_both.ipynb` | Google Colab notebook for the full comparative suite (both algorithms). |

---

## How Training Starts

### Single Training Run (`main.py`)

```bash
python main.py --algo td3 --mode train --headless
```

1. `main.py` parses CLI arguments
2. Detects headless environment (auto or `--headless` flag)
3. Initializes Pygame and sets random seed
4. Creates `CarRacingEnv` with default reward mode (`shaped`) and noise
5. Creates a TD3 or DDPG agent
6. Optionally loads a checkpoint (`--resume` or `--checkpoint`)
7. Calls `train()` from `train.py`
8. Training loop runs for up to 2,000 episodes (configurable via `--max-episodes`)

### Batch Experiment Run (`run_experiments.py`)

```bash
python run_experiments.py --algo td3 --headless
```

1. Iterates through all 12 experiments from `config.EXPERIMENTS` (4 reward × 3 noise)
2. For each experiment, iterates through seeds [0, 42, 123]
3. Creates isolated log and model directories per (algo, experiment, seed)
4. Calls `train_with_config()` with the experiment-specific parameters
5. Skips completed experiments when `--resume` is used

---

## How Logs Are Generated

The `MetricsTracker` class (`metrics_tracker.py`) writes one JSON line per episode to a `training_log.jsonl` file.

Each log entry contains:
- `experiment_name`, `reward_mode`, `sensor_noise_std`, `seed`
- `episode`, `reward_total`, `reward_mean`, `reward_std`
- `length`, `speed_mean`, `speed_max`, `steering_smooth`
- `laps_completed`, `collisions`, `termination_reason`
- `reward_rolling_avg_100`, `exploration_noise`, `replay_buffer_size`

Log directory structure:
```
logs/
├── td3/
│   ├── R1_N1/
│   │   ├── seed_0/training_log.jsonl
│   │   ├── seed_42/training_log.jsonl
│   │   └── seed_123/training_log.jsonl
│   ├── R1_N2/
│   │   └── ...
│   └── R4_N3/
│       └── ...
└── ddpg/
    └── ...
```

---

## How Plots Are Generated

```bash
# Individual algorithm plots
python plot_metrics.py --algo td3

# TD3 vs DDPG comparison + grouped noise-level plots
python plot_metrics.py --compare-algos
```

`plot_metrics.py` reads JSONL logs, computes rolling averages, aggregates across seeds (mean ± std), and generates:

- **Individual plots** → `results/plots/{algo}/individual/{experiment_id}/`
- **Comparison plots** → `results/plots/comparison/`
- **Grouped plots** → `results/grouped/`

---

## How Models Are Saved

During training, models are saved to the `models/` directory:

- `{algo}_best.pth` — best single-episode reward
- `{algo}_best_avg100.pth` — best 100-episode rolling average
- `{algo}_ep{N}.pth` — periodic checkpoints every 100 episodes

For experiments, the path includes the experiment tag and seed:
```
models/{algo}/{experiment_tag}/seed_{seed}/{algo}_best.pth
```

---

## How Colab Execution Fits In

The Colab notebooks (`colab_demo_*.ipynb`):

1. Clone the repository
2. Install dependencies (`pip install -r requirements.txt`)
3. Automatically detect headless mode via `detect_headless_environment()` in `utils.py`
4. Set SDL environment variables for off-screen rendering
5. Run training and experiments in headless mode
6. Generate plots from the resulting logs
7. Provide download links for results as ZIP files

The code is identical between local and Colab execution — the headless detection and dummy Pygame driver handle the differences transparently.

---

## Further Reading

- [training_and_evaluation.md](training_and_evaluation.md) — Detailed commands for training and evaluation
- [experiment_design.md](experiment_design.md) — How the 12-experiment grid is structured
- [environment_design.md](environment_design.md) — The racing environment details
