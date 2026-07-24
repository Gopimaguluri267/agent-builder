# Model Catalog — Banking & Capital Markets

> **Disclaimer, stronger than usual for this file specifically:** compliance certifications change over time and vary by product tier, region, and contract terms. The entries below are a **snapshot from research compiled 2026-07-24**, not a live feed. **Always verify current certification status directly with the vendor** before relying on any entry here — this is the source material's own caveat, not just ours. Treat every attribute below as "reportedly, as of last verification," never as a permanent fact.

This narrows `core/governance-schema.json`'s `GOV-003` (`approved_models`) for this vertical specifically: an entry being listed in `GOV-003.allowed_values` (Claude / GPT-4o / Gemini) means the *provider* is generically approved — this file adds the *compliance attributes* that determine whether a specific model is eligible for a specific project's declared `data_classification` and `risk_tier`.

**This file is the prose explanation. `model-catalog.json` alongside it is the machine-readable twin — a validator script should filter/cross-check `approved_models` selections against this file's compliance attributes for the project's declared `data_classification`. `.json` is authoritative on disagreement.**

## Entry Format

- `model_id` — stable identifier (`MODEL-###`)
- `provider`
- `model_family`
- `compliance_attributes` — `soc2_type2`, `iso_27001`, `iso_42001`, `fedramp_level` (`none`/`moderate`/`high`), `pci_dss_v4`, `data_residency_options`, `zero_data_retention_available`
- `minimum_eligible_data_classification` — the lowest `core/governance-schema.md` `data_classification` tier this model is reportedly suitable for, given its attributes (informational guidance, not a hard machine rule — a project should still verify against its own risk assessment)
- `last_verified`
- `notes`

## MODEL-001 — Claude (Anthropic)

- **compliance_attributes:** SOC 2 (reported), ISO/IEC 27001 (reported), ISO/IEC 42001 (reported); FedRAMP not confirmed in this research pass; PCI-DSS v4.0 not confirmed in this research pass; Zero-Data-Retention reportedly available on enterprise/API tiers.
- **minimum_eligible_data_classification:** Confidential (based on SOC 2 + ISO 27001 + ZDR availability) — **Restricted eligibility not confirmed**, verify FedRAMP/PCI-DSS status with Anthropic directly before using for Restricted-classification data.
- **last_verified:** 2026-07-24 (secondary sources, not vendor-confirmed)
- **notes:** Used as the model gateway target in this plugin's existing worked examples (e.g. the old `finance-aml`/`finance-card-servicing` reference packs in `ignore_dir/`).

## MODEL-002 — Gemini / Vertex AI (Google)

- **compliance_attributes:** SOC 1/2/3 (reported), ISO/IEC 42001 (reported), HITRUST (reported), FedRAMP High (reported), PCI-DSS v4.0 (reported); regional (EU) data-at-rest residency reportedly available.
- **minimum_eligible_data_classification:** Restricted (broadest reported coverage of any model in this catalog, including FedRAMP High and PCI-DSS v4.0) — still verify directly with Google before relying on this for a Restricted-classification production deployment.
- **last_verified:** 2026-07-24 (secondary sources, not vendor-confirmed)
- **notes:** The only entry in this catalog with a reported PCI-DSS v4.0 attestation specifically — relevant given this vertical's `BCM-TECH-001` PAN-masking requirement, though PCI-DSS attestation of the model provider does not by itself satisfy the *application's* own PAN-masking obligation.

## MODEL-003 — GPT (OpenAI)

- **compliance_attributes:** SOC 2 (reported), ISO/IEC 27001 (reported); ISO/IEC 42001 not confirmed in this research pass; regional (EU) data residency reportedly available; Zero-Data-Retention reportedly available on enterprise/API tiers.
- **minimum_eligible_data_classification:** Confidential (based on SOC 2 + ISO 27001 + ZDR availability) — **Restricted eligibility not confirmed**, verify FedRAMP/PCI-DSS/ISO 42001 status with OpenAI directly.
- **last_verified:** 2026-07-24 (secondary sources, not vendor-confirmed)
- **notes:** listed in `core/governance-schema.md`'s `GOV-003` as "GPT-4o" — current model naming should be re-verified with OpenAI at time of use, model names change faster than this catalog is likely to be updated.

## How to use this file

1. A project declares `data_classification` and `approved_models` in `scope-bind.yaml` (Scope & Bind step).
2. Cross-check: does every declared `approved_models` entry's `minimum_eligible_data_classification` here meet or exceed the project's declared `data_classification`? If not, either the model choice or the data classification needs to change — this is a `block` or `hold` verdict (see `commands/forge.md`'s verdict vocabulary), not something to silently wave through.
3. Re-verify this file's contents periodically — it is a research snapshot, not a live compliance feed, and the disclaimer at the top of this file applies with full force.

## Sources

- [Requesty — Security & Compliance Checklist: SOC 2, HIPAA, GDPR for LLM Gateways](https://www.requesty.ai/blog/security-compliance-checklist-soc-2-hipaa-gdpr-for-llm-gateways-1751655071)
- [Layer3labs — AI Model Compliance Comparison (2026)](https://www.layer3labs.io/guides/ai-model-compliance-comparison)
- [Vanta — FedRAMP and SOC 2: An in-depth comparison](https://www.vanta.com/collection/fedramp/fedramp-and-soc-2)

Full research trail in the local, gitignored `ignore_dir/research-sources.md` §G.
