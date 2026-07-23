# Build Scope (Core)

Domain-agnostic. Defines what kinds of agents this plugin is even allowed to produce — the architecture options `architecture-recommender` may choose from, and what `egress_gate.py`/`validate_architecture.py` treat as in-scope vs. a build failure. This is separate from the governance envelope (`governance-schema.md`, which constrains *data*) — this file constrains the *shape of the system itself*.

**This file is the prose explanation. `build-scope.json` alongside it is the machine-readable twin — `validate_architecture.py` loads the `.json` to check a recommended/generated architecture stays inside allowed values. `.json` is authoritative on disagreement.**

## Entry Format

- `field_id` — stable identifier (`BS-###`)
- `field_name` — the key used in `architecture/architecture-decisions.yaml`
- `allowed_values` — each tagged `status`: `supported` (this plugin can generate it today), `planned` (on the roadmap, not yet buildable — recommending it should prompt a fallback, not a silent substitution), or `unsupported` (explicitly out of scope, e.g. because it can't be made to satisfy the egress gate)
- `description`
- `consumed_by`
- `validation_rule`

---

## BS-001 — agent_archetype

- **allowed_values:**
  - `single-agent` — `status: supported`
  - `multi-agent` — `status: planned` (v1 ships single-agent only; the pipeline structure doesn't preclude multi-agent later, but no template/validator exists for it yet)
- **description:** Whether the generated system is one agent or an orchestrated set of multiple agents.
- **consumed_by:** `architecture-recommender`, `codegen-agent`
- **validation_rule:** must be `supported` for v1; if `architecture-recommender` proposes `planned`, it must say so explicitly and fall back to `single-agent` rather than attempting to generate something no template supports.

## BS-002 — pattern

- **allowed_values:**
  - `tool-calling` — `status: supported`
  - `react` — `status: supported`
  - `rag` — `status: supported`
  - `reflection` — `status: planned`
- **description:** The reasoning/execution pattern the agent uses. These are not mutually exclusive in practice (a ReAct agent commonly does tool-calling), but the project must declare its primary pattern for template selection.
- **consumed_by:** `architecture-recommender`, `codegen-agent` (selects scaffold variant)
- **validation_rule:** must be `supported`.

## BS-003 — memory

- **allowed_values:**
  - `none` — `status: supported`
  - `short` (conversation/session-scoped) — `status: supported`
  - `long` (persistent across sessions) — `status: planned` (raises its own governance questions — what's the retention policy for persisted case data under `data_classification`? — not resolved yet, so not offered in v1)
- **description:** What memory the agent has beyond the current task/session.
- **consumed_by:** `architecture-recommender`, `codegen-agent`
- **validation_rule:** must be `supported`; if `long` is genuinely required by the use case, `architecture-recommender` must flag it as a gap rather than silently building an unsupported persistence layer.

## BS-004 — framework

- **allowed_values:**
  - `langgraph` — `status: supported` (first-class v1 target, full template at `templates/agent-scaffold/langgraph/`)
  - `plain-python` — `status: planned`
  - `google-adk` — `status: planned`
  - `openai-agents-sdk` — `status: planned`
- **description:** The agent framework the generated code is built on. Template system is structured so `planned` frameworks can be added later without redesigning this file or the pipeline — only `templates/agent-scaffold/<framework>/` and this list need to change.
- **consumed_by:** `skills/codegen/SKILL.md`, `codegen-agent`, `egress_gate.py` (some detections, e.g. what counts as a "framework-internal" call vs. a raw provider call, are framework-specific)
- **validation_rule:** must be `supported`; `codegen-agent` must refuse to generate code for a `planned`/`unsupported` framework rather than improvising an untemplated scaffold.

## BS-005 — deployment_platform

- **allowed_values:**
  - `containerized` (Docker, deployable to any container runtime) — `status: supported`
  - `serverless` — `status: planned`
- **description:** Target runtime shape for the deployment artifacts (`deploy/` folder, IaC).
- **consumed_by:** `skills/deployment-iac/SKILL.md`, `iac-generator`
- **validation_rule:** must be `supported`.

---

## Notes

- "Planned" is not a permission to build it anyway with best effort — `architecture-recommender` must present it as a known gap and recommend the nearest `supported` alternative, so the requirements/architecture documents stay honest about what this plugin can actually deliver today versus what the use case might ideally want.
- A pack may further restrict (never widen) these — e.g. a future high-risk pack could disallow `rag` if ungrounded retrieval is judged too risky for that domain — but that restriction lives in the pack's own manifest, not by editing this core file.
