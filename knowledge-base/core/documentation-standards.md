# Documentation Standards (Core)

Domain-agnostic. Defines the standard structure for the three formal documents `skills/requirements-doc-writer` generates — a Business Requirements Document (BRD), Product Requirements Document (PRD), and Functional Requirements Document (FRD) — plus the shared document-control block every one of them must carry. The section lists here are confirmed against real business-analysis/product-management practice (see `ignore_dir/research-sources.md` §H), not invented for this plugin. A later pass added several sections (marked below) based on user-provided reference material rather than fresh web research — see the note at the end of `DOC-STD-001`/`003`/`004` for exactly which, and the "Sources" section for where they came from.

**MD only — no JSON twin.** These are structural/content requirements for an LLM agent to follow when writing a document, not something a script parses field-by-field — `skills/requirements-doc-writer` reads this file directly.

## Entry Format

- `doc_id` — stable identifier (`DOC-STD-###`)
- `document_type`
- `grounding` — what practice/standard this structure is confirmed against
- `sections` — the standard section list
- `source`

---

## DOC-STD-001 — Business Requirements Document (BRD)

- **grounding:** general business-analysis practice; BABOK®/IIBA is the globally recognized standard for the discipline behind requirements elicitation and verification, though it does not prescribe one single fixed BRD template — the section list below is a confirmed real-world convention, not a literal BABOK template. Sections marked **(user-sourced)** were added from reviewed reference material, not independently researched.
- **sections:**
  1. **Executive Summary** — one-paragraph framing of what's being built and why, written for a reader who will only read this section.
  2. **Business Objectives** *(user-sourced)* — SMART objectives (Specific, Measurable, Achievable, Relevant, Time-bound), each with its own ID, as a table: `Objective ID | Objective | Measure | Target | Timeline`. Avoid vague objectives ("improve productivity"); require a measurable statement ("reduce manual requirement intake effort by 40% within the first release cycle"). This replaces a single-paragraph "Objective" — every objective is individually traceable (see the FRD's Traceability Matrix, `DOC-STD-003`).
  3. **Project Scope** — what's in scope; cross-reference the usecase's `usecase-manifest.yaml` description.
  4. **Stakeholders** *(user-sourced)* — table: `Name/Role | Department/Group | Interest | Influence (High/Medium/Low) | Responsibility (RACI role)`. Include, where applicable: executive sponsor, product owner, business owner, technical lead/architect, end users by persona, compliance/security representative, finance/budget approver. This is who the multi-role Sign-Off in `DOC-STD-004` draws its approver list from — keep the two consistent.
  5. **Current State Analysis** *(user-sourced)* — the "before" picture: current process/system description, pain points, bottlenecks, quantified impact where available (cost, time, error rate, cycle time). Without this, Business Objectives and Success Criteria have nothing concrete to measure improvement against.
  6. **Assumptions** — what's being taken as given (e.g. an existing identity-verification system is already in place and this agent only calls it).
  7. **Constraints** — hard limits, including the governance constraints already declared in `scope-bind.yaml` (risk tier, data classification, approved models) — restate them here in business language, don't just link out.
  8. **Risks** — business-level risks, distinct from the technical `controls-catalog.json` risk mapping (though they should be consistent with it).
  9. **Success Criteria** — how the business will know this succeeded.
  10. **KPIs** — specific, measurable indicators tied to the success criteria.
- **source:** [IIBA — BABOK Guide](https://www.iiba.org/knowledgehub/business-analysis-body-of-knowledge-babok-guide/); user-sourced sections per `ignore_dir/BRD_SKILL_cleaned.md`.

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

- **grounding:** standard business-analysis/systems-analysis practice — "a formal statement of an application's functional requirements that serves the same purpose as a contract" between the builder and the client. Sections marked **(user-sourced)** were added from reviewed reference material. Interface Specifications, State Management, and API/circuit-breaker-level detail from that same reference material were deliberately **not** incorporated here — that level of detail belongs to the (not-yet-built) architecture/codegen pipeline step, which hasn't chosen a framework or connector shape yet; specifying API contracts in the FRD would prescribe implementation ahead of that decision.
- **sections:**
  1. **Use Cases** — narrative scenarios per user role. *(user-sourced enhancement)* Each use case must include, at minimum: the trigger, the main flow (numbered steps), at least one **Alternative Flow** (a variation that still succeeds, e.g. "reissue to address on file" as the alternative to a new-address request), and at least one **Exception Flow** (something goes wrong — verification fails, a gate is hit — and how it's handled). A use case with only a main flow is incomplete.
  2. **Features** — the concrete functional capabilities, more granular than the PRD's feature list.
  3. **Business Rules** *(user-sourced, new section — previously folded into Validation Rules)* — the rules that govern system behavior, distinct from input/output validation: computation/derivation rules, constraint rules (limits/thresholds), authorization rules (who can do what), sequencing rules, timing rules. Table: `Rule ID | Description | Condition | Action | Exception | Source (regulatory/business) | Applied In (which features/use cases)`. Every business rule with a regulatory basis must cite the actual `BCM-REG-###`/usecase-rule ID in `Source` — never a vague "policy."
  4. **Validation Rules** — the concrete input/output validation the system must enforce; this is where `controls-catalog.json` entries and `usecase-rules.md` requirements become checkable statements (e.g. "the system must not display more than the first 6 and last 4 digits of a PAN — CARD-UC-R004 / CARD-CTL-001"). Distinct from Business Rules above: validation rules check data at a point (format, range, required); business rules govern behavior/decisions.
  5. **Workflows** — step-by-step flows, including where a human-approval gate interrupts the flow (e.g. the reissue-to-new-address flow explicitly shows the step-up-verification interrupt point).
  6. **Data Requirements** — what data is read/written, through which connector, referencing `allowlist.json` connector IDs (e.g. "customer address — read via CARD-CONN-004").
  7. **Security Requirements** *(user-sourced)* — surfaces, as its own reader-facing section, what's otherwise implicit in `controls-catalog.json`/`core/pii-taxonomy.md`: authentication expectations (referencing the usecase's identity-verification control), authorization (who/what can invoke which connector), data protection (classification → redaction treatment, from `pii-taxonomy.json`), and audit trail (what's logged, referencing `core/approval-policy.md`'s schema). This section doesn't invent new requirements — it makes existing KB-derived controls visible to a reader who won't otherwise open `controls-catalog.json`.
  8. **Test Scenarios** *(user-sourced)* — functional, edge-case, and security test scenarios derived from the use cases and validation/business rules above, each traced to what it tests. Table: `ID | Scenario | Steps | Expected Result | Traces To (use case / requirement / control ID)`. This is groundwork for the (not-yet-built) codegen step's `tests/golden_cases/` — write these as if they'll eventually become real test fixtures, not as decoration.
  9. **Traceability Matrix** *(user-sourced)* — table tying every BRD Business Objective through to a PRD feature, an FRD use case, the requirements/business rules/validation rules that implement it, and the test scenario(s) that verify it. `BRD Objective | PRD Feature | FRD Use Case | Requirements | Business/Validation Rules | Test Scenarios`. **No orphans**: every BRD objective must appear in this table with something in every column — a business objective with nothing tracing to it downstream is itself a gap, and should have been caught by `requirements-validator` (if it wasn't, say so in this document rather than silently completing the table anyway).
- **source:** [Tutorialspoint — Functional Requirements Document](https://www.tutorialspoint.com/business_analysis/business_analysis_functional_requirements_document.htm), [Savio — FRD Template & Examples](https://savioglobal.com/blog/business-analysis/functional-requirements-document-frd-template-examples/); user-sourced sections per `ignore_dir/FRD_SKILL_no_text_markdown_fences.md`.

## DOC-STD-004 — Shared Document-Control Block

- **grounding:** financial-services documentation control practice — regulated-industry documents are expected to carry explicit regulatory references, named approval authority, version history, and captured sign-off, distinct from the document's substantive content. The multi-approver Sign-Off model below is **user-sourced** (previously a single approver per document) — see `ignore_dir/BRD_SKILL_cleaned.md`'s approval matrix.
- **required fields**, present at the top of every generated document (BRD/PRD/FRD alike):
  - `document_type`, `version` (starts at `0.1`, bumped on any post-signoff revision)
  - `project_id`, `run_id` (the `.agentbuilder` run this document belongs to)
  - `vertical`, `usecase` (from `scope-bind.yaml`)
  - `prepared_by` — literally "Agent Builder plugin" plus the session's operator context, not a fabricated human author
  - `date_prepared`
  - `status` — `draft` / `pending_signoff` / `signed` — `signed` only once **every** required approver below has signed; if any are still outstanding, status stays `pending_signoff` even if some have already signed (see per-approver tracking below).
  - `regulatory_references` — the specific `BCM-REG-###`/`BCM-TECH-###`/usecase-rule IDs actually relevant to this document's content, pulled from the KB, never invented or left generic
- **required approver roles, per document type** — this is who `requirements-doc-writer` must obtain sign-off from, sourced from the BRD's Stakeholders section (`DOC-STD-001`) rather than invented per document:
  - **BRD**: Executive Sponsor, Product Owner, Business Owner, Technical Lead/Architect, Compliance/Security Representative (5 — the full matrix, since the BRD is the governing business document).
  - **PRD**: Product Owner, Business Owner, Technical Lead/Architect (3 — product-level sign-off).
  - **FRD**: Technical Lead/Architect, QA Lead, Compliance/Security Representative (3 — technical sign-off, since the FRD is "a formal statement of an application's functional requirements that serves the same purpose as a contract" for the people who build and verify it).
  - A project may need a different set for a specific usecase — if so, that's a usecase-manifest-level override, not something `requirements-doc-writer` should improvise per run.
- **required fields, per approver, in the Sign-Off section** (appears at the bottom, see `skills/document-rendering/SKILL.md` for the concrete HTML rendering):
  - `approver_name` — filled in only once actually provided by a person in the session, never pre-filled or guessed
  - `approver_role` — must be one of the required roles for this document type above
  - `affirmation_statement` — e.g. "I have reviewed and approve this [document type] as documented above."
  - `signed_at` — timestamp, captured at the moment of *that approver's* affirmative approval, not backdated
  - `evidence_hash` — **one hash per document, shared across all its approvers**, computed once from the document's content in its `pending_signoff` state (before any approver signs) — it represents "what every approver reviewed," not a per-approver artifact. See `skills/requirements-doc-writer/SKILL.md` for exactly when this is computed.
- **Per-approver resumability**: log each approver's signature to `state.md` individually as it happens (`[human] Signed off <type> — approver: <name> (<role>), evidence_hash: <hash>`), the same granularity already used for individual governance fields and intake fields elsewhere in this plugin — this is what lets a document with 3 of 5 approvers signed resume correctly instead of re-asking already-signed roles or losing track of who's left.
- **source:** [Docsie — Financial Services Documentation Templates 2026](https://www.docsie.io/blog/articles/financial-services-documentation-templates-2026/), [ProjectManagement.com — BRD Sign-off Template](https://www.projectmanagement.com/deliverables/287499/business-requirements-document---customer-client-sign-off-template); multi-approver model per `ignore_dir/BRD_SKILL_cleaned.md`.

## DOC-STD-005 — Content Depth Standard

- **grounding:** internal quality bar, added after a review of an early rendering test found the mechanics (structure, styling, sign-off flow) were sound but the placeholder content used to test them was exactly the kind of one-liner-per-section output this standard exists to prevent. Not an external citation — a self-imposed bar because none of DOC-STD-001/002/003 said how much content each section actually needs, and left unspecified, an LLM filling nine sections quickly will default to terse.
- **Sections that are brief by genre convention — do not pad these:**
  - BRD **Executive Summary** — 3–5 sentences. It exists so a reader who reads nothing else still understands the document; padding it defeats the point.
  - PRD **Problem Statement** — a short paragraph, framed from the user's perspective, not a restated title.
- **Sections that require real substance — a single sentence here is a defect, not a valid output:**
  - BRD **Business Objectives, Project Scope, Stakeholders, Current State Analysis, Assumptions, Constraints, Risks, Success Criteria, KPIs** — each is a list or table; every row must be complete and specific, not a fragment. "Risk: fraud" is not acceptable. "Risk: an attacker who has partially compromised a customer's identity could use a replacement-card request to redirect a new card to an address they control — mitigated by CARD-CTL-005's mandatory step-up gate on any reissue-to-new-address request" is the bar.
  - PRD **User Personas** — each persona needs a real narrative (2–3 sentences): who they are, what situation brings them to this agent, what they need from the interaction — not just a job title.
  - PRD **Feature Requirements** — each feature needs a name plus a substantive description (2–4 sentences: what it does, why it's needed, any conditions/exceptions) and its `usecase-rules.md` cross-reference. A bare feature title with no description is not acceptable. See the worked example below.
  - PRD **Out-of-Scope, Dependencies, Open Questions** — same complete-sentence bar as the BRD lists above.
  - FRD **Use Cases** — each one is a short narrative or numbered step sequence (not a one-line label) describing an actual scenario from a specific user role's perspective, including the trigger, main flow, at least one alternative flow, and at least one exception flow (per `DOC-STD-003`).
  - FRD **Business Rules, Security Requirements** — each rule/requirement row states the actual rule/expectation in full, with its source, not a keyword. A `Business Rules` row reading only "PAN masking" with no rule text is not acceptable.
  - FRD **Workflows** — numbered steps, each step a full sentence describing what happens and who/what performs it; any human-approval-gate interrupt point must be visually and textually called out, not buried in a step like any other.
  - FRD **Validation Rules, Data Requirements, Test Scenarios, Traceability Matrix** — table rows are acceptable *if* each row's content is specific and complete, not a fragment. A `Validation Rules` row reading only "PAN masking" with no rule text is not acceptable; it must state the actual rule ("Display shows no more than the first 6 and last 4 digits of the PAN in any output") alongside its reference ID. A `Traceability Matrix` row with a blank column is a gap, not a formatting choice — if something genuinely doesn't trace to a test scenario yet, say so explicitly rather than leaving the cell empty.
- **Anti-patterns to actively avoid:**
  - Any list where every item is under ~8 words.
  - Generic filler that could apply to any project ("various risks exist," "several features are planned," "data will be handled appropriately").
  - A Feature Requirements or Use Cases section with only titles and no elaboration.
  - Treating the KB cross-reference ID as a substitute for actually explaining the rule in the document's own words — the reference supports the sentence, it doesn't replace it.
  - An empty cell in a table (Assumptions, Risks, Traceability Matrix, etc.) where the honest content is "unknown" or "not yet determined" — write that explicitly, don't leave it blank.

### Worked example — what "good" looks like for one PRD Feature Requirement entry

> **Feature: Immediate Card Block** *(implements `CARD-UC-R002`)*
> Once the customer's identity has been verified, the agent blocks the reported card immediately, without requiring a separate approval step. This is a deliberate design choice rather than a limitation: blocking is a protective, reversible action, and introducing an approval delay here would leave a customer exposed to further fraud while waiting for a human to respond. The block action is logged with the exact report timestamp, since that timestamp determines the customer's liability protection under `BCM-REG-006` (TILA §1643/FCBA).

### Worked example — what "good" looks like for one BRD Business Objective row and one FRD Traceability Matrix row

> `O1 | Reduce median time-to-block for a reported lost/stolen card to under 60 seconds | Measured via audit-log timestamp delta between report and block confirmation | <60s median | Within first production release`

> `O1 | Immediate Card Block | UC-001: Verified cardholder reports lost card | CARD-UC-R002, CARD-CTL-004 | BR-001 (block permitted without approval gate) | TS-001 (happy-path block), TS-E001 (verification fails mid-block)`

That's the depth bar — every column filled with something specific and checkable, not a placeholder.

---

## Relationship to other core files

- The **mechanics** of a sign-off (no silent proceed, evidence-before-consent, logged with actor/timestamp/case_id/evidence_hash) are already defined in `core/approval-policy.md` — `DOC-STD-004`'s Sign-Off section is that mechanism applied specifically to a generated document (now with multiple approvers per document) rather than a runtime agent action.
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
- User-provided reference material, reviewed and selectively adapted (not adopted verbatim — see notes above for what was and wasn't incorporated, and why): `ignore_dir/BRD_SKILL_cleaned.md`, `ignore_dir/FRD_SKILL_no_text_markdown_fences.md`.

Full research trail in the local, gitignored `ignore_dir/research-sources.md` §H.
