# Regulatory Landscape — Banking & Capital Markets

> **Disclaimer:** Compressed, illustrative summary of publicly available US federal regulatory guidance, compiled 2026-07-24 from the sources cited inline and listed under "Sources" below. **Not legal advice**, not a substitute for review by qualified compliance/legal counsel. Exact figures, deadlines, and scope details must be verified against current regulator guidance before any production use. This file is vertical-wide — it applies to *any* banking/capital-markets use case built on this pack, not just one. A specific usecase's `usecase-rules.md` should cite these entries by ID rather than restating them.

Covers **US federal** regulation only — see `vertical-manifest.yaml` → `open_gaps` for what's not covered (capital-markets-specific rules, EU/UK equivalents).

## Entry Format

- `requirement_id` — stable identifier (`BCM-REG-###`)
- `name`
- `citation` — the actual statute/regulation/CFR part
- `applies_when` — what triggers relevance to a generated agent
- `summary`
- `source`

---

## BCM-REG-001 — PCI-DSS (Payment Card Industry Data Security Standard)

- **citation:** PCI Security Standards Council, PCI-DSS v4.0
- **applies_when:** the agent handles, displays, stores, or transmits payment card data (PAN, cardholder data) in any form.
- **summary:** Industry-mandated (not a government statute, but contractually required by card networks) security standard for cardholder data. Requirement 3.3 specifically governs PAN masking/truncation — see `technical-standards.md` for the operational detail.
- **source:** see `technical-standards.md` for full citation.

## BCM-REG-002 — GLBA (Gramm-Leach-Bliley Act) Safeguards Rule

