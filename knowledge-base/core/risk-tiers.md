# Risk Tiers (Core)

Domain-agnostic. Defines what `risk_tier` (`GOV-004` in `governance-schema.md`) actually means in practice — both for the build pipeline (which controls become mandatory) and for the deployed agent's own behavior (via the `{risk_tier_behavior}` slot in `governance-schema.md`'s `GOV-004` `prompt_directive`).

**MD only — no JSON twin.** Nothing in `scripts/` parses this file directly. Per-control mandatory/recommended status for a given tier lives in each pack's `controls-catalog.json` (`severity_floor` field), which is what `governance-validator` actually checks. This file is read by `prompt-engineer` (to fill `{risk_tier_behavior}`) and by humans/LLM agents reasoning about what a tier implies.

## Tier Definitions

### Low

- **What it means:** Read-only or low-blast-radius actions; no direct effect on a customer, account, or regulatory obligation without a separate, already-governed system acting on the output.
- **Mandatory posture:** Standard guardrails (PII redaction, prompt-injection detection) required; human approval gate and heavy audit logging are recommended, not mandatory, unless a pack overrides this.
- **`risk_tier_behavior` text:** "Because this agent operates at Low risk tier, you may complete read-only research, summarization, and analysis tasks without pausing for approval. Still flag anything ambiguous or evidence-poor rather than guessing."

### Medium

- **What it means:** Agent output materially influences a downstream human or system decision, but does not itself execute an irreversible or regulated action.
- **Mandatory posture:** Standard guardrails required; escalation policy required (agent must flag low-confidence or contradictory findings rather than silently picking one); human approval required before any output leaves the immediate working context (e.g. before it's attached to a case file or forwarded).
- **`risk_tier_behavior` text:** "Because this agent operates at Medium risk tier, treat your output as an input to a human decision, not the decision itself. Flag low-confidence findings explicitly rather than presenting them with unwarranted certainty, and route anything leaving your immediate working context through the required approval step."

### High

- **What it means:** Agent's domain involves actions that are legally consequential, irreversible, or could cause direct harm to a customer/counterparty/the institution if wrong (e.g. anything touching a regulatory filing, account restriction, or customer communication in a regulated context). This is the default floor for regulated-industry packs (e.g. `finance-aml` defaults here — see its `pack-manifest.yaml`).
- **Mandatory posture:** All standard guardrails mandatory; human approval gate mandatory before any non-read-only action, logged per `approval-policy.md`'s schema; every material claim must cite the specific evidence it's based on; escalate rather than infer whenever evidence is missing, stale, or contradictory.
- **`risk_tier_behavior` text:** "Because this agent operates at High risk tier, treat all non-read-only actions as requiring explicit, logged human approval before execution — never proceed on silence or assumed consent. Cite the specific evidence backing any material claim before presenting it for review, and escalate explicitly rather than filling a gap with a plausible-sounding guess."

## Overriding a pack's risk-tier floor

If a pack sets a floor (e.g. `finance-aml` requires at least `High`), a project may not declare a lower tier without an explicit, logged override justification captured in `scope-bind.yaml` (see `governance-schema.json`'s `GOV-004.validation_rule.override_requires_justification`). The override justification itself becomes part of the audit trail — "we chose Medium because X" must be defensible to a reviewer, not just a config toggle.
