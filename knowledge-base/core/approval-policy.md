# Approval Policy (Core)

Domain-agnostic. Defines what `human_approval_required` actually means and how it must be implemented, both in the build pipeline (a validator checks the pattern exists) and in the generated agent's own runtime code (`guardrails/human_approval_required.py`).

**MD only — no JSON twin.** The enforceable contract here (the log schema) is realized directly in code — `templates/agent-scaffold/_shared/guardrails/human_approval_required.py.tmpl` and `templates/agent-scaffold/_shared/audit/audit_log.py.tmpl` — not parsed from this file by any script. This file is the specification those templates must satisfy, read by `codegen-agent` and by a human reviewing the generated code.

## The pattern (validated precedent)

`vyayasan/kyc-analyst` (see `ignore_dir/research-sources.md` §C.1) proves this pattern works in a real, shipped Claude Code plugin for this exact domain — 17 mandatory stagegates, none of which auto-proceed on silence:

```
GATE → PRESENT evidence → WAIT for explicit human input → PROCEED only on explicit consent
```

Every action a project's `scope-bind.yaml` lists under `human_approval_gate.actions_requiring_approval` must follow this pattern in the generated agent, both at build time (this plugin's own conversational steps, e.g. confirming a framework choice) and at runtime (the deployed agent's own code, e.g. before filing/finalizing something).

## Rules

1. **No silent proceed.** The agent must never treat the absence of a response, a timeout, or an ambiguous reply as consent. Only an explicit, recognized affirmative signal (e.g. a specific approval action/keyword) counts.
2. **Evidence must be presented before consent is requested.** The human being asked to approve must be shown what they're approving — the specific claim, the specific evidence backing it, and what will happen if they approve — not just "OK to proceed?"
3. **Every approval is logged**, with these fields (the fixed schema `audit_log.py.tmpl` must implement):
   - `actor` — who gave the approval (identity, not just "a human")
   - `timestamp` — when
   - `case_id` (or equivalent project/task identifier) — what it applies to
   - `evidence_hash` — a hash of the evidence that was presented at the moment of approval, so the log is tamper-evident and provably tied to what the approver actually saw (not what the evidence later became)
4. **Denial is a valid, first-class outcome**, not an error path. The agent must have a defined behavior for "human declined" (e.g. return to investigation, escalate to a different reviewer) — never just retry the same request.
5. **The approval gate itself cannot be bypassed by configuration the agent controls.** `scope-bind.yaml`'s `human_approval_gate.actions_requiring_approval` list is set at Scope & Bind time (Step 1) and is read-only to every later pipeline step and to the deployed agent — nothing downstream can silently shrink this list.

## Relationship to risk tier

`risk-tiers.md` determines *how much* of the agent's action surface requires this gate (Low: mostly none; Medium: anything leaving the immediate working context; High: all non-read-only actions). This file defines *how* the gate itself must behave once required — the mechanics are constant across tiers, only the coverage changes.
