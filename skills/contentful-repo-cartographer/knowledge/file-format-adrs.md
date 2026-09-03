# Architecture Decision Records — File Format Reference

> **Scope: internal-only.** ADRs are never proposed to public repos — their sourcing rules require Glean/Confluence/Jira citations that cannot ship publicly. For private repos, they are proposed to the target repo's `docs/ADRs/` as usual.

ADRs live in `docs/ADRs/` in the target repo (private repos only). Each records a single architectural decision with its context and consequences.

## Directory Structure

```
docs/ADRs/
├── README.md                          # Index table
├── YYYY-MM-DD-short-title.md          # Individual ADR
├── YYYY-MM-DD-short-title.md
└── ...
```

## Index File (`docs/ADRs/README.md`)

```markdown
# Architecture Decision Records

| Date                                      | Status   | Title       |
| ----------------------------------------- | -------- | ----------- |
| [YYYY-MM-DD](./YYYY-MM-DD-short-title.md) | Accepted | Short title |
```

The date-based filename is the canonical identifier. Do not introduce a separate sequential numbering scheme — use the date as both filename prefix and index reference.

## Individual ADR Format

```markdown
# <Title>

## Status

Accepted | Superseded by [YYYY-MM-DD-title](./YYYY-MM-DD-title.md) | Deprecated

## Context

What problem or decision was faced? What constraints existed? What alternatives were considered?

## Decision

What was chosen and why. Be specific — link to evidence from git history, Slack discussions, RFCs, or Glean results where possible.

## Consequences

Trade-offs accepted. What this enables. What it prevents or makes harder. Any follow-up work created.
```

## Rules

- **Date from source.** Use the date of the relevant git commit, RFC, or Slack thread. Fall back to today only if no source is available.
- **Research-backed.** Git archaeology + Glean give the "why." Never fabricate reasoning. If you can't find the "why," say "Context not found — likely a default/inherited choice" in the Decision section.
- **One decision per ADR.** Don't bundle multiple decisions into one file.
- **Status is mandatory.** Most generated ADRs will be `Accepted`. Use `Superseded` if you find evidence the decision was later reversed.
- **Verify commit hashes.** Never transcribe commit hashes from memory. Always verify with `git log --oneline | grep '<first-5-chars>'` before writing a hash into an ADR. Character transposition errors (e.g., `a0a55db` vs `a0ad65b`) are common and undermine trust in the document.
