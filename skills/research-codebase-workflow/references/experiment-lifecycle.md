# Durable experiment and report lifecycle

## One analysis, one temporary handoff

Create `analysis_status/<analysis-slug>.md` before asynchronous jobs or a multi-stage analysis. It must let another agent resume without chat history. Record:

- objective, comparison arms, data/splits, seeds, and baseline provenance;
- model/config profiles, material overrides, training budget, and checkpoint-selection rule;
- evaluation protocol, metrics, aggregation, and intended report title;
- exact commands or manifests, job IDs and states, relevant log/artifact paths, and selected checkpoints;
- decisions, deviations, failures, and the next safe action.

Update the file after meaningful state changes. A job disappearing from the queue is not evidence of success: check its exit status and relevant logs. Do not edit generated logs or copy large artifacts into the status record.

## Permanent report and index

Default to `reports/<sequence>_<analysis-slug>.txt`, using an increasing sequence and the user's title when supplied. Begin comparisons with brief findings and a readable fixed-width summary table unless another format was requested. Put detailed metrics, variability, methods, job IDs, checkpoint choices, and limitations below. Separate observed results from interpretation and incomplete arms.

Maintain `reports/INDEX.md` as a compact catalog of all analyses. Each row has an identifier/title, description (150 words maximum; shorter is better), state, and link to the active status file or permanent report. It is a navigation index, not a duplicate report. Update its row from running to complete.

```markdown
| ID / title | Description | State | Record |
| --- | --- | --- | --- |
| <number / title> | <brief purpose and comparison> | running / complete | <status or report link> |
```

When a PDF is requested, keep the `.txt` report as the durable evidence record and place presentation source and PDF under `reports/latex/`, linking them from report and index. Follow an established active location in an existing repository unless the owner requests migration.

## Completion rule

Mark complete only after required runs/evaluations are checked, results and selected checkpoints are recorded, and the permanent report is written. Update the index to point to the report. Retire the temporary status file only when all useful provenance is preserved elsewhere. For a stopped or inconclusive analysis, retain a report explaining that outcome.

For a new repo, create the status directory and index when the first analysis needs them; empty scaffolding is optional.
