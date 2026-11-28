# Application layout

Use a single embedded Pi backend when profiles can share deployment and request
handling. A greenfield layout can start with one profile and grow as features
arrive.

```text
agents/
  <profile>/
    SYSTEM.md
    skills/
      <skill-name>/
        SKILL.md
        references/                # When the skill needs source material
        examples/                  # When examples improve the output
        templates/                 # When output follows a fixed structure
src/
  agents.js                        # Server-owned profile registry
  pi/                              # Session creation, resource loading, scoped reader
  cli.js                           # Shared command-line entry point, when needed
  server.js                        # HTTP startup and route bindings, when needed
  <feature>/                       # Feature tools and output checks
  shared/                          # Retrieval, computation, or artifact services
  workspace/                       # Durable API, worker, and storage, when needed
.env.example
```

Resolve profile names through the server registry. Each profile can select a
provider/model, system prompt, skill root, enabled tools, and execution limits.
Cache or reuse initialized profile resources when they are stable. Create or
reopen a session for the authorized thread or run. Keep tool closures,
attachments, results, and cancellation state scoped to the invocation. Use
stable IDs and explicit ownership in service calls.

### Build the first vertical slice

1. Pin the Pi SDK and supported Node.js or Bun version. Put one profile in
   the registry and give it a small system prompt and a skill with an example
   or reference for a realistic task.
2. Wire a CLI command or API route through the same profile runner that later
   entry points can call. Resolve the profile and its capabilities on the host.
   Define the answer or artifact contract and implement its validation before
   widening tool access.
3. Set up discovery and an explicit tool surface using the
   [SDK guide](pi-sdk.md). Run a fixture from the public entry point and
   confirm it can read the selected resource and produce a validated result.
4. Add persistence when the workflow needs history, a disconnect-safe run, or
   clarification. Use the [jobs guide](jobs-and-artifacts.md) for those paths.
   Add another profile by registering its model, prompt, skill roots, and tool
   factories in the same runtime. Reuse services through their access-checked
   interfaces. Deploy separately if isolation or operating needs require it.

### Profile or skill

| Need | Shape |
| --- | --- |
| Generate scripts for several documented APIs | One scripting profile with a skill per tool or tool family. Bundle versioned manual extracts and known-good examples. Return source, references actually read, and the validation status. Keep generated code as data unless the application calls for a controlled execution environment. |
| Grounded chat with simple calculations | One profile with retrieval and approved formulas. Let the agent select a formula and let application code evaluate it with typed, unit-aware inputs. A separate calculator UI can recompute without calling the model. |
| Analyze uploaded datasets and produce interactive plots | A profile with approved parsers and deterministic metrics. Return computed values, units, data provenance, and a renderer-ready plot specification. |
| Write cited reports from standards, evidence, and saved plots | A profile with scoped retrieval, artifact reads, and report submission. Give each supported report type a skill and a versioned template. Use a durable job when clarification or report generation outlives the HTTP request. |

These are examples of boundaries. Match the user's app and preserve established
entry points when adding profiles. Add a shared service when more than one
feature uses the same data contract or permission rules.

### Configuration

Commit an `.env.example` with non-secret defaults and empty credentials. Match
the environment keys to the settings the code reads. Keep model provider
credentials, backend auth tokens, and database URLs server-side. Show which
settings a scaffold reserves for future integration. Make local environment
loading explicit in the entry point or package scripts, and let deployment
environment variables take precedence.

Use the requested model and provider. When using Hugging Face Inference Providers,
Pi's provider ID is `huggingface` and the token variable is `HF_TOKEN`. Use a
fine-grained token permitted to call Inference Providers and verify that the
selected routed model supports tool calls in the deployment. Keep per-profile
model overrides optional. [Hugging Face Pi integration](https://huggingface.co/docs/inference-providers/en/integrations/pi)
