# Pack Template

Copy this directory to `knowledge-base/packs/<your-pack-id>/` to bootstrap a new vertical (e.g. `finance-card-servicing`, `finance-kyc`, `healthcare-prior-auth`). Every file here mirrors the shape of `knowledge-base/packs/finance-aml/` — treat that pack as the worked example of "what good looks like" when filling these in, not just this template's placeholder text.

## Steps to bootstrap a new pack

1. **Rename the directory** to your `pack_id` (kebab-case, e.g. `finance-kyc`).
2. **Fill `pack-manifest.yaml` first** — `pack_id`, governance floors, `jurisdictions_supported`, and an honest `open_gaps` list. Don't skip `open_gaps` — if you're only covering one jurisdiction or one product line, say so explicitly rather than letting a validator silently assume full coverage.
3. **Research before writing `regime-rules.md`.** Do not write regulatory content from memory alone — use web search to find authoritative sources (regulator websites, official examination manuals, statute text), and cite every requirement inline with a real URL and a short direct quote, the same way `finance-aml/regime-rules.md` does. Put a disclaimer at the top of the file matching `finance-aml`'s: illustrative, not legal advice, verify before production use.
4. **Write `controls-catalog.md`/`.json` next**, mapping the plugin's four guardrail categories (`pii_redaction`, `prompt_injection_detection`, `escalation_policy`, `human_approval_required`) to your domain's specific behavior, with every control tracing back to a `regime-rules.md` requirement ID via `maps_to_regime_rule`. Mark any control enforcing a hard legal prohibition (not just a risk-tradeoff) as `non_negotiable: true`, following `finance-aml`'s pattern for its approval-gate and tipping-off-equivalent controls.
5. **Write `allowlist.md`/`.json` last** — one connector per category of external system the agent needs, each scoped to exactly one file path (`module_path_pattern`) so `egress_gate.py` can enforce it. Use placeholder host patterns (e.g. `*.your-system.internal.example`) and say so explicitly, exactly as `finance-aml/allowlist.md` does — do not invent real-looking endpoints.
6. **Run `scripts/validate_pack.py`** (once it exists) against your new pack directory before considering it done — it checks structural consistency (every `.md`/`.json` pair matches, every cross-referenced ID actually exists) independent of whether the content itself is regulatorily correct.

## What a pack may and may not do

Per `knowledge-base/core/governance-schema.md`'s "Notes for pack authors" section, a pack may:
- set a `risk_tier` floor (never below what the domain actually warrants),
- set a `data_classification` floor,
- restrict `jurisdictions` to the subset the pack's content actually covers,
- extend the `pii_data` taxonomy with domain-specific categories (added to `core/pii-taxonomy.md`, not redefined locally).

A pack may never remove/redefine a core `GOV-###` field, add new top-level governance fields, or weaken a core validation rule.

## File checklist

- [ ] `pack-manifest.yaml`
- [ ] `regime-rules.md` (with real citations, disclaimer, and a "Sources" section)
- [ ] `controls-catalog.md` + `controls-catalog.json`
- [ ] `allowlist.md` + `allowlist.json`
