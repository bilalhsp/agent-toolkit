# Jobs and shared artifacts

## Long-running work

Use a background job if the user must be able to close the browser, return
later, or answer a clarification without holding a worker. A create endpoint
returns a job ID. The worker persists the Pi session and user-visible state.
The status endpoint returns progress, clarification questions, or the final
result. Polling is sufficient for a first version.

- Make job claims exclusive. Resume a session with the authorized job's saved
  state. Persist the pending question and release the worker while awaiting
  user input. Re-queue on reply with the same session, and handle repeated
  requests without duplicating work or outputs. Advance the committed
  checkpoint under the same lease or other ownership check used to publish
  output. Keep uncommitted retries from replacing the latest good state.
- Apply job-specific limits for time, tool calls, retrieval volume, and model
  spend as appropriate. A client disconnect can end progress delivery without
  canceling the job. Use the job's lifetime to govern cancellation.
- Store the brief, exact evidence, and completed artifacts externally.
  Compaction can summarize the transcript and reduce context size. Stable
  evidence IDs let a later drafting turn recover excerpts and citations.
  For the task facts that should survive compaction and be restored alongside
  the session, see [working memory](harness-optimization.md#4-persist-task-facts-at-workflow-boundaries).
  [Pi compaction](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/compaction.md)

## Analytics-to-report handoff

Save the plot and findings after deterministic computation. The frontend
attaches the resulting artifact ID to the report request. The backend checks
project and attachment access before returning the artifact through a
read-only tool. The analytics and report agents can run at different times.

| Artifact field | Purpose |
| --- | --- |
| `id`, `schemaVersion`, title | Identify and interpret a saved revision. |
| `metrics` | Store computed values with units, scenarios, labels, and locators such as series or bus IDs. |
| `plot` | Store a declarative plot specification or a reference to it, a caption, and an optional rendered asset for export. |
| `provenance` | Point to the source dataset or run and record the relevant filters, metric definitions, and computation version. |

Keep ownership in backend storage and derive access from authenticated
context. Bind the read tool to that context and to the request's attachment
list. Return the compact metric and provenance summary to the report agent.
The renderer can load the full artifact to insert a figure for the frontend or
export. Keep artifacts immutable or cite a specific revision so reports remain
reproducible.

## Grounding and output

Share retrieval across profiles when they use the same authorized corpus.
Return small passages with stable source IDs, edition, and clause or page.
Keep evidence IDs distinct from figure IDs. Render citations from stored source
records and verify that each cited ID exists and is authorized. Separately
check whether the passage supports the claim when the task needs that level
of review. Use a versioned report template and check required sections at
submission or rendering.
