# Governance Schema (Core)

Domain-agnostic. Defines the "Scope & Bind" governance envelope every generated project must declare in its `governance/scope-bind.yaml` before any downstream pipeline step runs. A pack (e.g. `packs/finance-aml/`) may narrow these (e.g. force a minimum `risk_tier`) but never widen them.

**This file is the human/LLM-readable prose explanation. `governance-schema.json` alongside it is the machine-readable twin with identical IDs and field values — scripts (`kb_loader.py`, `validate_scope_bind.py`) load the `.json`, this `.md` is what a person or an LLM should read to understand *why*. If the two ever disagree, the `.json` is authoritative for validation behavior and this file should be corrected to match.**

## Entry Format

Each field below is defined with:
- `field_id` — stable identifier (`GOV-###`)
- `field_name` — the key used in `scope-bind.yaml`
- `type` — `enum` (closed list) or `open-taxonomy` (extensible, defined in a separate taxonomy file)
- `allowed_values` — for `enum` types
- `description` — what it constrains
- `consumed_by` — which pipeline steps/scripts must read and respect this value
- `validation_rule` — what a validator script checks
- `prompt_directive` — if non-empty, a natural-language instruction template that the prompt-generation step should inject into the generated agent's system prompt, so the deployed agent — not just the build pipeline — respects this constraint at runtime

---

## GOV-001 — data_classification

- **field_name:** `data_classification`
- **type:** enum
- **allowed_values:** `Public`, `Internal`, `Confidential`, `Restricted`
- **description:** The sensitivity tier of the data this agent will handle. Four-tier model, aligned with common ISO 27001 / NIST 800-53-adjacent classification practice (see `ignore_dir/research-sources.md` §B for sourcing).
- **consumed_by:** `scope-and-bind` (elicits value), `architecture-recommender` (may only propose connectors/storage rated for this tier or higher), `codegen-agent` (wires redaction/handling guardrails accordingly), `egress_gate.py` (cross-checks connector `approved_for_data_classification` in `allowlist.json`)
- **validation_rule:** value must be a member of `allowed_values`; a pack may set a floor (e.g. finance-aml requires at minimum `Confidential`) which validators enforce as `>= floor` on the ordinal `Public < Internal < Confidential < Restricted`.
- **prompt_directive:** "This system handles data classified as **{data_classification}**. Handle it accordingly: never output Restricted/Confidential data to a channel or party not explicitly authorized for that classification, and never lower the effective classification of data by omission or paraphrase that removes its sensitivity markers."

## GOV-002 — jurisdictions

- **field_name:** `jurisdictions`
- **type:** enum (multi-select)
- **allowed_values:** `US`, `EU`, `UK`
- **description:** The regulatory jurisdiction(s) this deployment operates under. Determines which jurisdiction-specific content within an active pack applies (e.g. a finance-aml pack may have US-specific BSA/FinCEN rules and separate EU/UK equivalents).
- **consumed_by:** `scope-and-bind`, `requirements-analyst` (jurisdiction-specific requirement prompts), pack content selection (regime-rules.md sections are tagged by jurisdiction where applicable)
- **validation_rule:** non-empty subset of `allowed_values`; at least one jurisdiction required.
- **prompt_directive:** "This deployment operates under the following jurisdiction(s): **{jurisdictions}**. Apply jurisdiction-specific rules and terminology accordingly (e.g. SAR filing under US BSA/FinCEN vs. SAR filing under EU AMLD5/UK MLR 2017 differ in specifics) — do not assume US rules apply to non-US jurisdictions."

## GOV-003 — approved_models

- **field_name:** `approved_models`
- **type:** enum (multi-select)
- **allowed_values:** `Claude`, `GPT-4o`, `Gemini`
- **description:** Which model provider(s) this deployment is authorized to call. Each approved model must map to a registered `model-gateway` connector in the active pack's `allowlist.json` — "approved" is not just a label, it must correspond to an actual governed connector, never a raw provider SDK call.
- **consumed_by:** `architecture-recommender`, `codegen-agent` (wires `connectors/model_gateway.py`), `egress_gate.py` (fails the build if any model identifier appears in code that isn't in this list, and fails if an approved model has no matching connector)
- **validation_rule:** non-empty subset of `allowed_values`; every entry must have a corresponding connector in `allowlist.json` with matching `category: model-gateway`.
- **prompt_directive:** (none — this is a build-time/infrastructure constraint, not something the deployed agent needs to reason about at runtime; the model it's running on is already fixed by the time it's deployed)

## GOV-004 — risk_tier

- **field_name:** `risk_tier`
- **type:** enum
- **allowed_values:** `Low`, `Medium`, `High`
- **description:** Overall risk classification of this agent's actions and blast radius. Drives which controls in `core/risk-tiers.md` and a pack's `controls-catalog.md` are mandatory vs. recommended. A pack may set a default/floor (finance-aml defaults to `High` unless explicitly justified otherwise).
- **consumed_by:** all pipeline steps from architecture design onward; `governance-validator` (checks mandatory controls for the declared tier are actually present in generated code)
- **validation_rule:** value must be a member of `allowed_values`; if a pack floor exists and the declared value is below it, validation fails unless an explicit, logged override justification is present.
- **prompt_directive:** "This agent operates under a **{risk_tier}** risk tier. {risk_tier_behavior}" where `{risk_tier_behavior}` is looked up from `core/risk-tiers.md` (e.g. for High: "Treat all non-read-only actions as requiring explicit human approval before execution, and cite the specific evidence backing any claim before presenting it for review.")

## GOV-005 — pii_data

- **field_name:** `pii_data`
- **type:** open-taxonomy (extensible list, not a closed enum)
- **allowed_values:** defined in `core/pii-taxonomy.md` (e.g. `customer_name`, `ssn`, `account_number`, `dob`, ...); a project may declare any subset that appears in the taxonomy, and may propose new taxonomy entries via the same file rather than freeform strings.
- **description:** Which categories of personally identifiable information this agent will encounter and must handle under the `pii_redaction` guardrail.
- **consumed_by:** `prompt-engineer` (wires categories into `guardrails/pii_redaction.py` config), `egress_gate.py` (secondary check for hardcoded values matching declared PII patterns appearing in logs/output paths that shouldn't have them)
- **validation_rule:** every declared value must exist in `pii-taxonomy.md`'s catalog.
- **prompt_directive:** "This agent will encounter the following categories of PII: **{pii_data}**. Never include these in full, unredacted form in any output unless the specific recipient and channel is explicitly authorized to receive unredacted PII for that category (see `guardrails/pii_redaction.py` for the enforced redaction rules) — treat this as a hard constraint, not a style preference."

---

## Notes for pack authors

A pack (`knowledge-base/packs/<pack-name>/pack-manifest.yaml`) may:
- set a `risk_tier` floor,
- set a `data_classification` floor,
- restrict `jurisdictions` to a subset relevant to the pack's regulatory domain,
- extend the `pii_data` taxonomy with domain-specific categories (referencing `pii-taxonomy.md`, not redefining it).

A pack may never remove or redefine a `GOV-###` field, add new top-level governance fields outside this schema, or weaken a validation rule defined here.
