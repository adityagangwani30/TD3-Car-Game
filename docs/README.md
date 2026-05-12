# 📚 Documentation Index

This folder contains detailed technical documentation for the **TD3 vs DDPG Car Game** research project. These documents are intended to support the [main README](../README.md) and the accompanying research paper by providing deeper explanations of the algorithms, environment, experiments, and results.

---

## Quick Navigation

| File | Purpose |
|------|---------|
| [td3_explanation.md](td3_explanation.md) | Explains the TD3 algorithm and how it is implemented in this project |
| [ddpg_background.md](ddpg_background.md) | Background on DDPG and why TD3 was developed to improve upon it |
| [project_architecture.md](project_architecture.md) | Overview of the repository structure, code files, and data flow |
| [environment_design.md](environment_design.md) | Details of the 2D racing environment, state/action spaces, and physics |
| [reward_system.md](reward_system.md) | Explains the four reward modes (R1–R4) and reward design considerations |
| [observation_noise.md](observation_noise.md) | Explains the three sensor noise levels (N1–N3) and their effects |
| [experiment_design.md](experiment_design.md) | Explains the 12-experiment factorial grid and metrics used |
| [training_and_evaluation.md](training_and_evaluation.md) | How to train, evaluate, and run experiments with actual commands |
| [results_interpretation.md](results_interpretation.md) | How to read and interpret the generated results and plots |
| [reproducibility.md](reproducibility.md) | Reproducibility considerations, sources of randomness, and limitations |
| [troubleshooting.md](troubleshooting.md) | Common issues and their solutions |

---

## How to Use These Docs

- **The main [README.md](../README.md) remains the quick-start guide.** It covers installation, basic usage, and a high-level project overview.
- **This docs folder provides deeper technical explanation.** Start with [project_architecture.md](project_architecture.md) for an overview, then explore specific topics as needed.
- **These docs are meant to support the README and research paper.** They bridge the gap between the code and the written analysis, making the project accessible to beginners, reviewers, and future contributors.

---

## Who Should Read What

| Reader | Recommended Starting Points |
|--------|-----------------------------|
| **Beginner** | `td3_explanation.md` → `ddpg_background.md` → `environment_design.md` |
| **Reviewer** | `experiment_design.md` → `reward_system.md` → `results_interpretation.md` |
| **Contributor** | `project_architecture.md` → `training_and_evaluation.md` → `reproducibility.md` |
| **Troubleshooting** | `troubleshooting.md` |
