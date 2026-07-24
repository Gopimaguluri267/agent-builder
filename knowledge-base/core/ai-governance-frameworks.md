# AI Governance Frameworks (Core)

Domain-agnostic. These are cross-industry frameworks for governing AI/agentic systems — not written by or for any specific regulator or industry vertical. They ground the *practice* behind our own core mechanics (`risk-tiers.md`, `approval-policy.md`, `pii-taxonomy.md`, `build-scope.md`) and the four named guardrail categories (`pii_redaction`, `prompt_injection_detection`, `escalation_policy`, `human_approval_required`). A vertical's own regulatory citations (e.g. a banking-specific model risk management mandate) sit on top of these, not in place of them — see `verticals/<vertical>/` for the industry-specific layer.

**MD only — no JSON twin.** These are reference/citation content for humans and LLM agents to reason about; no script parses framework structure programmatically.

## Entry Format

- `framework_id` — stable identifier (`AIGF-###`)
- `framework_name`
- `publisher`
- `type` — management system / risk-management operating model / security taxonomy
- `scope_note` — why this is cross-industry, not vertical-specific
- `key_structure` — the framework's core organizing concept
- `grounds` — which of our own core mechanics or guardrail categories this framework's practice underlies
- `source`

## AIGF-001 — NIST AI Risk Management Framework (AI RMF)

- **publisher:** National Institute of Standards and Technology (NIST), part of the US Department of Commerce
- **type:** risk-management operating model
- **scope_note:** General-purpose, voluntary framework for organizations developing or using AI systems in any sector — not issued by or for a financial, healthcare, or any other specific regulator.
- **key_structure:** Four core functions — **Govern** (culture/policy for managing AI risk), **Map** (understand context and identify risks), **Measure** (analyze/assess/track risks), **Manage** (prioritize and act on risks) — applied continuously, not as a one-time gate.
- **grounds:** The overall shape of `core/risk-tiers.md` (risk-scaled mandatory controls) and the plugin's own Scope & Bind → build → gate pipeline is a Map/Measure/Manage cycle in miniature.
- **source:** referenced via [Modulos — AI Governance Frameworks Comparison](https://docs.modulos.ai/frameworks/comparison/index)

## AIGF-002 — ISO/IEC 42001

- **publisher:** International Organization for Standardization (ISO) / International Electrotechnical Commission (IEC)
- **type:** management system standard
- **scope_note:** An AI management system standard, structurally analogous to ISO/IEC 27001 for information security — applies to any organization operating an AI management system, regardless of industry.
- **key_structure:** Establishes requirements for an organization's AI management system: policy, roles/responsibilities, risk assessment, objectives, and continual improvement — a management-system wrapper around whatever specific risk practices (e.g. NIST AI RMF) an organization uses.
- **grounds:** The rationale for why this plugin insists on a governance envelope (`scope-bind.yaml`) being declared and validated *before* any generation happens, rather than bolted on afterward — that's the "AI management system" discipline operationalized.
- **source:** referenced via [Modulos — AI Governance Frameworks Comparison](https://docs.modulos.ai/frameworks/comparison/index), [eccouncil.org — EU AI Act vs NIST AI RMF vs ISO/IEC 42001](https://www.eccouncil.org/cybersecurity-exchange/responsible-ai-governance/eu-ai-act-nist-ai-rmf-and-iso-iec-42001-a-plain-english-comparison/)

## AIGF-003 — OWASP Top 10 for Large Language Model Applications

- **publisher:** OWASP (Open Worldwide Application Security Project) Foundation
- **type:** security taxonomy
- **scope_note:** Names concrete, observable security risks in LLM-based applications — a technology-scoped list (LLM applications), not an industry-scoped one. Feeds evidence into higher-order frameworks like NIST AI RMF/ISO 42001 rather than replacing them.
- **key_structure:** Named risk categories including Prompt Injection (LLM01), Insecure Output Handling, Training Data Poisoning, Sensitive Information Disclosure, Insecure Plugin Design, Excessive Agency, and others.
- **grounds:** Directly grounds our `prompt_injection_detection` guardrail category (LLM01) and informs why `core/pii-taxonomy.md`'s redaction rules exist (Sensitive Information Disclosure).
- **source:** referenced via [Modulos — OWASP for AI Security](https://docs.modulos.ai/frameworks/owasp/index)

## AIGF-004 — OWASP Top 10 for Agentic Applications

- **publisher:** OWASP Foundation
- **type:** security taxonomy
- **scope_note:** Extends (does not replace) the LLM Top 10 for systems that take autonomous action, not just generate text — since "most agent systems are also LLM applications and inherit the LLM-side risks." Technology-scoped (agentic systems), not industry-scoped.
- **key_structure:** Names risks specific to autonomous action-taking — excessive agency, unsafe tool invocation, goal manipulation, insufficient human oversight of consequential actions, among others.
- **grounds:** Directly grounds `escalation_policy` and `human_approval_required` — the reason this plugin treats "which actions can proceed autonomously vs. require a gate" as the central design question for every use case (see, e.g., the card-blocking usecase's block-vs-reissue-to-new-address distinction).
- **source:** referenced via [Modulos — OWASP Top 10 for Agentic Applications (2026)](https://docs.modulos.ai/frameworks/owasp-top-10-agentic/index)

## How these four fit together

NIST AI RMF and ISO/IEC 42001 both describe *the system* that manages AI risk (the Govern/Map/Measure/Manage loop, or the management-system wrapper around it); the two OWASP lists populate that system with *named, concrete risks* to evaluate and treat. None of the four are specific to finance, healthcare, or any other vertical — that specificity is added by the vertical layer, which cites *its own* regulator's mandates (e.g. `verticals/banking-and-capital-markets/model-risk-management.md`) as the industry-specific application of the same underlying practice these four frameworks describe generically.

## Sources

- [Modulos — AI Governance Frameworks Comparison (EU AI Act, ISO 42001, NIST AI RMF, OWASP LLM)](https://docs.modulos.ai/frameworks/comparison/index)
- [Modulos — OWASP Top 10 for Agentic Applications (2026)](https://docs.modulos.ai/frameworks/owasp-top-10-agentic/index)
- [Modulos — OWASP for AI Security](https://docs.modulos.ai/frameworks/owasp/index)
- [eccouncil.org — EU AI Act vs NIST AI RMF vs ISO/IEC 42001: A Plain English Comparison](https://www.eccouncil.org/cybersecurity-exchange/responsible-ai-governance/eu-ai-act-nist-ai-rmf-and-iso-iec-42001-a-plain-english-comparison/)

Full research trail in the local, gitignored `ignore_dir/research-sources.md` §E — not shipped with the plugin, kept for internal traceability only.
