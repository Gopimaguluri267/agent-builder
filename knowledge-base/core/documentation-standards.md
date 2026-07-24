# Documentation Standards (Core)

Domain-agnostic. Defines the standard structure for the three formal documents `skills/requirements-doc-writer` generates — a Business Requirements Document (BRD), Product Requirements Document (PRD), and Functional Requirements Document (FRD) — plus the shared document-control block every one of them must carry. The section lists here are confirmed against real business-analysis/product-management practice (see `ignore_dir/research-sources.md` §H), not invented for this plugin.

**MD only — no JSON twin.** These are structural/content requirements for an LLM agent to follow when writing a document, not something a script parses field-by-field — `skills/requirements-doc-writer` reads this file directly.

## Entry Format

- `doc_id` — stable identifier (`DOC-STD-###`)
- `document_type`
- `grounding` — what practice/standard this structure is confirmed against
- `sections` — the standard section list
- `source`

---

## DOC-STD-001 — Business Requirements Document (BRD)

- **grounding:** general business-analysis practice; BABOK®/IIBA is the globally recognized standard for the discipline behind requirements elicitation and verification, though it does not prescribe one single fixed BRD template — the section list below is a confirmed real-world convention, not a literal BABOK template.
- **sections:**
  1. **Executive Summary** — one-paragraph framing of what's being built and why, written for a reader who will only read this section.
  2. **Objective** — the specific business objective this agent is meant to achieve.
  3. **Project Scope** — what's in scope; cross-reference the usecase's `usecase-manifest.yaml` description.
  4. **Assumptions** — what's being taken as given (e.g. an existing identity-verification system is already in place and this agent only calls it).
  5. **Constraints** — hard limits, including the governance constraints already declared in `scope-bind.yaml` (risk tier, data classification, approved models) — restate them here in business language, don't just link out.
  6. **Risks** — business-level risks, distinct from the technical `controls-catalog.json` risk mapping (though they should be consistent with it).
  7. **Success Criteria** — how the business will know this succeeded.
  8. **KPIs** — specific, measurable indicators tied to the success criteria.
