# Harness optimization

Use this guide to design efficient prompts from the start or to diagnose cost,
context pressure, and continuity in an existing Pi application. The same
approach works with one profile or several. Keep optimization policy close to
the workload that needs it.

## 1. Measure before choosing a cache policy

Distinguish reusable resource loading in the host, stable provider request
prefixes, and provider-side inference caching. A reused resource loader saves
host work. Byte-identical prompt and tool definitions establish a candidate
prefix. Provider usage establishes whether it was reused.

- Capture a fingerprint of the rendered system instructions and ordered tool
  definitions at the inference boundary. Compare it across different requests,
  fresh process starts, and resumed sessions. Inspect sanitized serialized
  request shapes in fixtures because the Pi adapter may reorder or wrap what it
  sends. Record profile, run mode, deployment revision, model, effective provider
  route where available, and reasoning settings. Keep raw prompts, tool
  arguments, credentials, and customer data out of telemetry.
- Track per-call input, output, cache reads and writes, time to first response,
  tool result volume, and compaction or cache-warming calls. Compare total cost,
  latency, and task success per workflow as well as cache rates. Treat an absent
  upstream counter as unknown. Separate provider usage from estimates and
  foreground inference from maintenance traffic.
- Replay a short one-shot task and a longer tool-heavy task. For the same
  provider and settings, compare cold and warm tool loops, a reopened session,
  and the first request after compaction. Include a new process if deployment
  paths or discovery order can change. A mocked stream verifies request
  stability and replay. A live route verifies caching and actual savings.

## 2. Keep the leading request stable

Build and reuse the profile's resource configuration. Keep `SYSTEM.md`,
ordered skill metadata, and tool descriptions and schemas stable by profile
and run mode. Use Pi's prompt composition to keep built-in skill discovery.
Allowlist the applicable tools before starting the session. Keep authorization
checks current even when schemas stay stable. A real change in capabilities
or policy can legitimately change the prefix.

Keep timestamps, request IDs, attachments, retrieved facts, and working memory
in the current turn or behind a scoped read tool. Use role-appropriate input for
each. For example, the stable `read_artifact` schema accepts an artifact ID.
The run binds the authorized IDs in its handler and supplies the current
attachment choices as task data. Updating the choices leaves the tool
definition byte-identical. File locations emitted by the SDK can affect
stability across deployments, so inspect the rendered prompt before changing
resource discovery.

