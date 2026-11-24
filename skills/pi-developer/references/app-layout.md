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
  <feature>/                       # Feature tools, output checks, CLI or API adapter
  shared/                          # Services used by multiple features
  server.js                        # HTTP entry point, when needed
  jobs/                            # Persisted work, when needed
.env.example
```

Resolve profile names through the server registry. Each profile can select a
provider/model, system prompt, skill root, enabled tools, and execution limits.
Keep request state such as tool results and attachments scoped to the current
run. Use stable IDs and explicit ownership in service calls.

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
