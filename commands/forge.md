---
description: Forge a new governed agentic system, starting from your business requirement — pick a knowledge-base pack, declare its Scope & Bind governance envelope, and scaffold the project workspace. Resumes an in-progress run automatically if one exists.
argument-hint: "[project directory name]"
allowed-tools: ["Read", "Write", "Bash(mkdir *)", "Glob", "AskUserQuestion", "TodoWrite"]
---

# Agent Builder: Forge

You are running the entry point of the Agent Builder plugin — the start of the journey from business requirement to production agent. This command is idempotent and resumable: every invocation first checks whether a run is already in progress for this project before deciding whether to resume it or start a new one. Do not attempt requirements gathering, architecture design, or code generation in this command — those are later, not-yet-built pipeline steps. Stop once the governance envelope is written and tell the user what comes next.

Track your progress through the steps below with `TodoWrite` so the user can see where you are.

## Directory shape

All of this plugin's own tracking/bookkeeping lives under a hidden `.agentbuilder/` folder inside the project directory, kept separate from the project's actual deliverable files:

```
<project-dir>/
  .agentbuilder/
    state.md              # single source of truth: run index + full decision log
    run_20260723-143022-x7q2/   # first attempt — intermediate artifacts for that run (see "Run ID generation" in Step 2)
      governance/
        scope-bind.yaml
      requirements/
      architecture/
      prompts/
      src/
      tests/
      deploy/
    run_20260723-151204-p9k1/   # a later attempt, if one was started (see Step 2)
      ...
```

**Open question, not yet resolved — do not silently decide this:** whether a *completed* run's artifacts eventually get promoted/copied out of `.agentbuilder/run_xxxxxx/` to the clean project root as the actual deliverable, or whether everything permanently lives inside `.agentbuilder/`. This matters more once the deployment-artifacts pipeline step exists. For now, leave everything inside `.agentbuilder/run_xxxxxx/` and do not invent a promotion step.

## Step 1 — Determine the project directory

- If `$ARGUMENTS` was provided, use it as the project directory name.
- Otherwise, ask the user (a simple question, not `AskUserQuestion` — just ask directly) what to call this project. Suggest a kebab-case name based on what they tell you about the agent.
- If the directory does not exist, create it — this is a brand-new project.
- If it exists, proceed to Step 2 to check for prior state before doing anything else.

## Step 2 — Check for existing state and decide: resume or start a new run

- Look for `<project-dir>/.agentbuilder/state.md`.

**Run ID generation (applies any time a new run is created, below):** `run_<YYYYMMDD-HHMMSS>-<4 random alphanumeric chars>`, e.g. `run_20260723-143022-x7q2`. Timestamp makes runs sortable by creation time at a glance; the random suffix guarantees uniqueness even if two runs were somehow started in the same second. Before using a generated ID, check it doesn't already exist as a folder under `.agentbuilder/` or as an entry in `state.md`'s `runs` list — regenerate the random suffix (keep the timestamp) if it collides. Never reuse or guess an ID; never use a bare sequential number.

**If `state.md` does not exist** (brand-new project):
- Create `.agentbuilder/` and initialize `state.md` with an empty `runs` list and `active_run: null` in its frontmatter (see "state.md format" below).
- Generate a run ID per the scheme above, create that folder, set it as `active_run`, log a `[human]` entry noting the project was started, and proceed to Step 3.

**If `state.md` exists:**
- Read its frontmatter to find `active_run` and that run's `status`.
- **If `active_run`'s status is `in_progress`:** this is a resume. Read the full decision log for that run from `state.md` and **recap it to the user in full** before continuing (per the user's stated preference — don't just show a terse status line). Then determine the next incomplete step by checking which of the Step 4 governance fields already have a logged decision for this run, and continue from the first one that doesn't. Log a `[ai]` entry noting the resume.
- **If `active_run` is `null`, or every existing run's status is `complete` or `abandoned`:** start a new run. Generate a run ID per the scheme above, create its folder, set it as `active_run` in `state.md`'s frontmatter, log a `[human]` entry noting a new run was started and why (e.g. "previous run complete, new attempt requested"), and proceed to Step 3 fresh — do not carry over decisions from a prior run into a new one.

## state.md format

```markdown
---
project_id: <kebab-case project directory name>
active_run: run_20260723-151204-p9k1   # null if none in progress
runs:
  - run_id: run_20260723-143022-x7q2
    status: complete       # in_progress | complete | abandoned
    pack: finance-card-servicing
    started_at: <ISO 8601 timestamp>
    completed_at: <ISO 8601 timestamp, or null>
  - run_id: run_20260723-151204-p9k1
    status: in_progress
    pack: finance-card-servicing
    started_at: <ISO 8601 timestamp>
    completed_at: null
---

# Decision Log

## run_20260723-143022-x7q2 (complete)

- <timestamp> — [human] Started project
- <timestamp> — [human] Selected pack: finance-card-servicing
- <timestamp> — [human] data_classification = Confidential
- <timestamp> — [human] jurisdictions = [US]
- <timestamp> — [human] approved_models = [Claude]
- <timestamp> — [human] risk_tier = Medium (pack floor, no override)
- <timestamp> — [human] pii_data = [customer_name, account_number, ...]
- <timestamp> — [ai] Verdict: scope-and-bind validation = ship
- <timestamp> — [ai] Run marked complete

## run_20260723-151204-p9k1 (in_progress)

- <timestamp> — [human] Started new run (previous run complete, new attempt requested)
- ...
```

