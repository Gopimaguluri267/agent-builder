# Vertical Template

Copy this directory to `knowledge-base/verticals/<your-vertical-id>/` to bootstrap an entirely new industry vertical (e.g. `healthcare`, `retail`, `government`) — not a new use case within an existing vertical (for that, see `banking-and-capital-markets/usecases/_template/` instead).

## Steps to bootstrap a new vertical

1. **Rename the directory** to your `vertical_id` (kebab-case). Use the correct industry-standard term for the vertical, not an invented one — research how the industry itself names this sub-segment before committing to a directory name (see `ignore_dir/research-sources.md` §F for how "Banking and Capital Markets" was verified against real industry taxonomy, not assumed).
2. **Fill `vertical-manifest.yaml` first** — `vertical_id`, `primary_regulators`, and an honest `open_gaps` list. It's fine to ship with `status: placeholder` and empty `files: {}` if you're not ready to fully build it out yet — see `../wealth-and-asset-management/`, `../insurance/`, `../payments/` for examples of that intermediate state.
3. **Research before writing `regulatory-landscape.md`.** Same standard as `../banking-and-capital-markets/regulatory-landscape.md`: real citations with live URLs, a disclaimer block, ID-prefixed entries (`<PREFIX>-REG-###`), a "Sources" section. Only include regulations that are genuinely vertical-wide — if something only applies to one specific usecase, it belongs in that usecase's own `usecase-rules.md` instead.
4. **Write `model-risk-management.md`** if this vertical has its own regulator-specific model-governance mandate (most regulated industries do, in some form) — cite the current guidance, not an outdated version (check whether your regulator has updated guidance the way the 2026 Revised Guidance superseded SR 11-7/OCC 2011-12 for banking).
5. **Write `technical-standards.md`** for any vertical-wide technical requirements (data handling, encryption, industry-specific data formats).
6. **Write `model-catalog.md`/`.json`** — which models/providers are eligible for this vertical given its typical data sensitivity, with the same strong "verify with vendor, this is a snapshot" disclaimer `../banking-and-capital-markets/model-catalog.md` uses.
7. **Create `usecases/_template/`** inside your new vertical, modeled on `../banking-and-capital-markets/usecases/_template/`.

## What NOT to do

- Don't put anything genuinely cross-industry here (NIST AI RMF, ISO 42001, OWASP lists) — that belongs in `knowledge-base/core/ai-governance-frameworks.md`.
- Don't skip the research step and write regulatory content from memory — every claim needs a real, checkable citation.

## File checklist

- [ ] `vertical-manifest.yaml`
- [ ] `regulatory-landscape.md` (or `status: placeholder` in the manifest if deferring)
- [ ] `model-risk-management.md`
- [ ] `technical-standards.md`
- [ ] `model-catalog.md` + `model-catalog.json`
- [ ] `usecases/_template/`
