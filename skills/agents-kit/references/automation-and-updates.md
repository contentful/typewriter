# Automation and Updates Reference

Use this reference when keeping installed Agents Kit output current over time.

## Repository Skill Updates

First check whether the repository is backed by service-kit or `@contentful/nx`.
Those repositories handle Agents Kit update automation by default and usually
need no extra setup.

For repositories that are not covered by service-kit or `@contentful/nx`, and
that receive Renovate PRs for skill package dependency updates, add the shared
Agents Kit update workflow.

```yaml
name: Agents Kit update

on:
  pull_request:
    paths:
      - package.json
      - pnpm-lock.yaml

jobs:
  agents-kit-skills-install:
    permissions:
      contents: write
      id-token: write
      packages: read
      pull-requests: read
    uses: contentful/developer-platform/.github/workflows/job-agents-kit-update.yaml@main
    secrets: inherit
```

Match trigger paths to the package manager:

- pnpm: `package.json`, `pnpm-lock.yaml`
- npm: `package.json`, `package-lock.json`
- npm shrinkwrap: `package.json`, `npm-shrinkwrap.json`
- Yarn: `package.json`, `yarn.lock`

The workflow reruns `contentful-agents-kit skills install`, commits refreshed generated output, and pushes back to the Renovate branch.

## Vault Policies

The shared workflow needs a GitHub push token and package read access:

```yaml
version: 1
services:
  github-action:
    policies:
      - github-push
      - packages-read
```

Keep existing unrelated policies.

## Branch Protection

Require this status check on the main branch:

```text
agents-kit-skills-install / agents-kit-skills-install
```

This blocks merging skill package dependency updates until generated installed skills are refreshed or confirmed unchanged.

## User-Scope Updates

For user-scope skills, install the first-party update-check hook package in `~/.config/agents-kit`:

```json
{
  "agentsKit": {
    "$schema": "./node_modules/@contentful/agents-kit/schema.json",
    "version": 1,
    "hooks": {
      "uses": {
        "agents": ["codex", "claude-code"],
        "sources": [{ "type": "package", "name": "@contentful/agents-hook-skills-update-check" }]
      }
    }
  },
  "dependencies": {
    "@contentful/agents-hook-skills-update-check": "latest"
  }
}
```

Then run:

```sh
contentful-agents-kit hooks install --scope user
```

The hook runs `contentful-agents-kit skills update --check` on supported agent session starts. It is notify-only and throttled to once every 24 hours by default.

Override interval:

```sh
AGENTS_KIT_SKILLS_UPDATE_CHECK_INTERVAL_HOURS=0 contentful-agents-kit skills update --check
```

## Manual User Updates

Check:

```sh
contentful-agents-kit skills update --check
```

Apply:

```sh
contentful-agents-kit skills update
```

Force reinstall when versions have not changed:

```sh
contentful-agents-kit skills update --force
```
