# Controls Catalog — REPLACE_ME (pack_id)

Maps the plugin's four named guardrail categories (`pii_redaction`, `prompt_injection_detection`, `escalation_policy`, `human_approval_required`) to concrete, checkable behavior for this pack's domain. Every control here must enforce one or more `regime-rules.md` requirements — do not invent a new obligation here that isn't traceable back to a cited rule.

**This file is the prose explanation. `controls-catalog.json` alongside it is the machine-readable twin — keep them in sync manually; `.json` is authoritative on disagreement.**

## Entry Format

- `control_id` — stable identifier (`<PACK-PREFIX>-CTL-###`)
- `control_name`
- `guardrail_module` — which file in the generated agent implements it (should be one of the four `guardrails/*.py` modules, or a specific function within one)
- `maps_to_regime_rule` — which requirement ID(s) from `regime-rules.md` this enforces
- `enforcement_point` — `pre-tool` / `post-tool` / `pre-output` / `approval-gate` / `periodic`
- `severity_floor` — inherited from the regime rule(s) it maps to
- `adequacy_notes` — what "correctly implemented" looks like, concretely — ideally naming a specific golden test case that would prove it

## Controls

### <PREFIX>-CTL-001 — REPLACE_ME
- **guardrail_module:** REPLACE_ME
- **maps_to_regime_rule:** REPLACE_ME
- **enforcement_point:** REPLACE_ME
- **severity_floor:** REPLACE_ME
- **adequacy_notes:** REPLACE_ME

*(repeat one entry per control — cover all four guardrail categories at minimum; `finance-aml` needed 7 controls to cover its 4 categories across multiple distinct behaviors)*

## Severity Floor at This Pack's Risk Tier

REPLACE_ME — state which controls are mandatory at this pack's `governance_floors.risk_tier` (per `core/risk-tiers.md`), and explicitly call out any control that should be treated as `non_negotiable: true` regardless of declared risk tier because it enforces a hard legal/ethical prohibition rather than a risk tradeoff — follow `finance-aml/controls-catalog.md`'s reasoning for its three `non_negotiable` controls.
