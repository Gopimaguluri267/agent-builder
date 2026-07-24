---
name: document-rendering
description: Renders a validated requirements payload (BRD, PRD, or FRD content) into a polished, self-contained, leadership-appropriate HTML document using the plugin's shared document templates and styling. Use when generating or updating any formal requirements document for a project run.
when_to_use: When skills/requirements-doc-writer needs to produce brd.html, prd.html, or frd.html, or when re-rendering a document after a sign-off is captured.
disable-model-invocation: false
---

# Document Rendering

You render structured requirements content into a finished HTML document. You are not responsible for deciding *what* goes in the document (that's `skills/requirements-doc-writer`'s job, informed by `skills/requirements-validator`'s output) — you're responsible for making sure it *looks like something a bank's leadership team would actually accept*, not a wall of unstyled text.

**You don't read `state.md`, and that's intentional, not an oversight.** Unlike the other skills in this pipeline, you have no pipeline position of your own to track — you're a stateless rendering step invoked synchronously by `requirements-doc-writer` with the full content and document type handed to you each call. It already knows the run, the resume state, and what's being rendered; re-deriving any of that here would just be redundant.

## Design discipline

The generated document isn't an interactive artifact — it's a formal, often-printed business record, closer to a PDF report than a dashboard. Adjust accordingly:

- **Self-contained.** One HTML file, `<link>` to nothing external, no CDN fonts, no remote images. `templates/documents/_shared/styles.css` is inlined into a `<style>` block at render time — never left as a separate linked file in the output, since the output must be portable (emailable, single-file, works offline).
- **Print-first, not just screen-first.** Leadership will often print or export to PDF. Respect the `@media print` rules already in `_shared/styles.css` — don't add screen-only decoration (drop shadows, background images, sticky headers) that breaks or looks wrong on paper.
- **Restrained color, purposeful when used.** The only color that should carry real meaning is the status/verdict badge system already defined in `_shared/styles.css` (`abd-status-*`, `abd-verdict-*` classes) — reuse those classes exactly, don't invent new colors for emphasis elsewhere in the document body.
- **Hierarchy over decoration.** Use the existing heading classes (`abd-title`, `abd-section`, `abd-subsection`) and spacing — don't add borders, boxes, or icons that aren't already part of the shared stylesheet's vocabulary. A document that looks the same, structurally, every time it's generated is a *feature* here (consistency across every generated project), not a limitation.
- **Every regulatory/KB reference renders as a visible, styled reference** (`.abd-ref` class or the control-block's `.abd-reg-refs code` styling) — never buried as an unstyled inline mention. A reader should be able to visually scan a document and see exactly which KB rules it's grounded in.
- **Open questions are visually distinct, not hidden.** Use `.abd-open-question` for anything from `skills/requirements-validator`'s output that's still unresolved — these must be impossible to miss, not styled the same as settled content.

This isn't the same design space as an interactive Claude Code Artifact or a data dashboard (see Claude Code's own `artifact-design`/`dataviz` skills for that different genre) — it's closer to a styled print report. Borrow the general discipline (self-contained, accessible contrast, no unnecessary flourish) but don't import dashboard/chart conventions (stat tiles, sparklines, interactive legends) that don't belong in a static compliance document.

## How to render

1. Identify the document type (`brd` / `prd` / `frd`) and load `templates/documents/<type>.html.template`.
2. Load `templates/documents/_shared/styles.css` and inline its full contents into the template's `<style>` block — do not reference it as an external file in the output.
3. Fill every `{{PLACEHOLDER}}` in the template with content supplied by the calling skill. **Never leave a placeholder unfilled or invent content to fill one** — if the calling skill didn't supply something a placeholder needs, that's a bug in the calling skill, stop and say so rather than fabricating filler text.
4. For the document-control block (`DOC-STD-004` in `core/documentation-standards.md`): render `regulatory_references` as a list of `.abd-ref`-styled IDs, each one a real ID that exists in the cited KB file — never a plausible-looking but fabricated ID.
5. For the Sign-Off section: if no sign-off has been captured yet, render it in the "unsigned" state (`abd-signoff-unsigned` class, fields showing "Pending" rather than blank or fabricated values) with `status: pending_signoff`. Once a real sign-off is captured (name, role, explicit affirmative consent, timestamp — see `core/approval-policy.md`), re-render with those exact values filled in and `status: signed`. Never pre-fill a name or timestamp before they're actually provided.
6. Write the finished HTML to the path the calling skill specifies (e.g. `.agentbuilder/<run_id>/requirements/brd.html`).

## What you must never do

- Never invent a regulatory citation, KB ID, or compliance claim that isn't present in the actual knowledge base files the calling skill supplied.
- Never mark a document `signed` without an actual captured name + explicit affirmative statement + timestamp passed in by the calling skill.
- Never restyle or redesign the shared template on your own initiative mid-project — if the styling genuinely needs to change, that's a change to `templates/documents/_shared/styles.css` itself (affecting all three document types consistently), not a one-off tweak to a single rendered document.
