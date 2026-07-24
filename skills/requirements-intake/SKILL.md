---
name: requirements-intake
description: Collects and refines business/product/functional requirements for the active project run — objective, users, outcomes, features, workflows, constraints — grounded in the governance already declared and the usecase's own rules. Use after /agent-builder:forge has produced a signed scope-bind.yaml, before generating the BRD/PRD/FRD.
when_to_use: When the user wants to gather or continue gathering requirements for an in-progress Agent Builder run, or asks to move past Scope & Bind toward defining what the agent should actually do.
argument-hint: "[project directory, if not already clear from context]"
allowed-tools: ["Read", "Write", "Glob", "AskUserQuestion", "TodoWrite"]
---

# Requirements Intake

You collect the raw material for the BRD/PRD/FRD — you do not write those documents yourself (that's `skills/requirements-doc-writer`, after `skills/requirements-validator` clears what you capture here). Your job: ask good questions, capture answers precisely, and don't re-ask anything already decided elsewhere.

Track progress with `TodoWrite`.

## Step 1 — Locate the active run and confirm the precondition

- Determine the project directory (from `$ARGUMENTS`, or ask directly if unclear).
- Read `<project-dir>/.agentbuilder/state.md`. If it doesn't exist, or no run has a `governance/scope-bind.yaml` with `verdict: ship` or `ship_with_conditions`, **stop** and tell the user to run `/agent-builder:forge` first — do not improvise governance context (data classification, risk tier, approved models, etc.) yourself.
- Use the run with the most recent `scope-bind.yaml` as the active run for this skill (should match `state.md`'s `active_run` in the normal case; if they disagree, tell the user and ask which run they mean rather than guessing).

## Step 2 — Load context (read-only, don't ask the user for any of this)

- `.agentbuilder/<run_id>/governance/scope-bind.yaml` — vertical, usecase, governance values already decided.
- The usecase's `usecase-manifest.yaml`, `usecase-rules.md`, `controls-catalog.md` (path from `scope-bind.yaml`'s `vertical`/`usecase`).
- The vertical's `regulatory-landscape.md` and `technical-standards.md`.
- `core/documentation-standards.md` — so you know what the eventual BRD/PRD/FRD will need, and can steer questions toward gathering exactly that, not more or less.

## Step 3 — Check for resumable progress

- Look for existing intake field decisions logged in `state.md` for this run (same per-field logging pattern `forge.md` uses for Scope & Bind — look for entries tagged with an intake field name).
- If `<project-dir>/.agentbuilder/<run_id>/requirements/requirements-intake.yaml` already exists and is complete, tell the user it exists, summarize it briefly, and ask whether they want to revise it or proceed straight to validation — don't silently redo work.
- If partially complete, resume from the first missing field, same resumability discipline as `forge.md`.

## Step 4 — Collect the requirements

These are open-ended by nature — use direct conversational questions, not `AskUserQuestion`'s bounded-choice format, except where noted. Ask them grouped, not all 9 in one giant message — a natural grouping is (a) objective & users, (b) outcomes & features, (c) constraints & scope, (d) dependencies & open items. Log each field to `state.md` **immediately** after capture, same as `forge.md`'s Step 5 discipline.

1. **`business_objective`** — what business problem does this agent solve, in the sponsor's own terms?
2. **`target_users` / `personas`** — who actually interacts with this agent? For each, a short narrative persona (see the usecase's own framing for a starting point — e.g. `card-blocking-servicing`'s usecase-manifest already frames this as "the cardholder directly," not an internal analyst).
3. **`expected_outcomes` / `success_criteria`** — what does success look like, ideally in terms specific enough to become KPIs later. Ask directly: "how will you know this worked?"
4. **`features`** — the concrete capabilities wanted. As each one comes up, check it against `usecase-rules.md`/`controls-catalog.json` in your head — if a requested feature looks like it conflicts with a `non_negotiable: true` control, say so **now**, conversationally, rather than silently capturing it and letting the validator discover it later. This is the "ask follow-up/intake questions" the user described — probe, don't just transcribe.
5. **`workflows`** — how does a typical interaction actually flow, step by step? Note explicitly where a human-approval gate should interrupt the flow, if the usecase's controls require one.
6. **`constraints`** — business-level constraints beyond the governance ones already in `scope-bind.yaml` (which you should restate here for completeness, not silently omit).
7. **`assumptions`** — what's being taken as given.
8. **`out_of_scope`** — explicitly what this agent will NOT do — prompt with the usecase's own hard-rule boundaries as a starting point (e.g. "this agent won't do X without a human gate, per CARD-UC-R005 — anything else you want explicitly out of scope?").
9. **`dependencies`** — upstream systems, connectors, teams this agent depends on.
10. **`open_questions`** — anything neither of you can resolve right now. Capture as-is; do not pressure the user to resolve every open question before proceeding — some genuinely carry forward into the PRD's Open Questions section for a later reader to answer.

## Step 5 — Write the intake artifact

Write `.agentbuilder/<run_id>/requirements/requirements-intake.yaml`:

```yaml
project_id: <from scope-bind.yaml>
run_id: <active run>
vertical: <from scope-bind.yaml>
usecase: <from scope-bind.yaml>
business_objective: <string>
target_users:
  - persona: <name>
    description: <string>
expected_outcomes: [<string>, ...]
features:
  - name: <string>
    description: <string>
    references: [<usecase-rules.md IDs this implements or is constrained by, if any>]
workflows:
  - name: <string>
    steps: [<string>, ...]
    approval_gate_at_step: <step index, or null>
constraints: [<string>, ...]     # include the governance constraints from scope-bind.yaml here too, restated in business language
assumptions: [<string>, ...]
out_of_scope: [<string>, ...]
dependencies: [<string>, ...]
open_questions: [<string>, ...]
captured_at: <timestamp>
```

## Step 6 — Report and hand off

Tell the user intake is captured, summarize what you got (briefly — don't re-paste the whole file), and that the next step is validation against the knowledge base (`skills/requirements-validator`) before any document gets generated. Do not generate the BRD/PRD/FRD yourself from this skill.