Every entry is `<timestamp> — [human|ai] <what happened>`, one line per decision or notable event — not a paragraph. Log every governance field decision individually (this is what makes field-level resume possible), every verdict, every run transition, and anything else materially decided by either party during this command. Append-only — never edit or remove a past entry, even if a later decision supersedes it (log the supersession as a new entry instead).

## Step 3 — Discover and present available packs

*(Skip this step entirely if resuming a run that already has a `pack` recorded in `state.md` — use that pack and proceed to Step 4.)*

- List every subdirectory of `${CLAUDE_PLUGIN_ROOT}/knowledge-base/packs/` **except** `_template`.
- For each, read its `pack-manifest.yaml` and extract `pack_id`, `pack_name`, `description`, and `governance_floors`.
- Present these to the user with `AskUserQuestion` so they pick exactly one pack for this project.
- Read the chosen pack's full `pack-manifest.yaml`, `regime-rules.md`, and `controls-catalog.md`.
- Log the pack selection to `state.md` and set it as this run's `pack` in the frontmatter.

## Step 4 — Load the governance schema

- Read `${CLAUDE_PLUGIN_ROOT}/knowledge-base/core/governance-schema.json` for the field definitions (`GOV-001`..`GOV-005`) and `${CLAUDE_PLUGIN_ROOT}/knowledge-base/core/pii-taxonomy.json` for the PII category catalog.
- Note the chosen pack's `governance_floors` — these narrow, not replace, the core schema's allowed values.

## Step 5 — Elicit the Scope & Bind values

Use `AskUserQuestion` (multiple questions in as few calls as possible) to collect, in order — **skip any field that already has a logged decision for this run** (resume case):

1. **`data_classification`** — options are `GOV-001.allowed_values`, restricted to the pack's floor and anything stricter.
2. **`jurisdictions`** (multi-select) — options restricted to the pack's `jurisdictions_supported`; if the user needs one the pack doesn't support, tell them plainly (citing `open_gaps`) rather than letting them pick it anyway.
3. **`approved_models`** (multi-select) — options are `GOV-003.allowed_values`, filtered to those with a matching `model-gateway` connector in the pack's `allowlist.json`.
4. **`risk_tier`** — options are `GOV-004.allowed_values`; if the user wants to go below the pack's floor, require a written justification, not just an option pick.
5. **`pii_data`** (multi-select) — options are `pii-taxonomy.json`'s `category_name` values, described using their `description`/`redaction_treatment` fields.

Log each field's decision to `state.md` **immediately after it's captured**, not batched at the end — this is what makes resuming mid-elicitation possible.

## Step 6 — Validate and assign a verdict

Use this fixed vocabulary for the verdict — never freeform text: **`block`** (hard failure, cannot proceed), **`hold`** (needs a human decision the agent can't resolve on its own), **`ship_with_conditions`** (passes, but with a logged caveat), **`ship`** (clean pass).

Checks:
- Every selected value is a member of the allowed set offered → if not, `block` (should be unreachable since only valid options were offered, but check anyway).
- Every `approved_models` entry has a matching connector in `allowlist.json` → if not, `block`.
- `risk_tier` is at or above the pack's floor → `ship`. Below the floor with a justification captured → `ship_with_conditions` (the condition being the override justification, which must be included in the logged verdict). Below the floor with no justification → this should be unreachable per Step 5; if it happens anyway, `block`.

Log the verdict to `state.md`. If the verdict is `block`, do not write the governance artifact — return to Step 5 for the specific field that failed. `hold` is not expected to occur in this command as currently scoped (no check here requires a human judgment call beyond what Step 5 already captures) — if you find yourself wanting to use it, stop and ask the user, don't invent a new use for it silently.

## Step 7 — Write the governance artifact

Write `<project-dir>/.agentbuilder/<active_run>/governance/scope-bind.yaml`:

```yaml
project_id: <kebab-case project directory name>
agent_name: <ask the user for this if not already clear from conversation>
pack: <chosen pack_id>
pack_version: <chosen pack's version from pack-manifest.yaml>
governance:
  data_classification: <selected>
  jurisdictions: [<selected>]
  approved_models: [<selected>]
  risk_tier: <selected>
  risk_tier_override_justification: <only if applicable>
  pii_data: [<selected>]
verdict: <ship | ship_with_conditions>
created_at: <current date>
pipeline_step: scope-and-bind-complete
```

## Step 8 — Scaffold the rest of the run's workspace

Create these empty directories inside `<project-dir>/.agentbuilder/<active_run>/`, each with a one-line placeholder noting what will eventually go there: `requirements/`, `architecture/`, `prompts/`, `src/`, `tests/`, `deploy/`.

## Step 9 — Report and stop

Log a final `[ai]` entry to `state.md` noting Scope & Bind is complete for this run (do **not** mark the run's overall `status` as `complete` — that's reserved for when the full pipeline finishes, which is beyond this command's scope). Tell the user, concisely:
- What was created and where (including the run ID).
- Which pack and governance values were selected, and the verdict.
- That requirements gathering, architecture design, prompt generation, code generation, and deployment-artifact generation are the next pipeline steps and are **not yet implemented** — don't imply the project is further along than it is.

Do not proceed past this point in the same invocation.
