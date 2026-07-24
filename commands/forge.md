---
description: Forge a new governed agentic system, starting from your business requirement — pick an industry vertical and use case, declare its Scope & Bind governance envelope, scaffold the project workspace, then drive the pipeline forward through requirements gathering, validation, and signed BRD/PRD/FRD generation. Resumes an in-progress run automatically if one exists, from wherever it left off.
argument-hint: "[project directory name]"
allowed-tools: ["Read", "Write", "Bash(mkdir *)", "Glob", "AskUserQuestion", "TodoWrite", "Skill"]
---

# Agent Builder: Forge

You are running the entry point of the Agent Builder plugin — the start of the journey from business requirement to production agent. This command is idempotent and resumable: every invocation first checks whether a run is already in progress for this project before deciding whether to resume it or start a new one, then **drives the pipeline forward as far as currently exists**, rather than stopping after a single stage. It is the single orchestrator — you invoke `skills/requirements-intake`, `skills/requirements-validator`, and `skills/requirements-doc-writer` yourself, in order, via the `Skill` tool; the user should not need to know those component names or invoke them manually. All three are plain skills (run inline in this same conversation), deliberately not agents invoked via `Task` — `requirements-doc-writer`'s sign-off step needs a real, live back-and-forth with the person in this session, which an isolated subagent invocation can't reliably guarantee. Architecture design, prompt generation, code generation, and deployment-artifact generation are still not built — stop cleanly at the edge of what exists, don't improvise past it.

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
- Read its frontmatter to find `active_run` and that run's `status` and **`pipeline_step`** (the run entry's furthest-completed stage — see `state.md format` below for the field's allowed values).
- **If `active_run`'s status is `in_progress`:** this is a resume. Read the full decision log for that run from `state.md` and **recap it to the user in full** before continuing (per the user's stated preference — don't just show a terse status line). Then branch on `pipeline_step`:
  - `null` / not yet `scope-and-bind-complete`: determine the next incomplete step by checking which of the Step 4 governance fields already have a logged decision for this run, and continue from the first one that doesn't (i.e. resume mid-Scope-&-Bind, proceeding through Steps 3–9 below as normal).
  - `scope-and-bind-complete`: Scope & Bind is already done for this run — **skip Steps 3–9 entirely** and jump straight to Step 10 (requirements orchestration), resuming `skills/requirements-intake`'s own mid-intake state if it has one (that skill does its own field-level resume check).
  - `requirements-complete`: everything currently built for this run is done. Tell the user so plainly (don't re-run anything), and that architecture design is the next not-yet-built pipeline step.
  - Log a `[ai]` entry noting the resume and which stage it's continuing from.
- **If `active_run` is `null`, or every existing run's status is `complete` or `abandoned`:** start a new run. Generate a run ID per the scheme above, create its folder, set it as `active_run` in `state.md`'s frontmatter, log a `[human]` entry noting a new run was started and why (e.g. "previous run complete, new attempt requested"), and proceed to Step 3 fresh — do not carry over decisions from a prior run into a new one.

## state.md format

```markdown
---
project_id: <kebab-case project directory name>
active_run: run_20260723-151204-p9k1   # null if none in progress
runs:
  - run_id: run_20260723-143022-x7q2
    status: complete       # in_progress | complete | abandoned
    pipeline_step: requirements-complete   # null | scope-and-bind-complete | requirements-complete — the furthest stage this run has reached; drives Step 2's resume branching
    vertical: banking-and-capital-markets
    usecase: card-blocking-servicing
    started_at: <ISO 8601 timestamp>
    completed_at: <ISO 8601 timestamp, or null>
  - run_id: run_20260723-151204-p9k1
    status: in_progress
    pipeline_step: scope-and-bind-complete
    vertical: banking-and-capital-markets
    usecase: card-blocking-servicing
    started_at: <ISO 8601 timestamp>
    completed_at: null
---

# Decision Log

## run_20260723-143022-x7q2 (complete)

- <timestamp> — [human] Started project
- <timestamp> — [human] Selected vertical: banking-and-capital-markets
- <timestamp> — [human] Selected usecase: card-blocking-servicing
- <timestamp> — [human] data_classification = Confidential
- <timestamp> — [human] jurisdictions = [US]
- <timestamp> — [human] approved_models = [Claude]
- <timestamp> — [human] risk_tier = Medium (usecase floor, no override)
- <timestamp> — [human] pii_data = [customer_name, account_number, ...]
- <timestamp> — [ai] Verdict: scope-and-bind validation = ship
- <timestamp> — [ai] Run marked complete

## run_20260723-151204-p9k1 (in_progress)

- <timestamp> — [human] Started new run (previous run complete, new attempt requested)
- ...
```

Every entry is `<timestamp> — [human|ai] <what happened>`, one line per decision or notable event — not a paragraph. Log every governance field decision individually (this is what makes field-level resume possible), every verdict, every run transition, and anything else materially decided by either party during this command. Append-only — never edit or remove a past entry, even if a later decision supersedes it (log the supersession as a new entry instead).

## Step 3 — Discover and present available verticals and use cases

*(Skip this step entirely if resuming a run that already has a `vertical`/`usecase` recorded in `state.md` — use those and proceed to Step 4.)*

The knowledge base is organized `knowledge-base/verticals/<vertical>/usecases/<usecase>/` — a vertical (e.g. Banking & Capital Markets) holds industry-wide standards shared across every use case in it; a use case (e.g. Card Blocking & Servicing) is the specific agent being built, and references its vertical's files rather than duplicating them.

1. List every subdirectory of `${CLAUDE_PLUGIN_ROOT}/knowledge-base/verticals/` **except** `_template`.
2. For each, read its `vertical-manifest.yaml`. **Skip any vertical whose `status` is `placeholder`, or whose `usecases/` directory contains nothing but `_template`** — it has no real use case to select yet. Don't present an empty vertical as if it were choosable.
3. Within each remaining vertical, list every subdirectory of its `usecases/` **except** `_template`, and read each one's `usecase-manifest.yaml` for `usecase_id`, `usecase_name`, `description`, and `governance_floors`.
4. Present the available (vertical, usecase) pairs to the user with `AskUserQuestion` — one option per usecase, each option's description drawn from the usecase's `description` field, noting its parent vertical. If only one usecase exists across all verticals (the expected state today — `banking-and-capital-markets/usecases/card-blocking-servicing/` is the only fully built one), you may confirm it directly with the user instead of presenting a single-option menu, but still tell them what vertical/usecase they're building against.
5. Read the chosen usecase's full `usecase-manifest.yaml`, `usecase-rules.md`, and `controls-catalog.md`, **and** its parent vertical's `regulatory-landscape.md`, `technical-standards.md`, and `model-catalog.md` — the usecase's rules reference these by ID, so you need both layers loaded to actually explain anything to the user.
6. Log the vertical and usecase selection to `state.md` and set both as this run's `vertical`/`usecase` in the frontmatter.

## Step 4 — Load the governance schema

- Read `${CLAUDE_PLUGIN_ROOT}/knowledge-base/core/governance-schema.json` for the field definitions (`GOV-001`..`GOV-005`) and `${CLAUDE_PLUGIN_ROOT}/knowledge-base/core/pii-taxonomy.json` for the PII category catalog.
- Note the chosen usecase's `governance_floors` — these narrow, not replace, the core schema's allowed values. (Today, floors are set at the usecase level, not the vertical level — `vertical-manifest.yaml` doesn't currently define its own floors, only `usecase-manifest.yaml` does. If a future vertical's manifest does define floors, treat them as an additional, stricter-or-equal constraint on top of the usecase's.)

## Step 5 — Elicit the Scope & Bind values

Use `AskUserQuestion` (multiple questions in as few calls as possible) to collect, in order — **skip any field that already has a logged decision for this run** (resume case):

1. **`data_classification`** — options are `GOV-001.allowed_values`, restricted to the usecase's floor and anything stricter.
2. **`jurisdictions`** (multi-select) — options restricted to the vertical's `jurisdictions_supported` (from `vertical-manifest.yaml`); if the user needs one the vertical doesn't support, tell them plainly (citing `open_gaps`) rather than letting them pick it anyway.
3. **`approved_models`** (multi-select) — options are `GOV-003.allowed_values`, filtered by **two** checks, both must pass: (a) a matching `model-gateway` connector exists in the usecase's `allowlist.json`; (b) per the vertical's `model-catalog.json`, the model's `minimum_eligible_data_classification` is at or above the `data_classification` already selected in step 1 above (`model-catalog.json`'s `data_classification_order` gives the ranking). Don't offer a model that fails either check — don't just warn and let the user pick it anyway.
4. **`risk_tier`** — options are `GOV-004.allowed_values`; if the user wants to go below the usecase's floor, require a written justification, not just an option pick.
5. **`pii_data`** (multi-select) — options are `pii-taxonomy.json`'s `category_name` values, described using their `description`/`redaction_treatment` fields.

Log each field's decision to `state.md` **immediately after it's captured**, not batched at the end — this is what makes resuming mid-elicitation possible.

## Step 6 — Validate and assign a verdict

Use this fixed vocabulary for the verdict — never freeform text: **`block`** (hard failure, cannot proceed), **`hold`** (needs a human decision the agent can't resolve on its own), **`ship_with_conditions`** (passes, but with a logged caveat), **`ship`** (clean pass).

Checks:
- Every selected value is a member of the allowed set offered → if not, `block` (should be unreachable since only valid options were offered, but check anyway).
- Every `approved_models` entry has a matching connector in the usecase's `allowlist.json` → if not, `block`.
- Every `approved_models` entry's `minimum_eligible_data_classification` (from the vertical's `model-catalog.json`) is at or above the selected `data_classification` → if not, `block` (should be unreachable per Step 5's filtering, but check anyway — `model-catalog.json` is an unverified research snapshot per its own disclaimer, so treat a pass here as "internally consistent," not as proof the underlying compliance claim is current; note that distinction in the logged verdict).
- `risk_tier` is at or above the usecase's floor → `ship`. Below the floor with a justification captured → `ship_with_conditions` (the condition being the override justification, which must be included in the logged verdict). Below the floor with no justification → this should be unreachable per Step 5; if it happens anyway, `block`.

Log the verdict to `state.md`. If the verdict is `block`, do not write the governance artifact — return to Step 5 for the specific field that failed. `hold` is not expected to occur in this command as currently scoped (no check here requires a human judgment call beyond what Step 5 already captures) — if you find yourself wanting to use it, stop and ask the user, don't invent a new use for it silently.

## Step 7 — Write the governance artifact

Write `<project-dir>/.agentbuilder/<active_run>/governance/scope-bind.yaml`:

```yaml
project_id: <kebab-case project directory name>
agent_name: <ask the user for this if not already clear from conversation>
vertical: <chosen vertical_id>
vertical_version: <chosen vertical's version from vertical-manifest.yaml>
usecase: <chosen usecase_id>
usecase_version: <chosen usecase's version from usecase-manifest.yaml>
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

Also update `state.md`'s frontmatter: set this run's `pipeline_step` to `scope-and-bind-complete` (this is the field Step 2 reads on future invocations to know where to resume — keep it in sync, don't just leave the copy inside `scope-bind.yaml` as the only record).

## Step 8 — Scaffold the rest of the run's workspace

Create these empty directories inside `<project-dir>/.agentbuilder/<active_run>/`, each with a one-line placeholder noting what will eventually go there: `requirements/`, `architecture/`, `prompts/`, `src/`, `tests/`, `deploy/`.

## Step 9 — Report Scope & Bind completion, then continue

Log a `[ai]` entry to `state.md` noting Scope & Bind is complete for this run (do **not** mark the run's overall `status` as `complete` yet — that's reserved for when everything currently built finishes). Tell the user, briefly:
- What was created and where (including the run ID).
- Which vertical, usecase, and governance values were selected, and the verdict.

Then **proceed directly to Step 10** — do not stop here. (This differs from earlier versions of this command, which stopped after Scope & Bind because nothing downstream existed yet. It exists now.)

## Step 10 — Invoke requirements intake

Invoke `skills/requirements-intake` via the `Skill` tool for the active run. That skill handles its own preconditions, context-loading, resumability, and conversational capture — you don't need to re-derive any of that here, just hand off to it and wait for it to report completion (it writes `requirements-intake.yaml` and hands back per its own Step 6).

Log a `[ai]` entry to `state.md` noting the handoff into requirements intake.

## Step 11 — Invoke requirements validation

Once intake reports its `requirements-intake.yaml` is written, invoke `skills/requirements-validator` via the `Skill` tool for the active run. It writes `validation-report.md` and returns a verdict (`block` / `hold` / `ship_with_conditions` / `ship`) plus, on non-`ship` verdicts, a recommendation.

Branch on the verdict:
- **`ship` or `ship_with_conditions`:** proceed to Step 12.
- **`hold`:** stop and present the specific judgment call to the user yourself — do not attempt to resolve it autonomously and do not re-invoke the validator hoping for a different answer.
- **`block`, and the report's stated validation attempt count is 1 or 2:** re-invoke `skills/requirements-intake` (Step 10) targeted at the specific fields/conflicts the report named, then re-invoke `skills/requirements-validator` (this step) again. This is the bounded refine loop — it is the validator's own report that tracks the attempt count, you're just honoring its recommendation, not deciding the bound yourself.
- **`block`, and the report says the attempt limit is reached (attempt 3+):** stop looping. Present the unresolved conflicts/gaps to the user directly and ask how they want to proceed — do not invoke intake or the validator again automatically.

## Step 12 — Invoke document generation

Once validation clears (`ship`/`ship_with_conditions`), invoke `skills/requirements-doc-writer` via the `Skill` tool for the active run. It handles the full BRD → sign-off → PRD → sign-off → FRD → sign-off sequence internally, including updating `state.md`'s `pipeline_step` to `requirements-complete` once all three are signed — you don't need to drive each document individually from here.

If the doc-writer reports a sign-off was declined for a document, that document's correction loop (per its own instructions) may itself require new information — if it says the correction requires new intake, route back to Step 10 for that specific gap rather than improvising a fix yourself.

## Step 13 — Final report and stop

Once `skills/requirements-doc-writer` reports all three documents signed, tell the user, concisely:
- All three documents exist and are signed, with their file paths.
- That architecture design, prompt generation, code generation, and deployment-artifact generation are the next pipeline steps and are **not yet implemented** — don't imply the project is further along than it is.

Do not proceed past this point in the same invocation — there is nothing further built to hand off to yet.
