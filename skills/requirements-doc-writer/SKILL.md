---
name: requirements-doc-writer
description: Generates the Business Requirements Document, Product Requirements Document, and Functional Requirements Document for a project run, one at a time, each followed by a captured sign-off, using validated requirements data. Use only after requirements-validator has returned a ship or ship_with_conditions verdict.
when_to_use: When forge.md's orchestration reaches document generation, or when the user asks to (re)generate the BRD/PRD/FRD for an already-validated set of requirements.
allowed-tools: ["Read", "Write", "Bash(shasum *)", "AskUserQuestion", "Skill"]
---

You generate the three formal requirements documents and capture their sign-offs. You do not gather requirements yourself and you do not decide whether the requirements are acceptable — both of those already happened (`skills/requirements-intake/`, `skills/requirements-validator/`). If you're invoked and the latest `validation-report.md` verdict is `block` or `hold`, **stop immediately** and say so — do not generate documents from unvalidated or conflicting requirements.

This skill runs inline in the current conversation (not as an isolated subagent) specifically so the sign-off step below can actually have a live back-and-forth with the person in the session — that interactivity is the whole point of the sign-off, so don't let this logic get moved somewhere that can't guarantee it.

## Inputs

- `.agentbuilder/<run_id>/state.md` — read this **first**, not last. See "Resume support" below — it's what tells you whether this is a fresh run through all three documents or a continuation after an earlier interruption.
- `.agentbuilder/<run_id>/requirements/requirements-intake.yaml`
- `.agentbuilder/<run_id>/requirements/validation-report.md` (must show `ship` or `ship_with_conditions`)
- `.agentbuilder/<run_id>/governance/scope-bind.yaml`
- `core/documentation-standards.md` (`DOC-STD-001`/`002`/`003`/`004`/`005` — the section lists, document-control/sign-off requirements, **and the content depth standard — read `DOC-STD-005` before writing a single word, not after**)
- The usecase's `usecase-rules.md`/`controls-catalog.json` and the vertical's `regulatory-landscape.md`/`technical-standards.md` — for pulling real regulatory reference IDs into each document's control block, and for cross-referencing feature/rule content to real IDs (never invented ones)

## Resume support — check before starting the sequence below

Read `state.md`'s decision log for this run and look for `[human] Signed off <type>` entries. Three documents, three possible prior-sign-off states — **don't assume you're starting fresh**:

- No `Signed off` entries at all → start the sequence at BRD, as normal.
- `Signed off brd` present but not `prd`/`frd` → BRD is done; skip it entirely (do not regenerate or re-request sign-off — a signed document is a completed, immutable record for this run unless the user explicitly asks to revise it) and start the sequence at PRD.
- `Signed off brd` and `Signed off prd` present but not `frd` → start at FRD.
- All three present → all documents for this run are already signed. Say so, don't regenerate anything, and report the same way you would after freshly finishing (see "After all three are signed").

Also verify the corresponding `.html` file on disk actually shows `status: signed` before trusting the log entry — if the log says signed but the file is missing or still shows `pending_signoff`, treat that as a discrepancy and tell the user rather than silently picking one source of truth over the other.

## Content quality is not optional

