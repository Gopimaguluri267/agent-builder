# Usecase Rules — card-blocking-servicing

> **Disclaimer:** compiled 2026-07-24, illustrative pattern, not legal advice — verify before production use. This file is deliberately thin: the regulatory grounding lives at the vertical layer (`../../regulatory-landscape.md`, `../../technical-standards.md`) and is cited here by ID, not restated.

## Entry Format

Same convention as the vertical layer — `requirement_id`, `requirement_text`, `applies_when`, `requires_controls`, `severity_floor`, `references` (vertical-layer IDs this builds on, if any), `prompt_directive`.

---

## CARD-UC-R001 — Identity Verification Before Any Action

- **requirement_text:** The agent must verify the identity of the person it is speaking with, using layered/risk-based verification, before performing any account-affecting action or disclosing any non-public account information.
- **applies_when:** any interaction requesting an action on, or information about, a specific account.
- **requires_controls:** `CARD-CTL-003`
- **references:** `BCM-REG-004`, `BCM-TECH-003` (FFIEC layered authentication — this usecase's specific application, not a new rule)
- **severity_floor:** blocker
- **prompt_directive:** "Before taking any action on an account or disclosing any account information, verify you are actually speaking with the account holder or someone authorized on the account. Do not treat possession of a card number alone as sufficient verification."

## CARD-UC-R002 — Immediate Block Permitted Without a Separate Approval Gate

- **requirement_text:** Once identity is verified, the agent may block a reported lost/stolen card immediately, without a separate human-approval step. This is a deliberate design choice, not a regulatory requirement in itself.
- **applies_when:** a verified cardholder reports their card lost or stolen and requests it be blocked.
- **requires_controls:** `CARD-CTL-004`
- **references:** none directly — see `prompt_directive` for the rationale, which is consistent with (but not mandated by) `BCM-REG-006`'s own rapid-reporting incentive structure.
- **severity_floor:** medium
- **prompt_directive:** "Once you have verified identity, you may block the reported card immediately — you do not need a separate approval step for the block action itself. Acting quickly here directly protects the customer."

## CARD-UC-R003 — Liability Disclosure & Report-Timestamp Logging

- **requirement_text:** The agent must inform the customer of their liability protection and must log the exact timestamp of the report, since that timestamp determines the liability cutoff.
- **applies_when:** a card is reported lost/stolen and/or blocked.
- **requires_controls:** `CARD-CTL-002`
- **references:** `BCM-REG-006` (TILA §1643/FCBA — $50 cap before report, zero after) — see that entry for the full citation, not restated here.
- **severity_floor:** high
- **prompt_directive:** "Tell the customer clearly: they are not liable for any charges made after this report, and their liability for charges before the report is capped at $50 (many issuers waive this to zero — check the specific card's terms). Log the exact date and time of this report — it is the legal basis for that protection."

## CARD-UC-R004 — PAN Masking in All Outputs

- **requirement_text:** The agent must never display a full, unmasked card number in any output, to any party, including the verified cardholder.
- **applies_when:** any output that references a card number.
- **requires_controls:** `CARD-CTL-001`
- **references:** `BCM-TECH-001` (PCI-DSS Requirement 3.3) — full detail there, not restated here.
- **severity_floor:** blocker
- **prompt_directive:** "Never include a full card number in anything you say or write, to anyone — including the verified cardholder, who doesn't need it redisplayed since they already have the physical card. If you need to reference a card, use only the first 6 and last 4 digits."

## CARD-UC-R005 — Reissue-to-New-Address Requires Step-Up Verification / Approval

- **requirement_text:** If the customer requests a replacement card be sent to an address other than the one on file, the agent must route the request through a step-up verification / human-approval gate before the address change and reissue can proceed.
- **applies_when:** a replacement-card request specifies a delivery address different from the address on file.
- **requires_controls:** `CARD-CTL-005`
- **references:** `BCM-REG-004`, `BCM-TECH-003` (FFIEC layered authentication) — this usecase's specific, higher-bar application to a known account-takeover pattern in card servicing specifically; this exact scenario isn't independently cited by name in the vertical-wide FFIEC entry, so the risk reasoning is usecase-specific even though the underlying authentication principle isn't.
- **severity_floor:** blocker
- **prompt_directive:** "If a replacement card is requested for delivery to an address that is not the one currently on file, do not fulfill this yourself. Explain that address changes for card delivery require an additional verification step, and route the request accordingly — this does not affect or delay the block itself, which has already happened."

## CARD-UC-R006 — Reissue-to-Address-on-File May Proceed Without Additional Gate

- **requirement_text:** If a replacement is requested for delivery to the address already on file, the agent may arrange it directly, without the step-up gate required by `CARD-UC-R005`.
- **applies_when:** a replacement-card request specifies the address on file, or no address (defaulting to it).
- **requires_controls:** `CARD-CTL-004`
- **references:** none — the address-on-file has already passed whatever verification standard established it as part of the account.
- **severity_floor:** medium
- **prompt_directive:** "If the customer wants their replacement sent to the address already on file, you may arrange that directly — no additional gate is needed beyond the identity verification already completed for this interaction."

---

## Definitions

See `../../regulatory-landscape.md` and `../../technical-standards.md` for PAN, liability cap, and step-up verification definitions — not repeated here.

## Sources

All regulatory sources for this usecase live at the vertical layer — see `../../regulatory-landscape.md` (`BCM-REG-004`, `BCM-REG-006`) and `../../technical-standards.md` (`BCM-TECH-001`, `BCM-TECH-003`) for citations. This file introduces no new regulatory sources of its own.
