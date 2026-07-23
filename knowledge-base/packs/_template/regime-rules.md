# Regime Rules — REPLACE_ME (pack_id)

> **Disclaimer:** REPLACE_ME — adapt this disclaimer to your domain, following `finance-aml/regime-rules.md`'s pattern: state what sources this is compressed from, when it was compiled, that it is not legal/compliance advice, and that specific figures/deadlines must be verified against current authoritative guidance before production use.

Covers **REPLACE_ME (jurisdiction(s))** only (see `pack-manifest.yaml` → `open_gaps` for what's not covered).

## Entry Format

Do not change this format without also updating `knowledge-base/core/` files and any script that parses it — this structure is shared across every pack.

- `requirement_id` — stable identifier (`<PACK-PREFIX>-R###`, e.g. `FIN-AML-R001` for finance-aml — pick a short, unique prefix for your pack)
- `requirement_text` — the rule itself, plain language
- `applies_when` — the condition that activates this requirement
- `requires_controls` — which `controls-catalog.md` control ID(s) enforce it
- `severity_floor` — minimum severity if violated (`blocker` / `high` / `medium`)
- `source` — real citation: statute/regulation name if applicable, a live hyperlink to the actual source, and a short direct quote — never a paraphrase presented as if it were the source text
- `prompt_directive` — text for `prompt-engineer` to lift into the generated agent's system prompt, phrased as a direct instruction to the agent

---

## <PREFIX>-R001 — REPLACE_ME (requirement name)

- **requirement_text:** REPLACE_ME
- **applies_when:** REPLACE_ME
- **requires_controls:** REPLACE_ME
- **severity_floor:** REPLACE_ME
- **source:** REPLACE_ME — must be a real, checkable citation, not an invented one
- **prompt_directive:** REPLACE_ME

*(repeat one entry per requirement — `finance-aml/regime-rules.md` has 6; your pack may need more or fewer depending on domain complexity)*

---

## Definitions (for context, not independently machine-checked)

REPLACE_ME — define domain-specific terms an LLM agent or reviewer would need, each with a citation, following `finance-aml/regime-rules.md`'s "Definitions" section.

## Sources

REPLACE_ME — list every unique URL/citation used across all requirements above, deduplicated, in one place.