`DOC-STD-005` exists because an earlier rendering test used one-liner placeholder content and it was easy to mistake that for an acceptable output. It is not. Before writing any section, know which category it falls into: **brief-by-convention** (Executive Summary, Objective, Problem Statement — a few sentences, don't pad) or **needs real substance** (nearly everything else — Scope, Assumptions, Constraints, Risks, Success Criteria, KPIs, Personas, Feature Requirements, Out-of-Scope, Dependencies, Open Questions, Use Cases, Workflows, Validation Rules, Data Requirements). For the second category, every list item is a complete sentence explaining *why it matters*, not a fragment — `DOC-STD-005`'s worked example (a properly-detailed Feature Requirement entry) is the concrete bar to match, not just a suggestion.

If `requirements-intake.yaml` doesn't have enough captured detail to write a substantive version of some section, that is itself a signal: say so explicitly rather than padding with generic filler to hit a word count, and consider whether this should have been caught as a gap by `requirements-validator`'s report — note it in the document's Open Questions rather than papering over it.

## Sequence — one document at a time, sign-off before moving to the next

For **each** remaining document, in BRD → PRD → FRD order, starting from wherever "Resume support" above determined (not necessarily BRD):

1. **Compose the content** for that document's sections per `core/documentation-standards.md`'s `DOC-STD-00{1,2,3}`, at the depth `DOC-STD-005` requires, drawing only from `requirements-intake.yaml` and the KB — do not invent business content that wasn't captured during intake, but do expand what *was* captured into full, specific sentences rather than transcribing it verbatim as a fragment. If `validation-report.md` listed conditions (from a `ship_with_conditions` verdict), they must appear verbatim in the PRD's Open Questions section (or the BRD/FRD equivalent, if more relevant there) — do not drop them because they're inconvenient.
1a. **Self-check before rendering**: reread what you just composed. Does every substance-required section have multi-sentence, specific content? Is any list item under ~8 words with no elaboration? Is anything generic filler that could apply to any project? If any of these are true, rewrite that section before proceeding — do not rely on the human reviewer at sign-off to catch thin content.
2. **Render in the `pending_signoff` state**: invoke `skills/document-rendering` via the `Skill` tool with the composed content and the matching `templates/documents/<type>.html.template` — `status: pending_signoff`, Sign-Off section shows "Pending" for all fields. Write to `.agentbuilder/<run_id>/requirements/<type>.html`.
3. **Compute the pre-signature content hash**: `shasum -a 256 .agentbuilder/<run_id>/requirements/<type>.html`. This is the evidence hash — it must be computed on the document *as the approver is about to see it*, before any signature fields are filled in.
4. **Present the document to the user and request sign-off**, following `core/approval-policy.md`'s pattern exactly — no silent proceed:
   - Tell them what document this is, where it's written, and that you need their explicit sign-off to continue.
   - Ask directly (not via `AskUserQuestion`, since this needs a real name, not a menu pick) for their **name** and **role**.
   - Ask for **explicit affirmative consent** — e.g. "Do you approve this document as documented? (yes/no)" — a non-answer, a maybe, or silence is not consent. If they decline, this is a `block`: do not sign, ask what needs to change, and expect to loop back to step 1 for this document (not the other two) with the correction.
5. **On explicit approval**: invoke `skills/document-rendering` again to re-render the document with `status: signed` and the Sign-Off section filled in — `approver_name`, `approver_role`, `signed_at` (current timestamp), `evidence_hash` (from step 3, computed *before* this re-render). Overwrite `.agentbuilder/<run_id>/requirements/<type>.html` with the signed version.
6. **Log to `state.md`**: `[human] Signed off <type> — approver: <name> (<role>), evidence_hash: <hash>` — same actor/timestamp/case_id(run_id)/evidence_hash schema as every other approval gate in this plugin.
7. Move to the next document. Do not batch all three sign-off requests together — each is sequential, per the plan's explicit "per-document, immediately after generated" decision.

## After all three are signed

Update `state.md`'s run entry: set `pipeline_step: requirements-complete` for this run (do not mark the run's overall `status` as `complete` — later pipeline steps, architecture/prompts/codegen, are still not built). Tell the user all three documents are signed, where they are, and that architecture design is the next not-yet-built pipeline step.

## What you must never do

- Never fabricate an approver's name or mark a document signed without an actual explicit "yes" captured in this conversation.
- Never invent a regulatory citation or KB ID that doesn't exist in the actual files you read.
- Never proceed to the next document if the current one's sign-off was declined — surface the correction loop instead.
- Never regenerate the evidence hash *after* filling in the signature fields — it must reflect what the approver actually reviewed, not the final signed artifact (that would make the hash meaningless as evidence of what was approved).
