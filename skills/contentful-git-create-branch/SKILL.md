---
name: contentful-git-create-branch
description: Create and switch to a new Git branch in repositories owned by the contentful GitHub organization with safe branch checks and consistent naming. Use when a user asks to start work on a ticket, create a feature/fix/chore branch, or move off main/master before making commits.
---

# Contentful Git Create Branch

Apply this workflow only when asked to create a branch in a `contentful/*` repository.

## Scope Gate (Required)

Before applying any rule below, verify the current repository belongs to the `contentful` GitHub organization.

```bash
git remote get-url origin
```

Treat the repo as in-scope only when the origin URL contains `github.com` and `contentful/` (for example `git@github.com:contentful/<repo>.git` or `https://github.com/contentful/<repo>.git`).

If the repository is not in-scope, stop using this skill and fall back to generic branch guidance.

## Required Inputs

Collect:

- branch purpose/type (`feat`, `fix`, `chore`, `docs`, `refactor`, etc.)
- JIRA ticket key in uppercase (e.g. `CAO-123`, `PLAT-456`)
- short description slug in kebab-case

Use this naming pattern:

```text
<type>/<TICKET-KEY>-<short-description>
```

Examples:

- `feat/CAO-123-add-user-auth`
- `fix/PLAT-456-null-pointer-error`
- `docs/CAO-789-update-onboarding-guide`

## Workflow

### 1. Check current state

Run:

```bash
git branch --show-current
git status --porcelain
```

If the working tree is dirty, call it out before switching branch so the user can decide whether to continue.

### 2. Ensure branch name is valid

Require:

- JIRA ticket key in uppercase (e.g. `CAO-123`) immediately after the type slash
- lowercase letters, digits, and hyphens in the short description
- slash-separated `<type>/<TICKET-KEY>-<short-description>` format
- no spaces; only the TICKET-KEY part uses uppercase

### 3. Create and switch branch

Create the branch from the current HEAD:

```bash
git checkout -b <type>/<TICKET-KEY>-<short-description>
```

If the branch already exists, do not overwrite it. Switch to it instead:

```bash
git checkout <type>/<TICKET-KEY>-<short-description>
```

### 4. Verify result

Confirm:

```bash
git branch --show-current
```

Proceed only when the active branch matches the expected name and is not `main`/`master`.

## Safety Rules

- Never create branches in non-`contentful/*` repositories while this skill is active.
- Never delete, rename, or force-reset branches unless the user explicitly asks.
- Never auto-stash or auto-commit local changes without user instruction.
- If branch creation fails, report the exact git error and suggest the safest next command.
