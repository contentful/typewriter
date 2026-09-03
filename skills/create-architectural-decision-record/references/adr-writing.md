# ADR Writing

Use this file when drafting the ADR body or evaluating whether the draft is
reviewable.

## Required Substance

Every ADR should make these points obvious:

- what problem exists
- what constraints matter
- what options were seriously considered
- what was chosen
- why that option won
- what consequences follow

## Option Handling

- Include only materially distinct options.
- Merge trivial variations into one option when the tradeoff is the same.
- State both benefits and costs.
- Reject hand-wavy options like "keep things flexible" unless they map to a
  concrete technical approach.

## Decision Language

- Prefer direct phrasing: "Use X for Y because Z."
- Explain why the chosen option beats the real alternatives.
- Mark unknowns explicitly.
- Avoid fake certainty when the evidence is partial.

## Stable Decisions vs Volatile Implementation

- State the enduring decision in terms of responsibilities, capabilities,
  constraints, interfaces, and budgets.
- Classify named implementation details as decision-relevant or replaceable.
  Keep an exact name when its identity or exact value is a decision driver or
  materially changes behavior. Otherwise use capability language, such as deep
  cross-file reasoning, broad coverage, or low-latency merging.
- Keep changing rosters, aliases, versions, and endpoints in operational
  configuration or runbooks, and link to those sources from the ADR.
- Preserve concrete names, dates, and measurements when they are needed to
  make historical evidence auditable; label them as evidence or dated
  implementation history rather than as the enduring decision.

## Source Pointers

- Do not add routine source-file paths, line numbers, function names, or other
  code pointers to an ADR.
- Include a code pointer only when it is needed to support the decision or
  preserve auditable historical evidence, and label that role clearly.
- Prefer linking to the operational source of truth for changing
  implementation details.

## Consequence Language

Consequences should cover what the decision:

- enables
- constrains
- makes more expensive
- requires as follow-up work

## Avoid AI-Sounding Language

- Cut inflated words: "leverage," "robust," "seamless," "delve," "game-changer," "testament to."
- Cut hedge-and-transition filler: "in order to," "moreover," "it's important to note that."
- Avoid the "It's not X — it's Y" construction and other rhetorical templates.
- Prefer plain verbs and concrete specifics (names, numbers, dates) over vague intensifiers.
- If a word could be deleted without losing meaning, delete it.

## Quality Checks

Before finishing, verify:

- one ADR, one decision
- title matches the actual decision
- rationale is not just restated context
- consequences are concrete, not generic
- follow-up constraints are visible if downstream work must obey them
