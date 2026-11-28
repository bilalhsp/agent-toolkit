---
name: pi-developer
description: Design, build, or optimize applications that embed @earendil-works/pi-coding-agent. Use for new Pi harnesses, agent profiles and skills, tool integration, prompt caching, context budgets, and session or job continuity.
---

# Pi Developer

Use Pi for the model and tool loop, skill discovery, session replay, and
compaction. Keep profile selection, authorization, application state,
deterministic computation, and output validation in the host. The developer
skill guides the application build. Application agents load their own skills.

## Build the harness

1. For a new app, use the [application layout](references/app-layout.md).
   Start with one profile, one normal entry point, and one validated result.
   For an existing app, trace the user-facing entry points and pinned SDK first.
2. Share the Pi session factory. Add a skill when a task shares a profile's
   capabilities, permissions, lifecycle, and output contract. Add a profile
   when those boundaries materially differ. Keep model-visible conversation,
   mutable tool state, and working memory scoped to each authorized session.
3. Follow the [SDK integration guide](references/pi-sdk.md) for provider setup,
   profile prompts, built-in skill discovery, explicit tool allowlists, and
   scoped resource reads. Put procedures and examples in the skill and load
   them on demand.
4. Follow the [jobs and artifacts guide](references/jobs-and-artifacts.md) when
   runs must survive disconnects or clarification, or when a profile consumes
   another's work. Pass authorized artifact references between profiles.

## Optimize and verify

Read [harness optimization](references/harness-optimization.md) when designing
or tuning prompts, long sessions, or working memory. It includes a baseline
procedure, a rollout order, and concrete OpenClaw and Mini Coding Agent examples.

Keep stable profile instructions and tool declarations ahead of changing task
context. Measure provider cache usage before adding custom cache behavior.
Bound tool outputs before tuning Pi compaction. Persist essential facts as
validated task state whenever the workflow requires continuity independent of
the transcript.

Exercise the normal entry point with a local fixture or injected model. Check
profile selection, advertised skills, permitted and denied reads, active tools,
and a representative output contract. For durable flows, reopen the session and
check the specific facts and artifacts needed to finish. Report live inference
and cache results separately from offline verification.
