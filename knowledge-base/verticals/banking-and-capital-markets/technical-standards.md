# Technical Standards — Banking & Capital Markets

> **Disclaimer:** Compressed, illustrative summary compiled 2026-07-24. **Not legal advice.** Verify against the current PCI-DSS version and your organization's actual security requirements before production use.

Vertical-wide technical requirements — apply to any use case handling the relevant data type, not just one workflow.

## Entry Format

- `standard_id` — stable identifier (`BCM-TECH-###`)
- `name`
- `citation`
- `applies_when`
- `requirement`
- `source`

## BCM-TECH-001 — PAN Masking (PCI-DSS Requirement 3.3)

- **citation:** PCI Security Standards Council, PCI-DSS v4.0, Requirement 3.3
- **applies_when:** any output that references a card's Primary Account Number (PAN), to any party, in any channel — this includes chat responses, logs, exports, and error messages.
- **requirement:** Display masked to a **maximum of the first 6 and last 4 digits**; full PAN must never be shown unless there is a legitimate business need and the viewer is authorized personnel. Masking (temporary, for display) is distinct from truncation (permanent, for storage) — truncated storage is also bound to first-6/last-4 max.
- **source:** [RSI Security — PCI DSS Masking Requirements](https://blog.rsisecurity.com/pci-dss-masking-requirements/), [PCI DSS Guide — Acceptable Formats for Truncation of PAN](https://pcidssguide.com/what-are-the-acceptable-formats-for-truncation-of-pan/)

## BCM-TECH-002 — GLBA Safeguards Rule Technical Controls

- **citation:** 16 CFR Part 314 (see `regulatory-landscape.md` → `BCM-REG-002` for the full regulatory context)
- **applies_when:** any system storing or processing customer NPI (nonpublic personal information).
- **requirement:** Access controls and encryption for NPI at rest and in transit; maintained audit trails; periodic security testing; a designated qualified individual responsible for the security program.
- **source:** [eCFR — 16 CFR Part 314](https://www.ecfr.gov/current/title-16/chapter-I/subchapter-C/part-314)

## BCM-TECH-003 — Layered Authentication (FFIEC)

- **citation:** FFIEC interagency guidance (see `regulatory-landscape.md` → `BCM-REG-004`)
- **applies_when:** any action that affects an account or discloses non-public account information.
- **requirement:** Single-factor checks (e.g. possession of a card/account number alone) are insufficient. Use layered, risk-based, multi-factor verification — "out-of-wallet" checks the requester couldn't answer just by having stolen a card or a piece of mail.
- **source:** [CFPB — FFIEC Authentication Guidance](https://www.consumerfinance.gov/about-us/newsroom/ffiec-issues-guidance-on-authentication-and-access-to-financial-institution-services-and-systems/)

## Sources

- [RSI Security — PCI DSS Masking Requirements: Comprehensive Guide](https://blog.rsisecurity.com/pci-dss-masking-requirements/)
- [PCI DSS Guide — Acceptable Formats for Truncation of PAN](https://pcidssguide.com/what-are-the-acceptable-formats-for-truncation-of-pan/)
- [eCFR — 16 CFR Part 314](https://www.ecfr.gov/current/title-16/chapter-I/subchapter-C/part-314)
- [CFPB — FFIEC Authentication Guidance](https://www.consumerfinance.gov/about-us/newsroom/ffiec-issues-guidance-on-authentication-and-access-to-financial-institution-services-and-systems/)

Full research trail in the local, gitignored `ignore_dir/research-sources.md` §D, §F.
