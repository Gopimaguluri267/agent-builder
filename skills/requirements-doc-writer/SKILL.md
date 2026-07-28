---
name: requirements-doc-writer
description: Generates the Business Requirements Document, Product Requirements Document, and Functional Requirements Document for a project run, one at a time, each followed by a captured multi-approver sign-off, using validated requirements data. Use only after requirements-validator has returned a ship or ship_with_conditions verdict.
when_to_use: When forge.md's orchestration reaches document generation, or when the user asks to (re)generate the BRD/PRD/FRD for an already-validated set of requirements.
allowed-tools: ["Read", "Write", "Bash(shasum *)", "AskUserQuestion", "Skill"]
---

You generate the three formal requirements documents and capture their sign-offs. You do not gather requirements yourself and you do not decide whether the requirements are acceptable — both of those already happened (`skills/requirements-intake/`, `skills/requirements-validator/`). If you're invoked and the latest `validation-report.md` verdict is `block` or `hold`, **stop immediately** and say so — do not generate documents from unvalidated or conflicting requirements.

This skill runs inline in the current conversation (not as an isolated subagent) specifically so the sign-off step below can actually have a live back-and-forth with the person in the session — that interactivity is the whole point of the sign-off, so don't let this logic get moved somewhere that can't guarantee it.

## Inputs

- `.agentbuilder/<run_id>/state.md` — read this **first**, not last. See "Resume support" below — it's what tells you whether this is a fresh run through all three documents (and all their approvers) or a continuation after an earlier interruption.
- `.agentbuilder/<run_id>/requirements/requirements-intake.yaml`
- `.agentbuilder/<run_id>/requirements/validation-report.md` (must show `ship` or `ship_with_conditions`)
- `.agentbuilder/<run_id>/governance/scope-bind.yaml`
- `core/documentation-standards.md` (`DOC-STD-001`–`005` — the section lists, the multi-approver document-control/sign-off model, **and the content depth standard — read `DOC-STD-005` before writing a single word, not after**)
- The usecase's `usecase-rules.md`/`controls-catalog.json` and the vertical's `regulatory-landscape.md`/`technical-standards.md` — for pulling real regulatory reference IDs into each document's control block, and for cross-referencing feature/rule content to real IDs (never invented ones)

## Resume support — check before starting the sequence below

Read `state.md`'s decision log for this run. Sign-off is now per-approver, not per-document, so resumability needs two levels, not one:

1. **Which documents are fully signed** — look for `[human] Signed off <type>` entries logged only once *every* required approver for that type has signed (see Step 6 below). A fully-signed document is skipped entirely; do not regenerate or re-request sign-off on it (a signed document is a completed, immutable record for this run unless the user explicitly asks to revise it).
2. **Within the first not-yet-fully-signed document, which approvers already signed** — look for `[human] Signed off <type> — approver: <name> (<role>)` entries scoped to that document. Skip asking those specific roles again; only request sign-off from the roles in `DOC-STD-004`'s required list for this document type that don't yet have an entry.

Also verify the document's `.html` file on disk actually shows the expected number of signed rows in its Sign-Off table before trusting the log — if the log shows more or fewer signatures than the file, treat that as a discrepancy and tell the user rather than silently picking one source of truth over the other.

## Content quality is not optional

`DOC-STD-005` exists because an earlier rendering test used one-liner placeholder content and it was easy to mistake that for an acceptable output. It is not. Before writing any section, know which category it falls into: **brief-by-convention** (Executive Summary, Problem Statement — a few sentences, don't pad) or **needs real substance** (nearly everything else, including the newer sections — Business Objectives, Stakeholders, Current State Analysis, Assumptions, Constraints, Risks, Success Criteria, KPIs, Personas, Feature Requirements, Out-of-Scope, Dependencies, Open Questions, Use Cases with alternative/exception flows, Business Rules, Validation Rules, Workflows, Data Requirements, Security Requirements, Test Scenarios, Traceability Matrix). For the second category, every list/table row is complete and specific, explaining *why it matters*, not a fragment — `DOC-STD-005`'s worked examples (a Feature Requirement entry, a Business Objective row, a Traceability Matrix row) are the concrete bar to match, not just a suggestion.

If `requirements-intake.yaml` doesn't have enough captured detail to write a substantive version of some section, that is itself a signal: say so explicitly rather than padding with generic filler to hit a word count, and consider whether this should have been caught as a gap by `requirements-validator`'s report — note it in the document's Open Questions (or the Traceability Matrix, if it's a downstream tracing gap) rather than papering over it.

## Sequence — one document at a time, all its approvers before moving to the next

For **each** remaining document, in BRD → PRD → FRD order, starting from wherever "Resume support" above determined:

