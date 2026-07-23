# Governed Connector Allowlist — finance-aml

Defines the only files in a generated project allowed to make outbound calls (to a model provider, database, or external API). `scripts/egress_gate.py` uses this list (via `allowlist.json`) to fail the build if any raw SDK/DB/HTTP call appears outside a registered connector module.

**Note on terminology** (see `ignore_dir/research-sources.md` for the full finding): "governed connector" means something different at two layers of this system. This file governs the **generated agent's own source code** — which of *its* files may reach the outside world. It is not about which MCP servers Claude Code itself may call while *building* the agent (that's a separate, real pattern used by `vyayasan/kyc-analyst`'s `CONNECTORS.md`, gated by human-in-the-loop consent at the point of use — a good precedent, but for a different layer than this file).

**This file is the prose explanation. `allowlist.json` alongside it is the machine-readable twin — `egress_gate.py` loads the `.json` for its AST/regex matching. `.json` is authoritative on disagreement.**

## Entry Format

- `connector_id` — stable identifier (`AML-CONN-###`)
- `connector_name`
- `category` — `model-gateway` / `data-source` / `case-management` / `communication`
- `module_path_pattern` — the only file(s), by path glob, allowed to contain the raw call this connector wraps
- `allowed_host_patterns` — placeholder regexes for the external host(s) this connector may reach (illustrative — see note below)
- `approved_for_jurisdictions`
- `approved_for_data_classification`
- `notes`

**Placeholder disclaimer:** the `allowed_host_patterns` values below are illustrative placeholders (e.g. `api.examplebank.internal`), not real endpoints. Replace them with your organization's actual governed connector registry before any production use — this file describes the *shape* of a governed connector registry, not a specific vendor integration.

## Connectors

### AML-CONN-001 — Model Gateway
- **category:** model-gateway
- **module_path_pattern:** `src/*/connectors/model_gateway.py`
- **allowed_host_patterns:** organization's internal LLM gateway/proxy endpoint (never a direct provider API host)
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** The only file permitted to import/call a model-provider SDK. Must correspond to an entry in `core/governance-schema.json`'s `approved_models` for the active project.

### AML-CONN-002 — Transaction Monitoring Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/transaction_monitoring_connector.py`
- **allowed_host_patterns:** organization's internal transaction-monitoring system API
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Read access to flagged transactions and transaction history. Read-only by design — no write/mutate operations should exist on this connector for this pack.

### AML-CONN-003 — Case Management Connector
- **category:** case-management
- **module_path_pattern:** `src/*/connectors/case_management_connector.py`
- **allowed_host_patterns:** organization's internal case-management system API
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Read/append access to case records (notes, draft narratives). State-changing operations (close case, mark filed) must be gated behind `guardrails/human_approval_required.py` even though they route through this connector — being a governed connector does not exempt an action from the approval gate.

### AML-CONN-004 — Sanctions/Watchlist Screening Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/sanctions_screening_connector.py`
- **allowed_host_patterns:** organization's sanctions-screening provider, or a direct integration with public lists
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Illustrative example of real public data sources such a connector might integrate with, per the `kyc-analyst` precedent (see `ignore_dir/research-sources.md` §C.1): OFAC SDN List, UN Consolidated Sanctions List, EU Sanctions List, UK HMT Sanctions List. Named here as a reference point for what "sanctions screening" typically covers, not as a specific integration this plugin ships.

### AML-CONN-005 — KYC/Customer Master Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/kyc_connector.py`
- **allowed_host_patterns:** organization's internal customer master / KYC system API
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Source of most `core/pii-taxonomy.md` categories (customer_name, ssn, dob, address, etc.) — every read from this connector must pass through `guardrails/pii_redaction.py` before appearing in any output.

### AML-CONN-006 — Adverse Media Connector
- **category:** data-source
- **module_path_pattern:** `src/*/connectors/adverse_media_connector.py`
- **allowed_host_patterns:** organization's adverse-media screening provider
- **approved_for_jurisdictions:** US
- **approved_for_data_classification:** Confidential
- **notes:** Highest prompt-injection exposure of any connector — retrieved article text is external, potentially attacker-influenceable content. Every read from this connector must pass through `guardrails/prompt_injection_detection.py` (`AML-CTL-002`) before being incorporated into reasoning.

## Explicitly Disallowed

Anywhere outside the six `module_path_pattern`s above, the following fail the build (`egress_gate.py`):
- Direct provider SDK usage (`import openai`, `import anthropic`, `import google.generativeai`, etc.)
- Raw DB drivers or connection strings (`psycopg2.connect(`, `pymongo.MongoClient(`, literal `postgres://`/`mongodb://` strings)
- Raw HTTP calls (`requests.get/post(`, `httpx.Client(`) to any host not matching an `allowed_host_patterns` entry above
- Any tool/function whose name or docstring implies a SAR-filing or case-closure *action* (as opposed to a draft/recommendation) — there is intentionally no connector in this list for "file a SAR" or "close a case," because no such autonomous action is in scope for this pack (see `FIN-AML-R001`, `FIN-AML-R002`).
