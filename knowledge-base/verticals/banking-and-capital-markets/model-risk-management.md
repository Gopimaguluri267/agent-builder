# Model Risk Management — Banking & Capital Markets

> **Disclaimer:** Compressed, illustrative summary compiled 2026-07-24. **Not legal advice.** Verify against current agency guidance before production use.

**Time-sensitive note:** if you have encountered "SR 11-7" or "OCC 2011-12" referenced elsewhere as the current model risk management standard for banks — **that guidance was superseded on April 17, 2026.** Do not cite the 2011 framework as current.

## The current guidance

On **April 17, 2026**, the OCC, Federal Reserve, and FDIC jointly issued **Revised Guidance on Model Risk Management**, replacing:
- the 2011 Original Guidance (Federal Reserve **SR 11-7**, OCC **2011-12**), and
- the 2021 Interagency Statement on Model Risk Management.

Official citations for the current guidance:
- Federal Reserve: **[SR 26-2](https://www.federalreserve.gov/supervisionreg/srletters/SR2602.htm)**, "Revised Guidance on Model Risk Management"
- FDIC: **FIL-15-2026**, "Agencies Revise the Interagency Model Risk Management Guidance"
- OCC: **[Bulletin 2026-13](https://www.occ.gov/news-issuances/bulletins/2026/bulletin-2026-13.html)**, "Model Risk Management: Revised Guidance"

## Scope and applicability

Applies to national banks, state banks, bank holding companies, and other institutions for which the Federal Reserve, OCC, or FDIC is the primary supervisor. The revised guidance is explicitly **risk-scaled**: tailored to a bank's size, complexity, and model risk profile rather than a flat, one-size-fits-all requirement. It's described as most relevant to banking organizations with over $30 billion in total assets, but designed to scale based on actual model risk exposure rather than asset size alone — a smaller institution with significant model risk exposure (e.g. heavy reliance on an LLM-based agent for customer-facing decisions) is not automatically exempt.

## What "model" means here, and why it includes agentic systems

The guidance's definition of a "model" (carried forward conceptually from the 2011 Original Guidance) covers any quantitative method, system, or approach that processes input data into quantitative or qualitative estimates/outputs used to inform a decision. An LLM-based agent that assesses risk, recommends an action, or makes a determination affecting a customer falls within this definition — it is not exempt from model risk management just because it's generative AI rather than a traditional statistical model.

## Core expectations, and how this plugin operationalizes them

| MRM expectation | This plugin's mechanic |
|---|---|
| Model validation before deployment | `scripts/validate_scope_bind.py` and (once built) an equivalent pre-deployment gate for generated code |
| Independent review | `agents/governance-validator.md`'s cross-artifact check — distinct from the agent/team that built the system |
| Documented model risk classification | `core/risk-tiers.md`'s Low/Medium/High tiers, declared in `scope-bind.yaml` |
| Ongoing monitoring | `core/approval-policy.md`'s audit-log schema (actor/timestamp/case_id/evidence_hash) — provides the evidence trail monitoring would review |
| Governance tailored to risk exposure, not flat | `core/governance-schema.json`'s per-vertical `risk_tier` floor mechanism — a vertical can require a higher floor without forcing every use case in every vertical to the same bar |

This table is the vertical-specific regulatory *justification* for mechanics that already exist in `core/` — it doesn't introduce new mechanics, it explains why the existing ones are shaped the way they are for a banking context specifically.

## Sources

- [Federal Reserve — SR 26-2, Revised Guidance on Model Risk Management](https://www.federalreserve.gov/supervisionreg/srletters/SR2602.htm)
- [OCC — Bulletin 2026-13, Model Risk Management: Revised Guidance](https://www.occ.gov/news-issuances/bulletins/2026/bulletin-2026-13.html)
- [FDIC — Agencies Revise the Interagency Model Risk Management Guidance](https://www.fdic.gov/news/financial-institution-letters/2026/agencies-revise-interagency-model-risk-management-guidance)
- [Sullivan & Cromwell — Federal Banking Agencies Issue Revised Guidance on Model Risk Management](https://www.sullcrom.com/insights/memo/2026/April/OCC-Fed-FDIC-Issue-Revised-Guidance-Model-Risk-Management)
- [Davis Polk — Visual memo: Key changes under the federal banking agencies' revised model risk management guidance](https://www.davispolk.com/insights/client-update/visual-memo-key-changes-under-federal-banking-agencies-revised-model-risk)

Full research trail in the local, gitignored `ignore_dir/research-sources.md` §F.