1. **Compose the content** for that document's sections per `core/documentation-standards.md`'s `DOC-STD-00{1,2,3}`, at the depth `DOC-STD-005` requires, drawing only from `requirements-intake.yaml` and the KB — do not invent business content that wasn't captured during intake, but do expand what *was* captured into full, specific sentences rather than transcribing it verbatim as a fragment. If `validation-report.md` listed conditions (from a `ship_with_conditions` verdict), they must appear verbatim in the PRD's Open Questions section (or the BRD/FRD equivalent, if more relevant there) — do not drop them because they're inconvenient. For the FRD's Traceability Matrix specifically, build it from the BRD's Business Objectives and this document's own Use Cases/Business Rules/Validation Rules/Test Scenarios — every BRD objective needs a row; a gap here is a real gap, state it rather than leaving a blank cell.
1a. **Self-check before rendering**: reread what you just composed. Does every substance-required section have multi-sentence, specific content? Is any list/table row under ~8 words with no elaboration? Is anything generic filler that could apply to any project? Does every Traceability Matrix row have something in every column (or an explicit "not yet determined" rather than blank)? If any of these are true, rewrite before proceeding — do not rely on the human reviewer at sign-off to catch thin content.
2. **Determine the required approver roles** for this document type from `DOC-STD-004` (BRD: Executive Sponsor, Product Owner, Business Owner, Technical Lead/Architect, Compliance/Security Representative; PRD: Product Owner, Business Owner, Technical Lead/Architect; FRD: Technical Lead/Architect, QA Lead, Compliance/Security Representative), minus whichever are already signed per "Resume support" above.
3. **Render in the `pending_signoff` state**: invoke `skills/document-rendering` via the `Skill` tool with the composed content, the required-approver-role list (rendered as pending rows in the Sign-Off table, plus any already-signed rows from a prior partial run), and the matching `templates/documents/<type>.html.template` — `status: pending_signoff` unless every role is already signed. Write to `.agentbuilder/<run_id>/requirements/<type>.html`.
4. **Compute the pre-signature content hash** (only if not already computed in a prior partial run for this document — reuse the existing hash if one was already logged for this document, since it must represent what the *first* approver reviewed, not be recomputed per approver): `shasum -a 256 .agentbuilder/<run_id>/requirements/<type>.html`. This is the evidence hash, shared by every approver of this document — it must be computed on the document as approvers are about to see it, before any signature is filled in.
5. **For each required approver role, in turn**: present the document (or remind them it's already open, if this is the second+ approver in the same sitting) and request that specific role's sign-off, following `core/approval-policy.md`'s pattern exactly — no silent proceed:
   - Tell them which role you need sign-off from next (e.g. "Now I need sign-off from the Technical Lead/Architect").
   - Ask directly (not via `AskUserQuestion`, since this needs a real name, not a menu pick) for that approver's **name**.
   - Ask for **explicit affirmative consent** — e.g. "Do you approve this document as documented, in your capacity as [role]? (yes/no)" — a non-answer, a maybe, or silence is not consent. If they decline, this is a `block` for the whole document: do not sign any remaining roles either, ask what needs to change, and expect to loop back to step 1 with the correction (previously-signed roles on this same document stay signed only if the correction doesn't materially change what they already approved — if it does, their sign-off must be re-obtained too; use judgment and say explicitly which approvers need to re-review).
   - On explicit approval, invoke `skills/document-rendering` again to re-render with this role's row filled in (`approver_name`, `approver_role`, `signed_at`, the shared `evidence_hash`) and log to `state.md` immediately: `[human] Signed off <type> — approver: <name> (<role>), evidence_hash: <hash>` — do not batch all approvers' log entries until the end, log each as it happens (same per-item-immediately discipline used everywhere else in this plugin).
6. **Once every required role for this document has signed**: re-render one final time with `status: signed`, and log `[ai] <type> fully signed — all required approvers complete` to `state.md`. This combined entry (not the individual per-approver ones) is what "Resume support" checks to know a document is fully done.
7. Move to the next document. Do not start the next document's content composition until the current one's sign-offs are fully complete.

## After all three are signed

Update `state.md`'s run entry: set `pipeline_step: requirements-complete` for this run (do not mark the run's overall `status` as `complete` — later pipeline steps, architecture/prompts/codegen, are still not built). Tell the user all three documents are signed, where they are, and that architecture design is the next not-yet-built pipeline step.

## What you must never do

- Never fabricate an approver's name or mark any role's row signed without an actual explicit "yes" from that specific role captured in this conversation.
- Never invent a regulatory citation or KB ID that doesn't exist in the actual files you read.
- Never proceed to the next document until every required approver role on the current one has signed.
- Never proceed to the next approver if the current one declined — surface the correction loop instead.
- Never recompute the evidence hash per approver — it's one hash per document, computed once before the first signature, representing what every approver reviewed.