- **source:** [IIBA — BABOK Guide](https://www.iiba.org/knowledgehub/business-analysis-body-of-knowledge-babok-guide/)

## DOC-STD-002 — Product Requirements Document (PRD)

- **grounding:** standard product-management practice, confirmed across multiple independent sources (Aha.io, Product School, AltexSoft).
- **sections:**
  1. **Problem Statement** — the specific problem this agent solves, from the user's perspective.
  2. **Goals** — what the product (the agent) is meant to accomplish.
  3. **Success Metrics** — quantifiable measures, may overlap with the BRD's KPIs but framed at the product level.
  4. **User Personas** — who uses this agent, in narrative form (e.g. "a verified retail banking customer reporting a lost card via the mobile app").
  5. **Target Users** — the concrete population, distinct from the AML usecase's analyst-facing framing — for `card-blocking-servicing` this is the cardholder directly.
  6. **Feature Requirements** — the specific capabilities, each one cross-referenced to the `usecase-rules.md` requirement ID(s) it implements or is constrained by (e.g. "Block card [implements CARD-UC-R002]").
  7. **Out-of-Scope Pointers** — explicitly what this agent will NOT do, referencing the usecase's hard rules (e.g. "will not reissue to a new address without a step-up gate — see CARD-UC-R005").
  8. **Dependencies** — what this agent depends on (connectors, upstream systems, the vertical's model catalog eligibility).
  9. **Open Questions** — anything still unresolved after intake + validation; do not silently drop these, they must appear in the document for the reader to see.
- **source:** [Aha.io — PRD Templates: What To Include](https://www.aha.io/roadmapping/guide/requirements-management/what-is-a-good-product-requirements-document-template), [Product School — The Only PRD Template You Need](https://productschool.com/blog/product-strategy/product-template-requirements-document-prd)

## DOC-STD-003 — Functional Requirements Document (FRD)

- **grounding:** standard business-analysis/systems-analysis practice — "a formal statement of an application's functional requirements that serves the same purpose as a contract" between the builder and the client.
- **sections:**
  1. **Use Cases** — narrative scenarios per user role (e.g. "Verified cardholder reports lost card" / "Cardholder requests replacement to address on file" / "Cardholder requests replacement to a new address").
  2. **Features** — the concrete functional capabilities, more granular than the PRD's feature list.
  3. **Validation Rules** — the business rules the system must enforce; this is where `controls-catalog.json` entries and `usecase-rules.md` requirements become concrete, checkable statements (e.g. "the system must not display more than the first 6 and last 4 digits of a PAN — CARD-UC-R004 / CARD-CTL-001").
  4. **Workflows** — step-by-step flows, including where a human-approval gate interrupts the flow (e.g. the reissue-to-new-address flow explicitly shows the step-up-verification interrupt point).
  5. **Data Requirements** — what data is read/written, through which connector, referencing `allowlist.json` connector IDs (e.g. "customer address — read via CARD-CONN-004").
- **source:** [Tutorialspoint — Functional Requirements Document](https://www.tutorialspoint.com/business_analysis/business_analysis_functional_requirements_document.htm), [Savio — FRD Template & Examples](https://savioglobal.com/blog/business-analysis/functional-requirements-document-frd-template-examples/)

## DOC-STD-004 — Shared Document-Control Block

- **grounding:** financial-services documentation control practice — regulated-industry documents are expected to carry explicit regulatory references, named approval authority, version history, and captured sign-off, distinct from the document's substantive content.
- **required fields**, present at the top of every generated document (BRD/PRD/FRD alike):
  - `document_type`, `version` (starts at `0.1`, bumped on any post-signoff revision)
  - `project_id`, `run_id` (the `.agentbuilder` run this document belongs to)
  - `vertical`, `usecase` (from `scope-bind.yaml`)
  - `prepared_by` — literally "Agent Builder plugin" plus the session's operator context, not a fabricated human author
  - `date_prepared`
  - `status` — `draft` / `pending_signoff` / `signed`
  - `regulatory_references` — the specific `BCM-REG-###`/`BCM-TECH-###`/usecase-rule IDs actually relevant to this document's content, pulled from the KB, never invented or left generic
- **required fields, Sign-Off section** (appears at the bottom, see `skills/document-rendering/SKILL.md` for the concrete HTML rendering):
  - `approver_name` — filled in only once actually provided by a person in the session, never pre-filled or guessed
  - `approver_role`
  - `affirmation_statement` — e.g. "I have reviewed and approve this [document type] as documented above."
  - `signed_at` — timestamp, captured at the moment of affirmative approval, not backdated
  - `evidence_hash` — hash of the document's content at the moment of signing, per `core/approval-policy.md`'s schema
- **source:** [Docsie — Financial Services Documentation Templates 2026](https://www.docsie.io/blog/articles/financial-services-documentation-templates-2026/), [ProjectManagement.com — BRD Sign-off Template](https://www.projectmanagement.com/deliverables/287499/business-requirements-document---customer-client-sign-off-template)

## DOC-STD-005 — Content Depth Standard

- **grounding:** internal quality bar, added after a review of an early rendering test found the mechanics (structure, styling, sign-off flow) were sound but the placeholder content used to test them was exactly the kind of one-liner-per-section output this standard exists to prevent. Not an external citation — a self-imposed bar because none of DOC-STD-001/002/003 said how much content each section actually needs, and left unspecified, an LLM filling nine sections quickly will default to terse.
- **Sections that are brief by genre convention — do not pad these:**
  - BRD **Executive Summary** — 3–5 sentences. It exists so a reader who reads nothing else still understands the document; padding it defeats the point.
  - BRD **Objective** — 1–2 sentences, or a short paragraph if the objective genuinely has multiple parts.
  - PRD **Problem Statement** — a short paragraph, framed from the user's perspective, not a restated title.
- **Sections that require real substance — a single sentence here is a defect, not a valid output:**
  - BRD **Project Scope, Assumptions, Constraints, Risks, Success Criteria, KPIs** — each is a list; every list item must be a complete sentence that states not just *what* but *why it matters or what happens if it's wrong*. "Risk: fraud" is not acceptable. "Risk: an attacker who has partially compromised a customer's identity could use a replacement-card request to redirect a new card to an address they control — mitigated by CARD-CTL-005's mandatory step-up gate on any reissue-to-new-address request" is the bar.
  - PRD **User Personas** — each persona needs a real narrative (2–3 sentences): who they are, what situation brings them to this agent, what they need from the interaction — not just a job title.
  - PRD **Feature Requirements** — each feature needs a name plus a substantive description (2–4 sentences: what it does, why it's needed, any conditions/exceptions) and its `usecase-rules.md` cross-reference. A bare feature title with no description is not acceptable. See the worked example below.
  - PRD **Out-of-Scope, Dependencies, Open Questions** — same complete-sentence bar as the BRD lists above.
  - FRD **Use Cases** — each one is a short narrative or numbered step sequence (not a one-line label) describing an actual scenario from a specific user role's perspective, including at least the trigger, the main flow, and any point where it can diverge (e.g. verification fails, a gate is hit).
  - FRD **Workflows** — numbered steps, each step a full sentence describing what happens and who/what performs it; any human-approval-gate interrupt point must be visually and textually called out, not buried in a step like any other.
  - FRD **Validation Rules, Data Requirements** — table rows are acceptable *if* each row's rule/element description is specific and complete, not a fragment. A `Validation Rules` row reading only "PAN masking" with no rule text is not acceptable; it must state the actual rule ("Display shows no more than the first 6 and last 4 digits of the PAN in any output") alongside its reference ID.
- **Anti-patterns to actively avoid:**
  - Any list where every item is under ~8 words.
  - Generic filler that could apply to any project ("various risks exist," "several features are planned," "data will be handled appropriately").
  - A Feature Requirements or Use Cases section with only titles and no elaboration.
  - Treating the KB cross-reference ID as a substitute for actually explaining the rule in the document's own words — the reference supports the sentence, it doesn't replace it.

### Worked example — what "good" looks like for one PRD Feature Requirement entry

> **Feature: Immediate Card Block** *(implements `CARD-UC-R002`)*
> Once the customer's identity has been verified, the agent blocks the reported card immediately, without requiring a separate approval step. This is a deliberate design choice rather than a limitation: blocking is a protective, reversible action, and introducing an approval delay here would leave a customer exposed to further fraud while waiting for a human to respond. The block action is logged with the exact report timestamp, since that timestamp determines the customer's liability protection under `BCM-REG-006` (TILA §1643/FCBA).

That's the depth bar — four sentences carrying real information (what it does, why it's designed that way, what it's connected to, what the tradeoff was), not a bullet reading "Block card immediately."

---

## Relationship to other core files

- The **mechanics** of a sign-off (no silent proceed, evidence-before-consent, logged with actor/timestamp/case_id/evidence_hash) are already defined in `core/approval-policy.md` — `DOC-STD-004`'s Sign-Off section is that mechanism applied specifically to a generated document rather than a runtime agent action.
- Cross-referencing KB IDs into document content (rather than restating regulatory text) follows the same discipline established at the KB layer itself — a usecase's `usecase-rules.md` cites vertical IDs instead of restating them; these documents should cite usecase/vertical IDs instead of restating them too.

## Sources

- [IIBA — BABOK Guide](https://www.iiba.org/knowledgehub/business-analysis-body-of-knowledge-babok-guide/)
- [Aha.io — PRD Templates: What To Include for Success](https://www.aha.io/roadmapping/guide/requirements-management/what-is-a-good-product-requirements-document-template)
- [Product School — The Only PRD Template You Need](https://productschool.com/blog/product-strategy/product-template-requirements-document-prd)
- [AltexSoft — Product Requirements Document: PRD Templates and Examples](https://www.altexsoft.com/blog/product-requirements-document/)
- [Tutorialspoint — Functional Requirements Document](https://www.tutorialspoint.com/business_analysis/business_analysis_functional_requirements_document.htm)
- [Savio — Functional Requirements Document FRD – Template & Examples](https://savioglobal.com/blog/business-analysis/functional-requirements-document-frd-template-examples/)
- [Docsie — Financial Services Documentation Templates 2026](https://www.docsie.io/blog/articles/financial-services-documentation-templates-2026/)
- [ProjectManagement.com — BRD Customer/Client Sign-off Template](https://www.projectmanagement.com/deliverables/287499/business-requirements-document---customer-client-sign-off-template)

Full research trail in the local, gitignored `ignore_dir/research-sources.md` §H.
