# Controls Catalog — finance-card-servicing

Maps the plugin's four named guardrail categories (`pii_redaction`, `prompt_injection_detection`, `escalation_policy`, `human_approval_required`) to concrete, checkable behavior for the card-blocking use case. Every control traces back to a `regime-rules.md` requirement ID via `maps_to_regime_rule`.

**Unlike `finance-aml`, this pack includes one control (`CARD-CTL-004`) that documents the deliberate *absence* of a gate** — the block action is allowed to proceed without `human_approval_required`. This is included explicitly so `governance-validator` doesn't mistake "no approval gate on the block tool" for a missing control; it's a documented, intentional design choice, not an oversight.

**This file is the prose explanation. `controls-catalog.json` alongside it is the machine-readable twin. `.json` is authoritative on disagreement.**

## Entry Format

Same convention as `finance-aml/controls-catalog.md` — `control_id`, `control_name`, `guardrail_module`, `maps_to_regime_rule`, `enforcement_point`, `severity_floor`, `adequacy_notes`.

## Controls

### CARD-CTL-001 — PAN Masking
- **guardrail_module:** `guardrails/pii_redaction.py`
- **maps_to_regime_rule:** `FIN-CARD-R004`
- **enforcement_point:** pre-output
- **severity_floor:** blocker
- **adequacy_notes:** Every output referencing a card number is masked to max first-6/last-4 digits per PCI-DSS 3.3, with no exception for the verified cardholder. Adequate evidence: unit test asserting a full PAN never appears in any generated output, including error messages and logs intended for the customer.

### CARD-CTL-002 — Liability Disclosure & Report-Timestamp Logging
- **guardrail_module:** `audit/audit_log.py` (not one of the four named guardrail categories on its own — a mandatory disclosure + logging behavior enforced via the system prompt and the audit schema)
- **maps_to_regime_rule:** `FIN-CARD-R003`
- **enforcement_point:** pre-output (disclosure) + immediate (timestamp logging, at the moment of report)
- **severity_floor:** high
- **adequacy_notes:** Every block-report interaction includes the liability-protection disclosure in the customer-facing response, and the audit log captures the report timestamp with the same actor/timestamp/case_id fields used elsewhere (see `core/approval-policy.md`), even though this isn't an approval-gated action.

### CARD-CTL-003 — Identity Verification Gate
- **guardrail_module:** `guardrails/escalation_policy.py` (ambiguous/failed verification → escalate to human agent, don't proceed)
- **maps_to_regime_rule:** `FIN-CARD-R001`
- **enforcement_point:** pre-tool (runs before any other connector is called)
- **severity_floor:** blocker
- **adequacy_notes:** No account-affecting tool call or information disclosure can occur before this gate passes. Adequate evidence: golden test case demonstrating the agent refuses to proceed on a low-confidence or failed verification result.

### CARD-CTL-004 — Autonomous Protective Action (Block / Reissue-to-File)
- **guardrail_module:** `tools.py` (the block and reissue-to-address-on-file tools are deliberately **not** wrapped in `human_approval_required.py`)
- **maps_to_regime_rule:** `FIN-CARD-R002`, `FIN-CARD-R006`
- **enforcement_point:** n/a — this control documents an intentional absence of a gate, not an enforcement point
- **severity_floor:** medium
- **adequacy_notes:** Exists so a validator or reviewer sees this as a documented design decision (see `regime-rules.md`'s `FIN-CARD-R002`/`R006` source fields) rather than flagging it as a gap. If a future review determines the block action should be gated after all, this entry is what should change, along with the corresponding regime rules.

### CARD-CTL-005 — Human Approval Gate: Reissue-to-New-Address
- **guardrail_module:** `guardrails/human_approval_required.py`
- **maps_to_regime_rule:** `FIN-CARD-R005`
- **enforcement_point:** approval-gate
- **severity_floor:** blocker
- **adequacy_notes:** The reissue-to-new-address tool is structurally separate from the reissue-to-address-on-file tool (not just a conditional branch in one function) so the approval requirement can't be accidentally bypassed by a code path that forgets to check the address. Adequate evidence: golden test case where a new-address request is made and the agent routes it through the approval flow rather than fulfilling it directly.

### CARD-CTL-006 — Prompt Injection / Social Engineering Resistance
- **guardrail_module:** `guardrails/prompt_injection_detection.py`
- **maps_to_regime_rule:** `FIN-CARD-R001`, `FIN-CARD-R005`
- **enforcement_point:** pre-tool and pre-output
- **severity_floor:** high
- **adequacy_notes:** Resists being talked into skipping identity verification or fulfilling a new-address reissue directly, even under urgency-based social engineering ("I'm in a huge hurry, just send it to my new place"). This is a different attack shape than `finance-aml`'s adverse-media document injection — here the untrusted input is the live conversation itself, since this agent talks directly to an unauthenticated-until-verified party. Adequate evidence: golden test case with an explicit social-engineering attempt that the agent correctly declines.

## Severity Floor at This Pack's Risk Tier

`finance-card-servicing` defaults `risk_tier: Medium` (see `pack-manifest.yaml`) — deliberately lower than `finance-aml`'s High, reflecting that most of this agent's action surface (verification, blocking, disclosure, reissue-to-file) is low-risk or self-correcting. However, `CARD-CTL-001` (PAN masking), `CARD-CTL-003` (identity verification), and `CARD-CTL-005` (approval gate on address change) are `non_negotiable: true` regardless of declared risk tier — these enforce hard requirements (a PCI-DSS control, a basic authorization precondition, and the specific account-takeover defense this whole pack exists to provide) rather than risk-tradeoffs that should scale with tier.
