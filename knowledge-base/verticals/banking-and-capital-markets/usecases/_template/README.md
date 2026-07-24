# Usecase Template — Banking & Capital Markets

Copy this directory to `usecases/<your-usecase-id>/` to bootstrap a new use case within this vertical (e.g. `aml-transaction-monitoring`, `kyc-onboarding`, `lending-decisioning`). Treat `usecases/card-blocking-servicing/` as the worked example of "what good looks like" — thin, referencing the vertical layer by ID, not restating it.

## Steps to bootstrap a new usecase

1. **Rename the directory** to your `usecase_id`.
2. **Fill `usecase-manifest.yaml` first** — `usecase_id`, `governance_floors` (set honestly — don't just copy `card-blocking-servicing`'s Medium if your usecase actually warrants High, e.g. anything touching a regulatory filing), and `depends_on.vertical_files` listing which `../../regulatory-landscape.md` / `../../technical-standards.md` / `../../model-catalog.md` entries you'll reference.
3. **Check `../../regulatory-landscape.md` and `../../technical-standards.md` first**, before researching anything new — if your usecase needs a regulation already cited there (e.g. `BCM-REG-003` for BSA/AML general provisions), reference it by ID rather than re-deriving it. Only research and add *new* vertical-wide content if it's genuinely missing and genuinely vertical-wide (applies beyond just your usecase) — if it's specific to your usecase, it belongs in your own `usecase-rules.md` instead.
4. **Write `usecase-rules.md`** — requirement IDs prefixed with your own short code (not `CARD-UC-`), each with a `references` field pointing to vertical-layer IDs where applicable, and only usecase-specific regulatory citations where the vertical layer genuinely doesn't cover it. Disclaimer block at the top, "Sources" section at the bottom (even if it just says "see vertical layer" for everything).
5. **Write `controls-catalog.md`/`.json`** — map the four guardrail categories to your usecase's specific behavior, tracing back to your `usecase-rules.md` IDs via `maps_to_usecase_rule`.
6. **Write `allowlist.md`/`.json`** — one connector per external system category, each scoped to one file path. If a connector needs model-gateway access, reference `../../model-catalog.json` for eligibility the same way `card-blocking-servicing/allowlist.md` does.

## What NOT to do

- Don't restate a vertical-wide regulation's full citation/summary in your `usecase-rules.md` — reference its `BCM-REG-###`/`BCM-TECH-###` ID instead. If you find yourself writing more than a sentence about a regulation the vertical layer already covers, stop and just cite the ID.
- Don't invent a new top-level vertical file — if you think something is genuinely vertical-wide and missing, propose adding it to `../../regulatory-landscape.md` etc. directly, don't duplicate it inside your usecase folder.

## File checklist

- [ ] `usecase-manifest.yaml`
- [ ] `usecase-rules.md` (with real citations where usecase-specific, ID-references where not)
- [ ] `controls-catalog.md` + `controls-catalog.json`
- [ ] `allowlist.md` + `allowlist.json`
