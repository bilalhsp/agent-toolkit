---
name: research-codebase-workflow
description: Set up, extend, or integrate a Python or machine-learning research repository with modular code, Hydra-style configuration, factories, experiment scripts, scheduler launchers, and durable analysis records. Use for new repository design or cross-module experiment workflows.
---

# Research codebase workflow

Use this pattern to make research code extensible and experiments recoverable. It is a decision guide, not a mandate to reorganize a mature repository.

1. In an existing repository, read its instructions and trace a real construction, training, and evaluation path. Preserve coherent active conventions; trace consumers before calling code legacy.
2. Keep models, datasets, evaluation, training, and shared utilities in separate modules or subpackages. Put orchestration in thin entry points under `scripts/` when that layout fits. Reusable code must not depend on entry points or scheduler wrappers.
3. Select extensible objects through small documented factories or registries. Keep Hydra-style component groups under `configs/`, mirroring independently selectable code.
4. Keep `slurm/` for scheduler wrappers and scheduled-job manifests. Put general downloads or maintenance in `scripts/` unless they are scheduled jobs. Keep scientific choices out of resource wrappers.
5. Before a multi-step analysis, create a durable status file. Update it as jobs, checkpoints, evaluations, and decisions change. Complete a permanent `.txt` report, update the experiment index, and only then retire the temporary status file.
6. Keep `reports/` as the permanent result archive and `reports/latex/` for requested presentation PDFs and their sources. Do not make a PDF unless asked. Preserve established report locations in an existing repository unless migration is requested.
7. Read [references/repository-workflow.md](references/repository-workflow.md) for architectural and integration decisions. Read [references/experiment-lifecycle.md](references/experiment-lifecycle.md) before setting up or changing analysis tracking. For a new project, adapt [assets/repository-scaffold.md](assets/repository-scaffold.md).
8. Explain material deviations from the pattern, especially when retaining a mature local convention.

This skill does not authorize job submission, downloads, deletion, or other actions beyond the user's request. Follow project-specific operational instructions.
