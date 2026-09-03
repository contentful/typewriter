# ADR Lifecycle

Use this file when deciding whether to create, amend, or supersede an ADR.

## Default Model

Treat ADRs as historical records: once accepted, the decision itself is not
rewritten in place. Only cosmetic or minor edits happen in place; anything
that changes the decision gets a new, superseding ADR.

Default status flow:

`proposed -> accepted -> superseded`

`deprecated` and `rejected` are optional terminal states — see Status
Guidance below.

## Choose The Right Action

Create a new ADR when:

- the decision is materially new
- the repo has no ADR covering this choice
- the old ADR is related but does not actually decide the same thing

Amend an existing ADR only when the change is cosmetic or minor and does not
alter the decision itself, for example:

- fixing typos, links, or formatting
- clarifying wording without changing what was decided
- correcting a factual error that doesn't change the outcome

Supersede an older ADR whenever the actual decision changes, even partially:

- the prior decision is being replaced or materially revised
- readers need both the old and new rationale preserved

## Status Guidance

- `proposed`: draft or pending confirmation
- `accepted`: confirmed and current
- `superseded`: replaced by a later ADR
- `deprecated` or `rejected`: only if the repo already uses these states

## Superseding Rules

- Cite the older ADR explicitly.
- Say what changed and why the old decision no longer fits.
- Update the old ADR's own status field to `superseded by ADR-XXX`, matching
  the repo's existing status-field casing and format.
- Update any ADR index or nav surface that marks status.
- Do not silently rewrite history inside the old ADR unless the repo already
  does that.
