# Governed Connector Allowlist — REPLACE_ME (pack_id)

Defines the only files in a generated project allowed to make outbound calls (to a model provider, database, or external API) for this pack's domain. `scripts/egress_gate.py` uses this list (via `allowlist.json`) to fail the build if any raw SDK/DB/HTTP call appears outside a registered connector module.

**Reminder on terminology:** this file governs the *generated agent's own source code* — which of its files may reach the outside world. It is a different concept from which MCP servers Claude Code itself may call while building the agent (see `finance-aml/allowlist.md`'s note on this distinction if unclear).

**This file is the prose explanation. `allowlist.json` alongside it is the machine-readable twin — `egress_gate.py` loads the `.json`. `.json` is authoritative on disagreement.**

**Placeholder disclaimer:** use illustrative host patterns (e.g. `*.your-system.internal.example`), never real-looking endpoints, and say so explicitly — this file describes the *shape* of a governed connector registry for this domain, not a specific vendor integration.

## Entry Format

- `connector_id` — stable identifier (`<PACK-PREFIX>-CONN-###`)
- `connector_name`
- `category` — `model-gateway` / `data-source` / `case-management` / `communication` (add a new category here if your domain needs one not listed, and note it explicitly)
- `module_path_pattern` — the only file(s), by path glob, allowed to contain the raw call this connector wraps
- `allowed_host_patterns` — placeholder regexes for the external host(s) this connector may reach
- `approved_for_jurisdictions`
- `approved_for_data_classification`
- `notes`

## Connectors

### <PREFIX>-CONN-001 — Model Gateway
- **category:** model-gateway
- **module_path_pattern:** `src/*/connectors/model_gateway.py`
- **allowed_host_patterns:** REPLACE_ME
- **approved_for_jurisdictions:** REPLACE_ME
- **approved_for_data_classification:** REPLACE_ME
- **notes:** Every pack needs this one — the only file permitted to import/call a model-provider SDK.

*(add one connector per external system category this pack's agent needs — data sources, case/record systems, screening services, communication channels, etc.)*

## Explicitly Disallowed

Anywhere outside the `module_path_pattern`s above, the following fail the build (`egress_gate.py`):
- Direct provider SDK usage (`import openai`, `import anthropic`, `import google.generativeai`, etc.)
- Raw DB drivers or connection strings
- Raw HTTP calls to any host not matching an `allowed_host_patterns` entry above
- REPLACE_ME — call out any domain-specific action that should have NO connector at all, the way `finance-aml/allowlist.md` explicitly has no "file a SAR" or "close a case" connector because that action is out of scope for an autonomous agent.