Verify provider behavior for the pinned adapter and effective endpoint. Check
minimum cacheable tokens, cache markers or keys, retention and pricing, and
tenant boundaries before requesting a custom policy. OpenAI-compatible routers
can use different upstream implementations. Reusing a Pi session ID can help
routes that key on it, while preserving the conversation also preserves its
potential cached prefix. A cache hit still occupies model context. Include
refresh calls when assessing whether the SDK's cache warming policy pays off.
[Pi settings](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/settings.md)
· [Anthropic cache behavior](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
· [OpenAI prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching)

**OpenClaw example.** The TypeScript prompt renderer hashes stable inputs and
places a cache boundary after stable workspace context. Run-sensitive
capabilities and guidance are appended after that boundary. Its transport
layer applies provider-specific cache handling. A Pi app can use the same
principle by holding a stable system prompt and passing changing application
data with the turn, then inspecting the installed adapter's behavior.
[System prompt renderer](https://github.com/openclaw/openclaw/blob/main/src/agents/system-prompt.ts)
· [Provider cache policy](https://docs.openclaw.ai/reference/prompt-caching)

## 3. Reduce payloads before compacting history

1. **Bound new data.** Set per-result and aggregate budgets for large reads,
   retrieval, and tool output. Return a useful excerpt or complete section,
   an explicit truncation notice, and a revisioned ID or range for follow-up
   reads. Keep full drafts, datasets, logs, and evidence in authorized storage.
   Large tool-call arguments can also dominate history. For repeated draft
   submissions, consider a validated draft revision ID.
2. **Use Pi compaction.** Inspect defaults and supported overrides in the
   pinned SDK. Test output and tool-loop headroom on the actual model. Track
   compaction triggers, summary cost, and whether the result still respects
   recent corrections and the output contract. Keep canonical evidence
   outside the summary. [Pi compaction](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/compaction.md)
3. **Project old context only where needed.** If measurements still show
   repeated, reconstructible results, use a host-owned context hook supported
   by the pinned SDK. Prefer replacing oversized old result bodies with
   durable references. Deduplicate by canonical ID, revision, and range or
   content hash. Preserve recent turns, unresolved questions, user corrections,
   and tool-call/result IDs and pairing. Keep the original transcript where
   retention calls for it. Test provider serialization after projection.
   Repeatedly rewriting earlier history can lose cache reuse. For eligible
   routes, an idle or compaction boundary may be a better point to change it.
   [Pi context hooks](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md#context)

**OpenClaw example.** Its prompt preflight can try reducing oversized tool
results before escalating to compaction. Its client-side cache-TTL pruning can
retain a stable projected view for replay while preserving original tool
entries. Some routes use provider-side clearing. Start with ingress bounds in
a new app and add projection if the long-session traces justify it.
[Prompt preflight](https://github.com/openclaw/openclaw/blob/main/src/agents/embedded-agent-runner/run/attempt-prompt-preflight.ts)
· [Session pruning](https://docs.openclaw.ai/concepts/session-pruning)

## 4. Persist task facts at workflow boundaries

A Pi transcript supplies conversation replay. Compaction supplies a smaller
model-facing history. Working memory supplies specific task facts the app
must recover even after compaction, retry, or restart. A stateless task can
often use an in-memory Pi session throughout. For a durable workflow, persist
the smallest useful application snapshot in the existing job or thread store.

For a writer or reviewer, a snapshot might include objective, user-confirmed
constraints with their source message IDs, pending question and answer, draft
revision, and selected evidence IDs. Store exact passages and artifact content
in their owning stores. Keep confirmation and provenance so a model proposal
cannot silently become a user decision. Make revisions explicit when later
input supersedes a prior fact.

- Derive scope from the admitted request, such as owner, project, thread, and
  profile. Use a run ID for provisional state. Recheck scope when loading
  snapshots and dereferencing evidence. Keep memories independent across
  threads and profiles even when jobs run in the same process.
- Validate structured updates in the host and checkpoint them at meaningful
  boundaries such as saving a draft, waiting for input, and publishing.
  Coordinate checkpoint and state promotion with the same ownership or lease
  rules. Retry with the last committed state and make resume idempotent.
- Preserve the exact pending question with its reply outside the transcript.
  For a new revision run, rehydrate the specific authorized evidence it needs
  before validating references. Restoring a transcript alone does not
  repopulate a new run's evidence store.
- Supply a compact continuation payload or load the relevant snapshot fields
  on demand. Introduce cross-session recall or a separate model-based memory
  pass only when validated checkpoints miss important information and the
  application has a clear storage and access policy.

**OpenClaw example.** OpenClaw has per-agent session and workspace scopes and
can run a bounded pre-compaction memory flush when the route and permissions
allow it. For a job with typed draft and clarification actions, the host can
use those actions to save continuity state without an extra model turn.
OpenClaw's earlier embedded runtime used Pi. Its current built-in runtime
owns its own implementation, so verify SDK calls against the app's Pi version.
[Multi-agent scopes](https://docs.openclaw.ai/concepts/multi-agent)
· [Memory flush implementation](https://github.com/openclaw/openclaw/blob/main/src/auto-reply/reply/agent-runner-memory.ts)
· [Runtime architecture](https://github.com/openclaw/openclaw/blob/main/docs/agent-runtime-architecture.md)

**Mini Coding Agent example.** The Python example puts a prebuilt prefix before
working notes and reduced history. Its character limits and path-only read
deduplication illustrate why an application should check token pressure and
use revision and range when it identifies duplicates.
[Prompt assembly and history reduction](https://github.com/rasbt/mini-coding-agent/blob/main/mini_coding_agent.py#L309-L449)

## Acceptance checks

| Area | Check |
| --- | --- |
| Prefix | Equivalent same-profile requests match before changing task data. A deliberate model or tool policy change yields the expected variant. |
| Cache | Live usage and latency justify the policy on the intended route, including refresh and cache-write cost. |
| Context | Tool-heavy fixtures fit the chosen budget and keep the facts needed for valid output. Verify any projection preserves paired calls and results. |
| Continuity | Resume after a restart and after compaction. Check exact questions, corrections, evidence IDs, and artifact revisions. Cover retries and legacy sessions when applicable. |
| Isolation | Concurrent runs do not share mutable state. Another owner, project, or profile cannot read a snapshot or artifact without explicit access. |
| Rollout | Pilot on a representative long workflow and compare with a short-task control. Generalize the pieces that improve measured outcomes. |
