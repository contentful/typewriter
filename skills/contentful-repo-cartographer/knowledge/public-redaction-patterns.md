# Public Redaction Patterns

This file is the canonical pattern set for the public-bound redaction sweep. Phase 1 Step 6.5 and the equivalent step in `refresh.md` both source patterns from here. Edit this file (not the phase prompts) when a new leak vector is identified or when a false positive needs an exception.

## Scope

Patterns apply to files under `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/public-drafts/` only. Files under `internal-only/` and the research artifacts (`backstage-facts.md`, `decisions.md`, `GAPS.md`) are exempt — they are never proposed to the target repo.

## Sweep Behavior

The sweep is a **hard gate**. If any pattern matches, the cartographer must:

1. Rewrite the offending claim using a citation from the public-source allowlist (Step 6 of `phase-1-discovery.md`), OR
2. Move the claim to an internal-only artifact (`CLAUDE.md` or an ADR), OR
3. Delete the claim.

The sweep cannot be acknowledged-and-skipped.

## Patterns

| ID  | Category                                               | Regex                                                                                             | Action on match                                      |
| --- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| P1  | Email addresses                                        | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`                                                  | Strip; rewrite or delete                             |
| P2  | Internal URLs                                          | `(@contentful\.com\|atlassian\.net\|slack\.com/archives\|grafana\.\|datadoghq\.\|pagerduty\.com)` | Strip                                                |
| P3  | Jira keys                                              | `\b[A-Z]{2,8}-[0-9]+\b`                                                                           | Strip; rewrite from public source                    |
| P4  | Confluence numeric IDs                                 | `Confluence [0-9]+`                                                                               | Strip                                                |
| P5  | Bare long numeric IDs near Glean/Confluence references | `(Confluence\|Glean)[^[:space:]]{0,40}[0-9]{8,12}\b`                                              | Strip                                                |
| P6  | Slack channel names                                    | `#[a-z0-9][a-z0-9_-]{2,}`                                                                         | Strip (manual exempt list below)                     |
| P7  | Internal team handles outside CODEOWNERS               | `@contentful/team-[A-Za-z0-9_-]+`                                                                 | Strip if not present in `.github/CODEOWNERS`         |
| P8  | Internal hostnames                                     | `\.contentful\.tools\b`                                                                           | Strip                                                |
| P9  | Backstage internal hostnames                           | `backstage\.[A-Za-z0-9.-]*\.contentful`                                                           | Strip                                                |
| P10 | Glean citation markers                                 | `\[Source:[[:space:]]*Glean[^\]]*\]`                                                              | Strip; replace with public-source citation or delete |

### Pattern P6 — Slack channel false-positive exemptions

Some `#name` strings in code or docs are not Slack channels:

- C/C++ preprocessor directives: `#ifdef`, `#endif`, `#define`, `#include`, `#pragma`
- C#/IDE region markers: `#region`, `#endregion`
- CSS hex colors: e.g., `#abc123`, `#fff`, `#a0b0c0` — these match P6's regex but are not Slack channels. Treat as false positives during the sweep.
- Markdown anchor refs: `#some-heading` (a heading-style fragment in a URL)

When the cartographer reviews a P6 match, distinguish "this is a Slack channel reference" from these false positives. If it is a code construct, it stays. If it is a Slack channel, it goes.

### Pattern P7 — CODEOWNERS allowlist

Team handles already declared in the target repo's `.github/CODEOWNERS` are public information (CODEOWNERS is a public file in a public repo). Treat as allowed: any `@contentful/team-*` value that grep finds in `.github/CODEOWNERS`. Anything else gets stripped.

## Verification Command

```bash
# Run from your workspace root (the directory that contains <SCRATCH_ROOT>)
REPO=<repo-name>
SCRATCH="<SCRATCH_ROOT>/repo-cartographer/${REPO}/public-drafts"

grep -rnE \
  -e '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' \
  -e '@contentful\.com|atlassian\.net|slack\.com/archives|grafana\.|datadoghq\.|pagerduty\.com' \
  -e '\b[A-Z]{2,8}-[0-9]+\b' \
  -e 'Confluence [0-9]+' \
  -e '(Confluence|Glean)[^[:space:]]{0,40}[0-9]{8,12}\b' \
  -e '#[a-z0-9][a-z0-9_-]{2,}' \
  -e '@contentful/team-[A-Za-z0-9_-]+' \
  -e '\.contentful\.tools\b' \
  -e 'backstage\.[A-Za-z0-9.-]*\.contentful' \
  -e '\[Source:[[:space:]]*Glean' \
  "$SCRATCH"
```

Empty stdout = pass. Non-empty stdout = fix matches and re-run.

## Maintenance

When you add or change a pattern:

1. Update the table above.
2. Update the Verification Command grep call.
3. If the pattern needs an exemption mechanism (like P6 / P7), document it in a subsection.
4. Add an example match line in the relevant subsection so future readers can see what triggered the change.
