# Repository architecture and integration

## Start with evidence

For an existing codebase, inspect tracked files, repository instructions, configuration defaults, factories, entry points, and tests. Distinguish observed behavior from inferred intent. Identify authoritative model construction, training, evaluation, inference, output, and reporting paths. Preserve mature conventions rather than copying a scaffold over them.

## Boundaries and dependency direction

```text
scripts/ and scheduler launchers
             ↓
training and evaluation orchestration
             ↓
models, datasets, metrics, and task interfaces
             ↓
small shared utilities
```

- `models/`: architectures, model-local losses, and inference methods.
- `datasets/`: loading, preprocessing, split definitions, and batch contracts.
- `evaluation/`: metric implementations and aggregation. Reuse authoritative metrics for validation and final evaluation.
- `training/`: reusable loops, checkpoint selection, and run state.
- `scripts/`: thin Python CLIs and general maintenance or data-preparation scripts.
- `slurm/`: optional scheduler resource wrappers and manifests. Only place downloads here when the download itself is scheduled.
- `tests/`: configuration, construction, contracts, small forward passes, metrics, and checkpoint round trips.
- `docs/`: architecture, data contracts, configuration, and onboarding.
- `notebooks/`: requested exploration and plots; not the sole metric implementation.
- `reports/`: permanent results and an experiment index; `reports/latex/` for requested PDF source and output.

These are responsibilities, not required literal package paths. A mature flat package layout need not move to `src/`.

## Construction and configuration

Use a stable component name and small factory or registry for each family selected from configuration, especially models and datasets. A factory resolves the name, validates constructor inputs, and returns an object; it does not own scientific behavior. Define model inputs/outputs and dataset batch shapes separately from registration.

Maintain `configs/` with a root workflow config and groups that mirror selectable components, such as `model/`, `data/`, `training/`, and `evaluation/`. Compose explicit defaults; make CLI/manifests overrides reproducible; save the fully resolved config with each run. Give reusable architecture variants named profiles instead of relying indefinitely on long override strings. Keep scheduler limits and machine paths separate from scientific settings.

Hydra and OmegaConf are the preferred style for new Python research projects. When integrating an existing repository with a coherent alternative, assess migration cost before replacing it.

## Entry points and infrastructure

Training, evaluation, inference/generation, and data preparation may have separate entry points when control flow differs. Each should compose config, validate inputs, establish seed/logging, construct objects through factories, call reusable code, and record artifacts. Avoid experiment-specific name conditionals in general entry points.

Scheduler wrappers set resources, environment, working directory, and forwarded arguments. Manifests or config names express scientific choices and sweeps. Make caches and resource requirements machine-configurable. Check shell syntax and a small local/dry run before large submissions when feasible.

## Adding a component

1. Add implementation in its component package and define its contract.
2. Register it with the relevant factory; do not silently replace duplicate names.
3. Add a matching config profile and validate default construction.
4. Add focused tests, update documentation, and exercise it through a script or manifest.

Do not copy a trainer or evaluator merely to change hyperparameters. Preserve historical model identifiers when they carry research provenance and document what each means.
