---
name: pi-developer
description: Build or extend agentic applications that embed @earendil-works/pi-coding-agent. Use for Pi SDK backends, agent profiles, built-in SKILL.md discovery, app tools, and shared artifacts or background jobs across Pi agents.
---

# Pi Developer

**Keep the application lean, efficient, and well structured.** Reuse Pi and
existing app code where they already meet the need. Remove redundant code and
work within the scope of the change. Keep responsibilities clear and add
abstractions only when they simplify the code.

Use Pi's full runtime as the agent engine. Keep the application responsible for
identity, data access, lifecycle, and structured outputs. These instructions are
for building the app. Its agents have their own runtime skills.

## Shape the change

1. Inspect the app's entry points, pinned Pi SDK version, and existing agent
   layout. Reuse them. For a new app, use the
   [application layout](references/app-layout.md) as a starting point and build
   only the parts the request needs.
2. Add a skill to an existing profile when it shares the profile's tools,
   permissions, execution lifecycle, and output contract. Add a profile when
   those boundaries materially differ. Share one runtime and inject each
   profile's capabilities from the backend.
3. Keep app-wide behavioral rules in the profile's system prompt. Put
   domain procedures in portable `SKILL.md` folders with versioned references,
   examples, or templates as needed. Use Pi's resource loader for discovery.
   Read the [SDK integration guide](references/pi-sdk.md) when creating or
   changing the session factory, provider, tool registration, or skill discovery.
4. Use tools for retrieval, computation, validation, and publication. Let Pi
   decide the order of tool calls within the skill. Encode job state transitions,
   authorization, and output checks in the host where they can be enforced.
   Read the [jobs and artifacts guide](references/jobs-and-artifacts.md) when
   work must outlive a request or one agent consumes another's output.

## Verify the path users will take

Check that the normal CLI or API entry point selects the intended profile,
that Pi advertises and can read a bundled skill, and that its active tool set
matches the profile. Exercise one representative result from the skill through
the output contract. Use local fixtures or an injected model for repeatable
checks. Report a live model run separately when credentials or network access
are unavailable.
