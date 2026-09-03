---
name: contentful-update-agents-md
description: Audit and update AGENTS.md metadata in contentful/* repositories. Use when AGENTS.md may be stale, bloated, or incomplete after package, test, build, CI, runtime, or directory changes.
---

# Update AGENTS.md

Audit AGENTS.md for convention compliance, staleness, and gaps in `contentful/*` repositories.

## Scope Gate (Required)

Before applying any rule below, verify the current repository belongs to the `contentful` GitHub organization.

```bash
git remote get-url origin
```

Treat the repo as in-scope only when the origin URL contains `github.com` and `contentful/` (for example `git@github.com:contentful/<repo>.git` or `https://github.com/contentful/<repo>.git`).

If the repository is not in-scope, stop using this skill and fall back to generic AGENTS.md guidance.

## Workflow

Execute all three layers in order. For the full conventions reference, read [conventions.md](conventions.md).

### Layer 1 — Convention Compliance

1. Read the existing AGENTS.md. If missing, skip to Layer 3 with a note.
2. Count lines — flag if over ~100 lines (see conventions.md Section 4).
3. Scan content against Section 3 of conventions.md ("What Does NOT Belong").
4. Tag each finding:

| Tag         | Meaning                                                |
| ----------- | ------------------------------------------------------ |
| `bloat`     | Should be removed — discoverable or procedural content |
| `misplaced` | Belongs in a skill, README, or config instead          |
| `ok`        | Passes the litmus test — non-discoverable, high-impact |

### Layer 2 — Staleness Detection

1. Scan repo signals:

   ```bash
   ls package-lock.json pnpm-lock.yaml yarn.lock 2>/dev/null
   jq '{scripts, engines, dependencies, devDependencies}' package.json
   ls .nvmrc .node-version .tool-versions 2>/dev/null
   ls jest.config.* vitest.config.* .mocharc.* 2>/dev/null
   ls docker-compose.yml Dockerfile 2>/dev/null
   ls nx.json turbo.json 2>/dev/null
   ```

2. Cross-reference each signal against claims in AGENTS.md (see conventions.md Section 5 for the full checklist).
3. Flag contradictions:

| Field        | Content                      |
| ------------ | ---------------------------- |
| `claim`      | What AGENTS.md says          |
| `reality`    | What the repo actually shows |
| `suggestion` | Proposed correction          |

### Layer 3 — Gap Analysis

1. Reference conventions.md Section 6 for the full heuristics list.
2. For each heuristic: check if the signal exists in the repo AND the constraint is absent from AGENTS.md.
3. Only suggest additions that are genuinely non-discoverable — apply the litmus test from conventions.md Section 1.
4. Each suggestion includes:

| Field           | Content                           |
| --------------- | --------------------------------- |
| `signal`        | What was detected in the repo     |
| `rationale`     | Why this is non-discoverable      |
| `proposed-text` | Exact line(s) to add to AGENTS.md |

## Output

Present a structured audit report, then propose changes.

```text
## Audit Report

### Convention Compliance
[findings tagged bloat/misplaced/ok]

### Staleness
[claim/reality/suggestion for each stale item]

### Gaps
[signal/rationale/proposed-text for each gap]

## Proposed Changes
[Show the proposed AGENTS.md content or diff]
```

If the audit finds no issues, report clean and exit — do not propose unnecessary changes.

## Trigger Conditions

Re-run this audit when any of these file patterns change:

| Pattern             | Examples                                                     |
| ------------------- | ------------------------------------------------------------ |
| Package manifest    | `package.json` (dependencies, scripts, engines)              |
| Test config         | `jest.config.*`, `vitest.config.*`, `.mocharc.*`             |
| Build config        | `tsconfig.*`, `webpack.config.*`, `vite.config.*`, `nx.json` |
| CI config           | `.github/workflows/*`, `Jenkinsfile`                         |
| Container config    | `Dockerfile`, `docker-compose.yml`                           |
| Runtime version     | `.nvmrc`, `.node-version`, `.tool-versions`                  |
| Directory structure | New top-level dirs, moved modules                            |

## Skill Composition

If changes are approved by the user, invoke `$contentful-git-commit` to commit the AGENTS.md update.

This skill can be invoked from Cursor hooks or git hooks for automated audits on relevant file changes.

## Safety Rules

- Never overwrite AGENTS.md without explicit user confirmation.
- Never add discoverable content to AGENTS.md — apply the litmus test.
- Never remove content without explaining why it fails the conventions.
- If AGENTS.md doesn't exist, offer to create a minimal one from the skeleton in conventions.md Section 4.
- If the audit finds no issues, report clean and exit.
