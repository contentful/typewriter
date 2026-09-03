---
name: create-architectural-decision-record
description: Draft an Architectural Decision Record (ADR) with a clear decision, options, rationale, tradeoffs, and consequences. Use when capturing an architecture-significant decision, deciding whether to create a new ADR or supersede an older one, and producing a clean ADR from repository context and available decision evidence.
---

# Create Architectural Decision Record

Draft exactly one ADR that captures one architecture-significant decision.

Prefer the target repository's existing ADR location and document style when
one exists. If the repo has no clear ADR convention, use
[assets/adr-template.md](assets/adr-template.md).

## Workflow

1. Inspect the repository's ADR conventions first.
   - Look for existing ADR files, templates, storage locations, frontmatter, status values, section order, and index or nav pages.
   - If the repo already has a stable ADR location and document format, follow them.
   - If conventions are ambiguous, state the assumption and use the fallback template.

2. Decide whether the task needs a new ADR, an amendment, or a superseding ADR.
   - Read [references/adr-lifecycle.md](references/adr-lifecycle.md).
   - Create a new ADR for a materially new decision.
   - Amend only for cosmetic or minor changes that don't alter the decision itself.
   - Supersede whenever the decision itself changes, even partially.

3. Gather the minimum decision context before writing.
   - Infer from, in order: the open PR/branch description and linked ticket,
     the branch's commits and diff, related code and config, and any
     existing ADRs or docs covering adjacent decisions.
   - If a source gives a weak or partial signal, use it but mark it as an
     assumption in the draft (see Output Contract) rather than asking.
   - Ask the user only for whichever specific piece is still completely
     unresolved after checking those sources.
   - problem statement and scope
   - constraints and decision drivers
   - realistic options under consideration
   - consequences, risks, and follow-up constraints

4. Make options explicit before finalizing the decision.
   - Capture only materially distinct options.
   - Summarize the tradeoffs of each option.
   - Do not jump straight from context to solution.
   - Read [references/adr-writing.md](references/adr-writing.md), including its
     stable-versus-volatile guidance, before drafting or reviewing the body.
   - Classify each named implementation detail as decision-relevant or
     replaceable. Keep its exact name only when its identity or exact value
     drives the decision or materially changes behavior. Otherwise describe
     the durable capability or constraint instead and link the operational
     source of truth.

5. Write and store the ADR.
   - Store every new or superseding ADR as
     `YYYY-MM-DD-HHMM-<slug>.md`, using the UTC creation date, a zero-padded
     24-hour creation time, and a concise lowercase kebab-case decision slug.
     For example: `2026-08-27-1420-authorize-destination-routes.md`.
   - If an ADR already uses the same UTC minute, append `-2`, `-3`, and so on
     to the slug rather than reusing the filename.
   - Keep an amended ADR at its existing path and filename.
   - Keep it to one decision.
   - State the chosen option clearly.
   - Include rationale, tradeoffs, and consequences.
   - Mark unknowns explicitly instead of inventing facts.
   - Add revisit conditions when the decision should be re-evaluated under specific future conditions.
   - Add follow-up constraints when downstream work must respect this decision.

6. Maintain the repository's ADR surface.
   - If the repo keeps an ADR index, summary page, or docs navigation, update it after adding a new ADR.
   - If the task needs common maintenance patterns, read [references/adr-maintenance.md](references/adr-maintenance.md).

7. Run relevant validation after writing.
   - Run docs generation, linting, or link checks that already exist in the repo and are relevant to the changed ADR files.

## Output Contract

- Produce exactly one ADR unless the user explicitly asks for multiple ADR documents.
- Keep the document implementation-oriented and reviewable.
- Do not invent decision history or rationale.
- If evidence is incomplete, say what is unknown.
- The ADR should remain valid when replaceable implementation names change;
  retain exact names only as decision-relevant evidence or clearly dated
  implementation history.
- Do not add routine source-file paths, line numbers, function names, or other
  code pointers to the ADR. Include a code pointer only when it is needed to
  support the decision or preserve auditable historical evidence.

## Quality Bar

The final ADR should make these questions easy to answer:

- What problem are we solving?
- What options were considered?
- Why was this option chosen?
- What tradeoffs were accepted?
- What changes or constraints follow from this decision?
