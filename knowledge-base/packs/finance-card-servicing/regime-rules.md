# Regime Rules — finance-card-servicing

> **Disclaimer:** This is a compressed, illustrative summary of publicly available US TILA/FCBA, PCI-DSS, and FFIEC guidance, compiled 2026-07-23 from the sources cited inline and listed in full under "Sources" below. It is **not legal advice** and is **not a substitute for review by qualified compliance/legal counsel**. Exact liability figures and procedural requirements change over time and must be verified against current authoritative guidance before any production use. Two of the six requirements below (`FIN-CARD-R002`, `FIN-CARD-R005`) are **operational design principles**, not direct regulatory citations — this is called out explicitly in their `source` field rather than implied to be something it isn't.

Covers **US, credit cards specifically** (see `pack-manifest.yaml` → `open_gaps` for debit/EU/UK status).

## Entry Format

Same convention as `finance-aml/regime-rules.md` — `requirement_id`, `requirement_text`, `applies_when`, `requires_controls`, `severity_floor`, `source`, `prompt_directive`.

---

## FIN-CARD-R001 — Identity Verification Before Any Action

- **requirement_text:** The agent must verify the identity of the person it is speaking with, using layered/risk-based verification, before performing any account-affecting action or disclosing any non-public account information.
- **applies_when:** Any interaction where the user requests an action on, or information about, a specific account.
- **requires_controls:** `CARD-CTL-003`
- **severity_floor:** blocker
- **source:** [FFIEC guidance on authentication and access to financial institution services](https://www.consumerfinance.gov/about-us/newsroom/ffiec-issues-guidance-on-authentication-and-access-to-financial-institution-services-and-systems/) — FFIEC supports a layered-security approach and identifies weaknesses in single-factor authentication; guidance calls for multi-factor, risk-based, "out-of-wallet" verification for account-affecting actions. General authentication guidance, not card-blocking-specific.
- **prompt_directive:** "Before taking any action on an account or disclosing any account information, verify you are actually speaking with the account holder or someone authorized on the account. Do not treat possession of a card number alone as sufficient verification."

## FIN-CARD-R002 — Immediate Block Permitted Without a Separate Approval Gate

- **requirement_text:** Once identity is verified, the agent may block a reported lost/stolen card immediately, without a separate human-approval step. This is a deliberate design choice, not a regulatory requirement in itself.
- **applies_when:** A verified cardholder reports their card lost or stolen and requests it be blocked.
- **requires_controls:** `CARD-CTL-004`
- **severity_floor:** medium
- **source:** **Operational design principle, not a direct regulatory citation.** Rationale: blocking is protective (stops further unauthorized use) and reversible (can be undone if it was a mistaken report), so gating it behind an approval step would add friction to something that exists to protect the customer, without a corresponding safety benefit. This is consistent with the general intent of TILA/FCBA's rapid-reporting liability protection (see `FIN-CARD-R003`) — the law's own design rewards fast reporting, so the agent's own flow shouldn't slow that down.
- **prompt_directive:** "Once you have verified identity, you may block the reported card immediately — you do not need a separate approval step for the block action itself. Acting quickly here directly protects the customer."

## FIN-CARD-R003 — Liability Disclosure & Report-Timestamp Logging

- **requirement_text:** The agent must inform the customer of their liability protection ($50 cap on liability for charges before the report; zero liability for charges after the report is made) and must log the exact timestamp of the report, since that timestamp is what determines the liability cutoff.
- **applies_when:** A card is reported lost/stolen and/or blocked.
- **requires_controls:** `CARD-CTL-002`
- **severity_floor:** high
- **source:** Truth in Lending Act §1643, as implemented via the Fair Credit Billing Act — [Discover: What Is the Fair Credit Billing Act?](https://www.discover.com/credit-cards/card-smarts/fair-credit-billing-act/) ("You're not liable for charges made after you've reported your card as stolen... once you notify your card issuer that your card is lost or stolen, you have no liability for any subsequent fraudulent charges"); [consumerprotection.net — TILA liability limits](https://consumerprotection.net/blog/the-truth-in-lending-acts-little-known-limits-on-liability-for-unauthorized-credit-card-use/) ("TILA §1643 ... caps your liability at $50 and places the burden of proof on the card issuer").
- **prompt_directive:** "Tell the customer clearly: they are not liable for any charges made after this report, and their liability for charges before the report is capped at $50 (many issuers waive this to zero — check the specific card's terms). Log the exact date and time of this report — it is the legal basis for that protection."

## FIN-CARD-R004 — PAN Masking in All Outputs

- **requirement_text:** The agent must never display a full, unmasked card number (Primary Account Number / PAN) in any output, to any party, including the verified cardholder. Display is limited to a maximum of the first 6 and last 4 digits.
- **applies_when:** Any output that references a card number.
- **requires_controls:** `CARD-CTL-001`
- **severity_floor:** blocker
- **source:** PCI-DSS Requirement 3.3 — [RSI Security: PCI DSS Masking Requirements](https://blog.rsisecurity.com/pci-dss-masking-requirements/) ("Requirement 3.3 of the PCI DSS mandates organizations to mask PANs when they are displayed, ensuring truncated PAN cardholder data to display only a maximum of the first six and last four digits at any time. The full PAN must never be shown unless there is a legitimate business need, and only authorized personnel can access it").
- **prompt_directive:** "Never include a full card number in anything you say or write, to anyone — including the verified cardholder, who doesn't need it redisplayed since they already have the physical card. If you need to reference a card, use only the first 6 and last 4 digits, matching the format the customer would already recognize."

## FIN-CARD-R005 — Reissue-to-New-Address Requires Step-Up Verification / Approval

- **requirement_text:** If the customer requests a replacement card be sent to an address other than the one on file, the agent must not fulfill this directly — it must route the request through a step-up verification / human-approval gate before the address change and reissue can proceed.
- **applies_when:** A replacement-card request specifies a delivery address different from the address currently on file.
- **requires_controls:** `CARD-CTL-005`
- **severity_floor:** blocker
- **source:** **Operational/fraud-control design principle, grounded in FFIEC's risk-based authentication guidance** ([FFIEC guidance summary](https://www.consumerfinance.gov/about-us/newsroom/ffiec-issues-guidance-on-authentication-and-access-to-financial-institution-services-and-systems/)) rather than a single named statute. This is a well-recognized account-takeover pattern in the card-servicing industry: an attacker who has partially compromised a customer's identity uses a "lost card, please send the replacement here" request as a way to redirect a new card to an address they control. The block itself (`FIN-CARD-R002`) should not wait on this gate — only the address-change/reissue sub-action should.
- **prompt_directive:** "If a replacement card is requested for delivery to an address that is not the one currently on file, do not fulfill this yourself. Explain to the customer that address changes for card delivery require an additional verification step, and route the request accordingly — but this does not affect or delay the block itself, which has already happened."

## FIN-CARD-R006 — Reissue-to-Address-on-File May Proceed Without Additional Gate

- **requirement_text:** If a replacement is requested for delivery to the address already on file, the agent may arrange it directly, without the step-up gate required by `FIN-CARD-R005`.
- **applies_when:** A replacement-card request specifies the address currently on file (or no address, defaulting to the address on file).
- **requires_controls:** `CARD-CTL-004`
- **severity_floor:** medium
- **source:** **Operational design principle** — the address-on-file has already passed whatever verification standard was used to establish it as part of the account; redirecting a physical card to a new address is the actual point of incremental risk, not sending it to a location already associated with the verified customer.
- **prompt_directive:** "If the customer wants their replacement sent to the address already on file, you may arrange that directly — no additional gate is needed beyond the identity verification already completed for this interaction."

---

## Definitions (for context, not independently machine-checked)

- **PAN (Primary Account Number):** The card number itself. See PCI-DSS Requirement 3.3, cited above.
- **Liability cap:** The maximum amount a cardholder can be held responsible for in unauthorized-charge cases, per TILA §1643/FCBA — $50 before report, $0 after.
- **Step-up verification:** An additional, stronger authentication check beyond whatever was used for the base interaction — invoked specifically for higher-risk actions like an address change, per FFIEC's risk-based authentication guidance.

## Sources

- [Discover — What Is the Fair Credit Billing Act?](https://www.discover.com/credit-cards/card-smarts/fair-credit-billing-act/)
- [consumerprotection.net — The Truth in Lending Act's Little-Known Limits on Liability for Unauthorized Credit Card Use](https://consumerprotection.net/blog/the-truth-in-lending-acts-little-known-limits-on-liability-for-unauthorized-credit-card-use/)
- [Experian — What Is the Fair Credit Billing Act (FCBA)?](https://www.experian.com/blogs/ask-experian/what-is-the-fair-credit-billing-act/)
- [RSI Security — PCI DSS Masking Requirements: Comprehensive Guide](https://blog.rsisecurity.com/pci-dss-masking-requirements/)
- [PCI DSS Guide — Acceptable Formats for Truncation of PAN](https://pcidssguide.com/what-are-the-acceptable-formats-for-truncation-of-pan/)
- [CFPB — FFIEC Issues Guidance on Authentication and Access to Financial Institution Services and Systems](https://www.consumerfinance.gov/about-us/newsroom/ffiec-issues-guidance-on-authentication-and-access-to-financial-institution-services-and-systems/)

Not independently re-cited here but noted for completeness: Regulation E / EFTA (15 U.S.C. §1693g, 12 CFR §1005.6) is the debit-card equivalent regime, deliberately out of scope for this pass — see `pack-manifest.yaml` → `open_gaps`.
