---
name: contentful-git-commit
description: Commit code changes with Contentful commit hygiene standards, only in repositories owned by the contentful GitHub organization. Use when creating commits or validating commit readiness in contentful/* repositories. Enforce Conventional Commit format with required Jira key, single-intent commits, review-ready checks, commit size guidance, and clear author ownership.
---

# Contentful Git Commit

Apply this workflow only when asked to commit changes in a `contentful/*` repository.

## Scope Gate (Required)

Before applying any rule below, verify the current repository belongs to the `contentful` GitHub organization.

Treat the repo as in-scope only when the origin URL contains `github.com` and `contentful/` (for example `git@github.com:contentful/<repo>.git` or `https://github.com/contentful/<repo>.git`).

If the repository is not in-scope, stop using this skill and fall back to generic commit guidance.

## Skill Composition

If the user needs branch creation or branch correction before committing, do that first, then continue with this commit workflow.
If `$contentful-git-create-branch` is available, prefer using it.

## Prerequisites

Before committing, ensure you're working on a feature branch, not the main branch.
Determine the current branch before proceeding.
If you're on `main` or `master`, create or switch to a valid feature branch before proceeding.
If `$contentful-git-create-branch` is available, prefer using it.

Branch naming should follow the pattern enforced by `$contentful-git-create-branch`: `<type>/<short-description>`.

## Required Title Format

Use this format for commit subjects:

```text
<type>(<optional-scope>): <short description> [<TICKET-KEY>]
```

Examples:

- `fix(my-api-contract): require auth header on public endpoints [XY-234]`
- `chore(dependencies): bump service kit packages [XY-123]`
- `docs(studio): fix typo in onboarding guide [HK-120]`
- `fix(search-api): prevent timeout in query execution [INC-77]`

Do not produce titles without a ticket key.

## Ticket Rules

- Require one ticket key in square brackets.
- Accept normal Jira work items.

Use this validation pattern for the ticket part:

```text
\[[A-Z][A-Z0-9]+-\d+\]
```

### Ticket Resolution (lookup before asking)

Before asking the user for a ticket key, attempt to infer it automatically:

1. **Branch name** – inspect the current branch name for a Jira key pattern as stated in the validation pattern. A branch like `feat/PROJ-123-some-feature` yields `PROJ-123`.
2. **Commit log** – scan the branch-local commits (commits not yet on the base/remote branch, or up to the last 10 commits) for the Jira key pattern as stated in the validation pattern. Use the first match found.
3. **Ask the user** – only when neither source yields a key, stop and ask the user to provide one.

## Allowed commit Types

## Commit Types

| Type       | Purpose                               |
| ---------- | ------------------------------------- |
| `build`    | Build system or CI changes            |
| `chore`    | Routine maintenance tasks             |
| `ci`       | Continuous integration configuration  |
| `deps`     | Dependency updates                    |
| `docs`     | Documentation changes                 |
| `feat`     | New feature                           |
| `fix`      | Bug fix                               |
| `perf`     | Performance improvement               |
| `refactor` | Code refactoring (no behavior change) |
| `revert`   | Revert a previous commit              |
| `style`    | Code style and formatting             |
| `test`     | Tests added, updated or improved      |

## Subject Line Rules

- Use imperative, present tense: "add feature" not "added feature"
- Start the subject description in lowercase
- Must not be sentence-case, start-case, pascal-case, or upper-case (`subject-case`)
- No period at the end
- Maximum 70 characters
- no vague text like "apply review comments" or "initial work"

## Workflow

### 1. Inspect change state

Inspect both staged and unstaged changes before committing.
If staged changes exist, treat staged diff as commit scope by default.

### 2. Enforce single primary intent

A commit must be summarizable in one sentence.

Split changes into separate commits when they mix intents, such as:

- scaffolding plus behavior change
- refactor plus feature or bug fix
- cleanup plus functional change

If intent is mixed, stage only one logical subset and explain the split.

### 3. Run review-ready preflight

Before committing, verify:

- no debug output left in changed files
- no commented-out dead code
- no placeholder TODO cleanup markers
- no useless scaffolding files
- relevant tests or checks have been run when possible

If preflight fails, fix first and then commit.

If the commit touches `package.json`, build config, test config, CI config, or directory structure, check whether AGENTS.md needs updating by invoking `$contentful-update-agents-md`.

### 4. Check commit-size signal

Estimate staged size before committing.
Use an effective changed-lines count for this advisory by excluding these lockfiles from the staged diff total:

- `pnpm-lock.yaml`
- `package-lock.json`
- `yarn.lock`
- `bun.lock`
- `bun.lockb`
- `composer.lock`
- `Cargo.lock`
- `Gemfile.lock`
- `Podfile.lock`

Lockfiles still count as part of the real change; exclude them only from this size-warning heuristic.
If effective changed lines are over 500, warn and suggest splitting into multiple commits by intent.
If a large single commit is justified, include rationale in the commit body.

### 5. Generate commit subject

Generate one subject line that satisfies:

- required format
- imperative mood
- lowercase subject description to satisfy commitlint `subject-case`
- clear intent
- no vague text like "apply review comments" or "initial work"

Then commit:

```bash
git commit -m "<type>(<optional-scope>): <short description> [<TICKET-KEY>]"
```

## Safety Rules

- Never update git config.
- Never use destructive git commands without explicit user request.
- Never skip hooks with `--no-verify` unless user asks.
- Never force push protected branches.
- If commit fails due to checks, analyze the error and suggest a fix to the user.

## Principles

- **Required repo state over exact commands**: The repo checks and commit constraints are contractual; exact Git inspection commands are not.
- **Portable branch correction**: Require a valid working branch before committing; use `$contentful-git-create-branch` only when available.
