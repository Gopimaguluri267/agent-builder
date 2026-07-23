# Governed Connector Allowlist — finance-card-servicing

Defines the only files in a generated project allowed to make outbound calls, for the card-blocking use case. `scripts/egress_gate.py` uses this list (via `allowlist.json`) to fail the build if any raw SDK/DB/HTTP call appears outside a registered connector module.

**This file is the prose explanation. `allowlist.json` alongside it is the machine-readable twin. `.json` is authoritative on disagreement.**

**Placeholder disclaimer:** `allowed_host_patterns` below are illustrative placeholders, not real endpoints — replace with your organization's actual governed connector registry before production use.

## Entry Format

Same convention as `finance-aml/allowlist.md` — `connector_id`, `connector_name`, `category`, `module_path_pattern`, `allowed_host_patterns`, `approved_for_jurisdictions`, `approved_for_data_classification`, `notes`.

## Connectors

### CARD-CONN-001 — Model Gateway
- **category:** model-gateway
- **module_path_pattern:** `src/*/connectors/model_gateway.py`
- **allowed_host_patterns:** organization's internal LLM gateway/proxy endpoint
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Same role as `finance-aml`'s equivalent connector — the only file permitted to import/call a model-provider SDK.

### CARD-CONN-002 — Identity Verification Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/identity_verification_connector.py`
- **allowed_host_patterns:** organization's identity-verification/step-up-auth provider
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Runs the layered/risk-based verification check (`FIN-CARD-R001`). Read/verify-only — this connector confirms identity, it does not itself perform account actions.

### CARD-CONN-003 — Card Management Connector
- **category:** case-management
- **module_path_pattern:** `src/*/connectors/card_management_connector.py`
- **allowed_host_patterns:** organization's internal card-management system API
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Handles both the block action (autonomous once identity is verified, per `CARD-CTL-004`) and the reissue actions. The reissue-to-new-address code path must be gated behind `guardrails/human_approval_required.py` (`CARD-CTL-005`) even though it routes through this same connector — being a governed connector does not exempt an action from the approval gate, same principle as `finance-aml/allowlist.md`.

### CARD-CONN-004 — Customer Master Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/customer_master_connector.py`
- **allowed_host_patterns:** organization's internal customer master / account system API
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Source of the address-on-file (used to determine whether a reissue request needs the step-up gate) and other `core/pii-taxonomy.md` categories. Every read must pass through `guardrails/pii_redaction.py` before appearing in output.

### CARD-CONN-005 — Notification Connector
- **category:** communication
- **module_path_pattern:** `src/*/connectors/notification_connector.py`
- **allowed_host_patterns:** organization's customer notification/messaging system
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Sends the block-confirmation message. Itself a controlled action — a confirmation sent to the wrong party (e.g. before/without identity verification, or to a stale contact method) would leak account-status information to someone who may not be the actual cardholder. Must run after `CARD-CTL-003` (identity verification) has passed.

## Explicitly Disallowed

Anywhere outside the five `module_path_pattern`s above, the following fail the build (`egress_gate.py`):
- Direct provider SDK usage (`import openai`, `import anthropic`, `import google.generativeai`, etc.)
- Raw DB drivers or connection strings
- Raw HTTP calls to any host not matching an `allowed_host_patterns` entry above
- Any code path that fulfills a reissue-to-new-address request through `card_management_connector.py` **without** first passing through `guardrails/human_approval_required.py` — this is checked structurally (the two reissue tools must be distinct functions/entry points, not a single function with an internal conditional an author could accidentally bypass).
