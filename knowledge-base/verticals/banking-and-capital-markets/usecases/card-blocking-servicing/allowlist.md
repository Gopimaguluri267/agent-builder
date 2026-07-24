# Governed Connector Allowlist — card-blocking-servicing

Defines the only files in a generated project allowed to make outbound calls, for this usecase. `scripts/egress_gate.py` uses this list (via `allowlist.json`) to fail the build if any raw SDK/DB/HTTP call appears outside a registered connector module.

**This file is the prose explanation. `allowlist.json` alongside it is the machine-readable twin. `.json` is authoritative on disagreement.**

**Placeholder disclaimer:** `allowed_host_patterns` below are illustrative placeholders, not real endpoints.

## Connectors

### CARD-CONN-001 — Model Gateway
- **category:** model-gateway
- **module_path_pattern:** `src/*/connectors/model_gateway.py`
- **allowed_host_patterns:** organization's internal LLM gateway/proxy endpoint
- **eligibility:** must reference `../../model-catalog.json` — the selected model must have a `minimum_eligible_data_classification` at or above this project's declared `data_classification`.
- **notes:** the only file permitted to import/call a model-provider SDK.

### CARD-CONN-002 — Identity Verification Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/identity_verification_connector.py`
- **allowed_host_patterns:** organization's identity-verification/step-up-auth provider
- **notes:** runs the layered/risk-based check (`CARD-UC-R001`, grounded in `BCM-REG-004`/`BCM-TECH-003`). Read/verify-only.

### CARD-CONN-003 — Card Management Connector
- **category:** case-management
- **module_path_pattern:** `src/*/connectors/card_management_connector.py`
- **allowed_host_patterns:** organization's internal card-management system API
- **notes:** handles block (autonomous, `CARD-CTL-004`) and both reissue actions. Reissue-to-new-address must be gated behind `guardrails/human_approval_required.py` (`CARD-CTL-005`) even though it routes through this same connector.

### CARD-CONN-004 — Customer Master Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/customer_master_connector.py`
- **allowed_host_patterns:** organization's internal customer master / account system API
- **notes:** source of the address-on-file and PII categories per `core/pii-taxonomy.md`. Every read must pass through `guardrails/pii_redaction.py`.

### CARD-CONN-005 — Notification Connector
- **category:** communication
- **module_path_pattern:** `src/*/connectors/notification_connector.py`
- **allowed_host_patterns:** organization's customer notification/messaging system
- **notes:** sends the block-confirmation message — must run after `CARD-CTL-003` (identity verification) has passed.

## Explicitly Disallowed

Anywhere outside the five `module_path_pattern`s above, the following fail the build:
- Direct provider SDK usage, raw DB drivers/connection strings, raw HTTP calls to non-allowlisted hosts.
- Any code path that fulfills a reissue-to-new-address request through `card_management_connector.py` **without** first passing through `guardrails/human_approval_required.py` — checked structurally, since the two reissue actions must be distinct functions/entry points.
