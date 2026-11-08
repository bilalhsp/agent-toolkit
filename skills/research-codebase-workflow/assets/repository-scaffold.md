# Adaptable research-repository scaffold

Use placeholders and remove directories that the project does not need.

```text
<project>/
├── README.md                   # purpose, setup, primary commands
├── AGENTS.md                   # concise operational guidance
├── pyproject.toml              # packaging, dependencies, development tools
├── configs/
│   ├── <workflow>.yaml         # composed Hydra defaults
│   ├── model/                  # model profiles
│   ├── data/                   # dataset and split profiles
│   ├── training/               # optimization/checkpoint policy
│   └── evaluation/             # metrics and sampling policy
├── <package>/
│   ├── models/                 # implementations and factory
│   ├── datasets/               # loaders, contracts, factory
│   ├── evaluation/             # shared metrics and aggregation
│   ├── training/               # reusable training logic
│   └── utils/                  # narrow helpers
├── scripts/                    # entry points; general maintenance/downloads
├── slurm/                      # optional scheduler wrappers and manifests
├── tests/                      # automated construction and behavior checks
├── docs/                       # architecture and domain contracts
├── notebooks/                  # optional exploratory plots
├── analysis_status/            # temporary active-analysis handoffs
└── reports/
    ├── INDEX.md                # all analyses: title, description, state, link
    ├── <number>_<title>.txt     # permanent results and provenance
    └── latex/                  # requested PDF source and rendered PDFs
```

For a small project, combine closely related library modules and omit unused config groups, Slurm, notebooks, or PDF machinery. Retain a clear construction contract, one authoritative metric path, and durable experiment records. For a larger codebase, consider `src/<package>/` and task-specific training/evaluation modules.

Before adapting: inspect active paths; choose domain-appropriate names; identify which config groups vary independently; confirm factory contracts; separate machine settings; define checkpoint and report conventions; add only needed directories. Explain intentional deviations.