- **citation:** GLBA §501 & §505(b)(2); Safeguards Rule at [16 CFR Part 314](https://www.ecfr.gov/current/title-16/chapter-I/subchapter-C/part-314)
- **applies_when:** the agent handles customers' "nonpublic personal information" (NPI) — names, addresses, phone numbers, account numbers, income/credit histories, SSNs.
- **summary:** Requires financial institutions to maintain a written information security program with administrative, technical, and physical safeguards appropriate to size/complexity/data sensitivity. 2021 amendments added specific requirements: a designated qualified individual overseeing the program, periodic risk assessments, access controls and encryption, audit trails, security testing, incident response planning.
- **source:** [eCFR — 16 CFR Part 314](https://www.ecfr.gov/current/title-16/chapter-I/subchapter-C/part-314), [FTC — Gramm-Leach-Bliley Act](https://www.ftc.gov/business-guidance/privacy-security/gramm-leach-bliley-act)

## BCM-REG-003 — BSA (Bank Secrecy Act) — General Provisions

- **citation:** Bank Secrecy Act, 31 U.S.C. Chapter 53, as implemented via FinCEN regulations
- **applies_when:** the agent operates in a context involving customer identification, transaction monitoring, or recordkeeping for a financial institution.
- **summary:** Establishes recordkeeping and reporting obligations for financial institutions, including Customer Identification Program (CIP) requirements and the SAR (Suspicious Activity Report) filing obligation (see the `aml-transaction-monitoring` usecase in `ignore_dir/knowledge-base-packs-v1/finance-aml/` for SAR-specific detail — not currently an active usecase under this vertical, preserved for reference).
- **source:** [FinCEN — Suspicious Activity Reports (SARs)](https://www.fincen.gov/suspicious-activity-reports-sars), [FFIEC BSA/AML Examination Manual](https://bsaaml.ffiec.gov/manual)

## BCM-REG-004 — FFIEC Guidance (Authentication & Access)

- **citation:** FFIEC (Federal Financial Institutions Examination Council) interagency guidance
- **applies_when:** the agent performs identity verification or grants access to an account or system before taking an action.
- **summary:** Calls for layered, risk-based, multi-factor authentication — moving beyond single-factor checks (e.g. a card number alone) toward "out-of-wallet" verification for account-affecting actions. General guidance, not specific to any one workflow.
- **source:** [CFPB — FFIEC Issues Guidance on Authentication and Access to Financial Institution Services and Systems](https://www.consumerfinance.gov/about-us/newsroom/ffiec-issues-guidance-on-authentication-and-access-to-financial-institution-services-and-systems/)

## BCM-REG-005 — Regulation E / EFTA (Electronic Fund Transfer Act)

- **citation:** 15 U.S.C. §1693g; 12 CFR §1005.6
- **applies_when:** the agent handles **debit card** or electronic-fund-transfer unauthorized-transaction claims. **Not the same regime as credit cards — see BCM-REG-006.**
- **summary:** Tiered consumer liability for unauthorized EFTs: capped at $50 if reported within 2 business days of discovery; up to $500 if reported within 60 days; unlimited liability for transfers occurring after 60 days from the relevant statement, if unreported. Burden of proof that a transaction was authorized sits with the financial institution.
- **source:** [CFPB — 12 CFR §1005.6](https://www.consumerfinance.gov/rules-policy/regulations/1005/6/), [Consumer Compliance Outlook — Consumer Liability under EFTA/Reg E](https://www.consumercomplianceoutlook.org/2025/third-issue/consumer-liability/)

## BCM-REG-006 — TILA §1643 / Fair Credit Billing Act (FCBA)

- **citation:** Truth in Lending Act §1643, as implemented via the Fair Credit Billing Act
- **applies_when:** the agent handles **credit card** unauthorized-charge claims. **Not the same regime as debit/EFT — see BCM-REG-005.**
- **summary:** Caps consumer liability for unauthorized credit card charges at $50; **zero liability for any charges made after the loss/theft is reported.** Card issuer bears the burden of proof.
- **source:** [Discover — What Is the Fair Credit Billing Act?](https://www.discover.com/credit-cards/card-smarts/fair-credit-billing-act/), [consumerprotection.net — TILA liability limits](https://consumerprotection.net/blog/the-truth-in-lending-acts-little-known-limits-on-liability-for-unauthorized-credit-card-use/)

## BCM-REG-007 — Regulation B / ECOA (Equal Credit Opportunity Act)

- **citation:** 12 CFR Part 1002 (adverse action notice requirements at §1002.9)
- **applies_when:** the agent is involved in a lending/credit decision or communicates a credit decision outcome to an applicant. **Not yet relevant to any currently-active usecase under this vertical** — included here so a future lending usecase doesn't have to re-derive it.
- **summary:** Prohibits credit discrimination; requires adverse action notices within specific timeframes (30 days after a completed-application decision, incomplete-application adverse action, or existing-account adverse action; 90 days after an unaccepted counteroffer). Notice must disclose the creditor's identity, an ECOA antidiscrimination statement, the primary regulator's contact info, the action taken, and either specific reasons or the right to request them.
- **source:** [eCFR — 12 CFR Part 1002](https://www.ecfr.gov/current/title-12/chapter-X/part-1002), [FDIC Consumer Compliance Examination Manual — ECOA](https://www.fdic.gov/consumer-compliance-examination-manual/v-7-equal-credit-opportunity-act-ecoa)

## BCM-REG-008 — UDAAP (Unfair, Deceptive, or Abusive Acts or Practices)

- **citation:** Dodd-Frank Wall Street Reform and Consumer Protection Act; CFPB rulemaking authority
- **applies_when:** any customer-facing banking agent — this is a general conduct standard, not tied to one workflow.
- **summary:** Prohibits unfair, deceptive, or abusive acts/practices toward consumers of financial products/services. "Unfair" is a 3-part test: (1) causes or is likely to cause substantial injury, (2) not reasonably avoidable by the consumer, (3) not outweighed by countervailing benefits. Relevant to how a generated agent communicates with customers — e.g. a liability disclosure that's technically accurate but presented misleadingly could still raise UDAAP concerns.
- **source:** [NCUA — UDAAP overview](https://ncua.gov/regulation-supervision/manuals-guides/federal-consumer-financial-protection-guide/compliance-management/unfair-deceptive-or-abusive-acts-or-practices-udaap), [FDIC — UDAAP](https://www.fdic.gov/consumer-compliance/unfair-deceptive-or-abusive-acts-or-practices)

## Sources

- [eCFR — 16 CFR Part 314 (GLBA Safeguards Rule)](https://www.ecfr.gov/current/title-16/chapter-I/subchapter-C/part-314)
- [FTC — Gramm-Leach-Bliley Act](https://www.ftc.gov/business-guidance/privacy-security/gramm-leach-bliley-act)
- [FinCEN — Suspicious Activity Reports (SARs)](https://www.fincen.gov/suspicious-activity-reports-sars)
- [FFIEC BSA/AML Examination Manual](https://bsaaml.ffiec.gov/manual)
- [CFPB — FFIEC Authentication Guidance](https://www.consumerfinance.gov/about-us/newsroom/ffiec-issues-guidance-on-authentication-and-access-to-financial-institution-services-and-systems/)
- [CFPB — 12 CFR §1005.6 (Regulation E)](https://www.consumerfinance.gov/rules-policy/regulations/1005/6/)
- [Consumer Compliance Outlook — Consumer Liability under EFTA/Reg E](https://www.consumercomplianceoutlook.org/2025/third-issue/consumer-liability/)
- [Discover — What Is the Fair Credit Billing Act?](https://www.discover.com/credit-cards/card-smarts/fair-credit-billing-act/)
- [consumerprotection.net — TILA liability limits](https://consumerprotection.net/blog/the-truth-in-lending-acts-little-known-limits-on-liability-for-unauthorized-credit-card-use/)
- [eCFR — 12 CFR Part 1002 (Regulation B)](https://www.ecfr.gov/current/title-12/chapter-X/part-1002)
- [FDIC Consumer Compliance Examination Manual — ECOA](https://www.fdic.gov/consumer-compliance-examination-manual/v-7-equal-credit-opportunity-act-ecoa)
- [NCUA — UDAAP overview](https://ncua.gov/regulation-supervision/manuals-guides/federal-consumer-financial-protection-guide/compliance-management/unfair-deceptive-or-abusive-acts-or-practices-udaap)

Full research trail in the local, gitignored `ignore_dir/research-sources.md` §D, §F.
