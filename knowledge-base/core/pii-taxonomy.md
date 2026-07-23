# PII Taxonomy (Core)

Domain-agnostic catalog of PII categories a generated project may declare under `governance/scope-bind.yaml`'s `pii_data` field (`GOV-005`). Open taxonomy — a pack may propose new categories relevant to its domain, but must add them here (or in a pack-local extension referenced from here), never invent freeform strings that bypass this catalog.

Impact levels follow NIST SP 800-122's confidentiality-impact framing (Low / Moderate / High — potential harm to the individual or organization if the category is inappropriately accessed, used, or disclosed). See `ignore_dir/research-sources.md` §B for sourcing.

**This file is the prose explanation. `pii-taxonomy.json` alongside it is the machine-readable twin — `validate_scope_bind.py` loads the `.json` to check that every `pii_data` entry a project declares actually exists in the catalog. `.json` is authoritative on disagreement.**

## Entry Format

- `category_id` — stable identifier (`PII-###`)
- `category_name` — the string used in `scope-bind.yaml`'s `pii_data` list
- `description` — what it covers
- `nist_impact_level` — `Low`, `Moderate`, or `High`
- `redaction_treatment` — how `guardrails/pii_redaction.py` should handle it by default (a project may tighten, never loosen, this default)
- `typical_sources` — where this data tends to originate in a financial-services context

## Categories

| ID | category_name | description | nist_impact_level | redaction_treatment | typical_sources |
|---|---|---|---|---|---|
| PII-001 | `customer_name` | Full legal name of a customer | Moderate | Show in full only to an authorized analyst reviewing their own assigned case; mask in logs/exports (`J*** D**`) | KYC/customer master |
| PII-002 | `ssn` | US Social Security Number / equivalent national ID | High | Never display in full anywhere; mask to last 4 digits max, even for authorized analysts | KYC/customer master |
| PII-003 | `account_number` | Bank/financial account identifier | High | Mask to last 4 digits in all outputs, including analyst-facing ones, unless the specific tool call requires the full number to query a governed connector | Transaction/core banking system |
| PII-004 | `dob` | Date of birth | Moderate | Full display permitted to authorized analyst; redact in any output leaving the case-management system | KYC/customer master |
| PII-005 | `address` | Residential or business address | Moderate | Full display permitted to authorized analyst; redact in shared/exported reports unless address itself is the subject of the finding | KYC/customer master |
| PII-006 | `phone_number` | Contact phone number | Low | Mask middle digits in exports; full display permitted in-tool | KYC/customer master |
| PII-007 | `email` | Contact email address | Low | Mask local-part in exports (`j***@example.com`); full display permitted in-tool | KYC/customer master |
| PII-008 | `government_id_number` | Passport, driver's license, or other government-issued ID number (non-SSN) | High | Never display in full; mask to last 4 characters | KYC/customer master, onboarding documents |
| PII-009 | `tax_id` | Employer Identification Number / Individual Taxpayer ID Number | High | Never display in full; mask to last 4 digits | KYC/customer master (business entities) |
| PII-010 | `employer` | Stated employer / occupation | Low | Full display permitted; not typically redaction-sensitive on its own | KYC/customer master |
| PII-011 | `income` | Stated or inferred income/net worth | Moderate | Full display permitted to authorized analyst; redact in exports outside the case file | KYC/customer master |
| PII-012 | `counterparty_name` | Name of a transaction counterparty (may or may not be the bank's own customer) | Moderate | Same treatment as `customer_name`; counterparties are not automatically less sensitive | Transaction records, adverse media results |
| PII-013 | `ip_address` | IP address associated with a digital transaction or login | Low | Full display permitted in-tool for investigation; mask in long-term exports/reports | Digital banking / login telemetry |
| PII-014 | `device_id` | Device fingerprint/identifier associated with a transaction | Low | Same treatment as `ip_address` | Digital banking / login telemetry |

## Notes

- A category's `redaction_treatment` is a **default floor**, not a ceiling — a specific pack or project may require stricter handling (e.g. a pack could mandate `ssn` never appear even in-tool, only as a boolean "SSN on file: yes/no"), but may never loosen the default above.
- `nist_impact_level` is informational context for humans/LLM agents reasoning about a category's sensitivity; it is not itself machine-enforced — `redaction_treatment` is the field that actually drives guardrail behavior.
