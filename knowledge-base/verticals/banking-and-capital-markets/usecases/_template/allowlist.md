# Governed Connector Allowlist — REPLACE_ME (usecase_id)

Defines the only files in a generated project allowed to make outbound calls for this usecase. `scripts/egress_gate.py` uses this via `allowlist.json`.

**Placeholder disclaimer:** use illustrative host patterns, never real-looking endpoints.

## Connectors

### PREFIX-CONN-001 — Model Gateway
- **category:** model-gateway
- **module_path_pattern:** `src/*/connectors/model_gateway.py`
- **allowed_host_patterns:** REPLACE_ME
- **eligibility:** reference `../../model-catalog.json` for compliance-attribute filtering, same pattern as `card-blocking-servicing/allowlist.md`.

*(add one connector per external system category — data sources, case/record systems, communication channels, etc.)*

## Explicitly Disallowed

- Direct provider SDK usage, raw DB drivers/connection strings, raw HTTP to non-allowlisted hosts.
- REPLACE_ME — call out any action that should have NO connector at all, if applicable to this usecase's risk shape (the way `card-blocking-servicing` has no "reissue-to-new-address-without-approval" path).
