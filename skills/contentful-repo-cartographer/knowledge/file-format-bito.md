# Bito Config — File Format Reference

`.bito.yaml` and `.bito/guidelines/` configure Bito's automated PR review for this repo.

## File Structure

```
.bito.yaml                              # Top-level config
.bito/
└── guidelines/
    ├── review-posture.txt              # How to review PRs here
    ├── repo-truth-and-boundaries.txt   # Use docs as review context
    └── domain-invariants.txt           # Repo-specific rules
```

## `.bito.yaml`

```yaml
suggestion_mode: comprehensive
post_description: true
post_changelist: true
exclude_files: '<lockfile-name>'
exclude_draft_pr: false
secret_scanner_feedback: true
linters_feedback: true
repo_level_guidelines_enabled: true
sequence_diagram_enabled: true
custom_guidelines:
  general:
    - name: 'Review Posture'
      path: './.bito/guidelines/review-posture.txt'
    - name: 'Repo Truth And Alignment'
      path: './.bito/guidelines/repo-truth-and-boundaries.txt'
    - name: 'Domain Invariants'
      path: './.bito/guidelines/domain-invariants.txt'
```

Adapt `exclude_files` to the repo's lockfile (e.g., `pnpm-lock.yaml`, `package-lock.json`).

## Guideline Files

### `review-posture.txt`

How to review PRs in this repo:

- What role to take ("tech lead of X project")
- Priorities (behavior > style, contracts > process)
- How to frame feedback (actionable, explain why, clear next step)
- When to say "no issues found"

### `repo-truth-and-boundaries.txt`

Use Golden Context docs as review context:

- Reference README, ARCHITECTURE, AGENTS, CONTRIBUTING, ADRs
- Flag code/doc mismatches
- Reference ADRs for architecture-significant changes
- Distinguish public API from internal implementation

### `domain-invariants.txt`

Repo-specific technical rules:

- Sharp edges discovered during analysis (mirror AGENTS.md invariants)
- Patterns that must be followed
- Things that commonly go wrong
- Integration boundary rules

## Rules

- **Baseline structure from experience-design-system-sdk.** Keep the three-file guideline structure; customize content per repo.
- **Guideline content is repo-specific.** Never copy verbatim from another repo — write content based on analysis findings.
- **domain-invariants.txt should mirror AGENTS.md sharp edges** — these are the same rules, expressed for Bito's review context.
