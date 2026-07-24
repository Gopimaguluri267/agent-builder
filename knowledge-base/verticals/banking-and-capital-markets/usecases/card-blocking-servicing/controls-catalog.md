# Controls Catalog — card-blocking-servicing

Maps the plugin's four named guardrail categories to concrete behavior for this usecase. Every control traces back to a `usecase-rules.md` requirement ID via `maps_to_usecase_rule`.

Like the old `finance-card-servicing` pack this replaces, includes one control (`CARD-CTL-004`) documenting the deliberate *absence* of a human-approval gate on the block action — so a validator doesn't mistake it for an oversight.

**This file is the prose explanation. `controls-catalog.json` alongside it is the machine-readable twin. `.json` is authoritative on disagreement.**

## Controls

### CARD-CTL-001 — PAN Masking
- **guardrail_module:** `guardrails/pii_redaction.py`
- **maps_to_usecase_rule:** `CARD-UC-R004`
- **references:** `BCM-TECH-001` (vertical layer)
- **enforcement_point:** pre-output
- **severity_floor:** blocker
- **non_negotiable:** true
- **adequacy_notes:** unit test asserting a full PAN never appears in any generated output, per `BCM-TECH-001`'s first-6/last-4 max.

### CARD-CTL-002 — Liability Disclosure & Report-Timestamp Logging
- **guardrail_module:** `audit/audit_log.py`
- **maps_to_usecase_rule:** `CARD-UC-R003`
- **references:** `BCM-REG-006` (vertical layer)
- **enforcement_point:** pre-output
- **severity_floor:** high
- **non_negotiable:** false
- **adequacy_notes:** every block-report interaction includes the liability disclosure and logs the report timestamp with actor/case_id fields per `core/approval-policy.md`'s schema.

### CARD-CTL-003 — Identity Verification Gate
- **guardrail_module:** `guardrails/escalation_policy.py`
- **maps_to_usecase_rule:** `CARD-UC-R001`
- **references:** `BCM-REG-004`, `BCM-TECH-003` (vertical layer)
- **enforcement_point:** pre-tool
- **severity_floor:** blocker
- **non_negotiable:** true
- **adequacy_notes:** no account-affecting tool call before this gate passes; golden test for a low-confidence/failed verification result.

### CARD-CTL-004 — Autonomous Protective Action (Block / Reissue-to-File)
- **guardrail_module:** `tools.py` (deliberately not wrapped in `human_approval_required.py`)
- **maps_to_usecase_rule:** `CARD-UC-R002`, `CARD-UC-R006`
- **references:** none
- **enforcement_point:** n/a — documents an intentional absence of a gate
- **severity_floor:** medium
- **non_negotiable:** false
- **adequacy_notes:** exists so this reads as a documented decision, not a gap.

### CARD-CTL-005 — Human Approval Gate: Reissue-to-New-Address
- **guardrail_module:** `guardrails/human_approval_required.py`
- **maps_to_usecase_rule:** `CARD-UC-R005`
- **references:** `BCM-REG-004`, `BCM-TECH-003` (vertical layer)
- **enforcement_point:** approval-gate
- **severity_floor:** blocker
- **non_negotiable:** true
- **adequacy_notes:** reissue-to-new-address is a structurally separate tool from reissue-to-file, not a conditional branch — golden test confirming the gate can't be bypassed.

### CARD-CTL-006 — Prompt Injection / Social Engineering Resistance
- **guardrail_module:** `guardrails/prompt_injection_detection.py`
- **maps_to_usecase_rule:** `CARD-UC-R001`, `CARD-UC-R005`
- **references:** grounded generally in `core/ai-governance-frameworks.md`'s `AIGF-004` (OWASP Agentic Top 10)
- **enforcement_point:** pre-tool
- **severity_floor:** high
- **non_negotiable:** false
- **adequacy_notes:** resists urgency-based social engineering attempting to skip verification or force a direct new-address reissue; golden test with an explicit social-engineering attempt.

## Severity Floor at This Usecase's Risk Tier

`card-blocking-servicing` defaults `risk_tier: Medium` (see `usecase-manifest.yaml`) — deliberately lower than a typical banking floor, since most of the action surface (verification, blocking, disclosure, reissue-to-file) is low-risk or self-correcting. `CARD-CTL-001`, `CARD-CTL-003`, and `CARD-CTL-005` remain `non_negotiable: true` regardless of declared risk tier.
