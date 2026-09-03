# Consumer Reference

Use this reference when configuring a downstream repository or user profile to consume Agents Kit skills or hooks.

## Setup Flow

1. Detect the package manager from `package.json` and lockfiles.
2. Install the CLI package as a dev dependency:

```sh
pnpm add -D @contentful/agents-kit
```

3. Preview inferred config:

```sh
pnpm exec contentful-agents-kit init --dry-run
```

4. Apply setup:

```sh
pnpm exec contentful-agents-kit init
pnpm install
pnpm exec contentful-agents-kit skills install --dry-run
pnpm exec contentful-agents-kit skills install
```

Use `--scope user` for skills that should apply across repositories. User scope stores config in `~/.config/agents-kit`.

## Config Shape

Agents Kit consumer config lives in `package.json` under `agentsKit.skills.uses`.

Minimal repo-scope config uses version 2. Omitting `install` selects the canonical
`./skills` root and relative native-agent symlinks:

```json
{
  "agentsKit": {
    "$schema": "./node_modules/@contentful/agents-kit/schema.json",
    "version": 2,
    "skills": {
      "uses": {
        "agents": ["codex", "cursor", "claude-code"],
        "sources": [
          { "type": "package", "name": "@contentful/agents-skill-repository-bundle" },
          { "type": "package", "name": "@contentful/agents-kit" },
          { "type": "directory", "path": "./skills/repo-local-skill" }
        ]
      }
    }
  }
}
```

`init` adds the first two package sources by default. It also writes
`@contentful/agents-kit` as a direct dev dependency, so the shipped skill resolves
from the repository after dependencies are installed. Supported public agent ids are
`codex`, `cursor`, and `claude-code`.

## Version 1 deprecation

Consumer configuration version 1 is deprecated. It remains supported throughout
the Agents Kit 1.x lifecycle, including copy-mode behavior when `install` is
omitted.

Migrate repository consumers to version 2 before any future removal of version
1 support. Version 2 uses the canonical `./skills` root and relative native-agent
symlinks by default. Consumers that need copied skill files can set
`agentsKit.skills.uses.install.mode` to `copy`.

## Shared-Root-Only Consumers

To update only a shared root and create no `.agents/skills`, `.cursor/skills`, or
`.claude/skills` projections, omit `agents` or set it to an empty array. Version 2
already defaults to the required shared-root symlink layout:

```json
{
  "agentsKit": {
    "version": 2,
    "skills": {
      "uses": {
        "agents": [],
        "sources": [{ "type": "package", "name": "@contentful/agents-skill-repository-bundle" }]
      }
    }
  }
}
```

`skills install` warns that no agent targets are configured, then updates only `./skills`.

User-scope init keeps version 1 with an explicit symlink root and defaults to the
developer bundle plus `@contentful/agents-kit`. Run its printed package-manager
install command before installing skills.

## Common Commands

- Discover Contentful skill packages: `pnpm exec contentful-agents-kit skills find`
- Discover package and GitHub-hosted user skills: `pnpm exec contentful-agents-kit skills find --scope user`
- Add package or GitHub skill sources: `pnpm exec contentful-agents-kit skills add <source>`
- Remove configured sources: `pnpm exec contentful-agents-kit skills remove <source>`
- Preview skill install output: `pnpm exec contentful-agents-kit skills install --dry-run`
- Install configured skills: `pnpm exec contentful-agents-kit skills install`
- Approve reviewed deletes and replacements in a non-interactive run: `pnpm exec contentful-agents-kit skills install --allow-destructive`
- Preview every configured resource install: `pnpm exec contentful-agents-kit install --dry-run`
- Install every configured resource: `pnpm exec contentful-agents-kit install`
- Install selected resources: `pnpm exec contentful-agents-kit install --skills --hooks`
- List installed skills: `contentful-agents-kit skills list`
- Update user-scope skills: `contentful-agents-kit skills update`

Use npm, Yarn, or Bun equivalents when the repository does not use pnpm.

## Source Types

- `package`: reusable npm package source resolved from installed dependencies.
- `directory`: repo-local skill root or local multi-skill package.
- `github`: user-scope GitHub URL source for skills outside npm packages.

Package skill sources must be installed dependencies. Directory skill roots must contain `SKILL.md`.

For an existing initialized consumer, add the shipped guidance explicitly:

```sh
pnpm exec contentful-agents-kit skills add @contentful/agents-kit
```

## Add a Local Directory Skill

Use this flow when the skill belongs to the current repository instead of a shared package.

1. Choose a source location outside native agent target folders.

Preferred when using symlink mode:

```text
skills/my-repo-skill/
  SKILL.md
```

Also valid when the team wants source skills separate from installed output:

```text
agent-skills/my-repo-skill/
  SKILL.md
```

Never create source skills under `.agents/skills`, `.cursor/skills`, or `.claude/skills`; those are install targets.

2. Create `SKILL.md`:

```md
---
name: my-repo-skill
description: Use this skill for this repository's local workflow.
---

# My Repo Skill

Describe when to use the skill and the steps the agent must follow.
```

3. Add the directory source to `package.json`:

```json
{
  "agentsKit": {
    "$schema": "./node_modules/@contentful/agents-kit/schema.json",
    "version": 2,
    "skills": {
      "uses": {
        "agents": ["codex", "cursor", "claude-code"],
        "sources": [{ "type": "directory", "path": "./skills/my-repo-skill" }]
      }
    }
  }
}
```

4. Preview and install:

```sh
pnpm exec contentful-agents-kit skills install --dry-run
pnpm exec contentful-agents-kit skills install
```

In symlink mode, the install command creates the links from native agent target folders to the shared skill root. Do not hand-create those symlinks unless you are repairing generated output.

Before writing, `skills install` reports existing paths that the resolved plan would remove or replace because their content or link target differs. Interactive terminals can adopt valid untracked skills into the stable source root and add them to config, delete them, or cancel. Non-interactive runs require `--allow-destructive` to delete untracked skills or replace conflicting content. Cancelling leaves project files and persisted install-mode config unchanged.

5. Commit the source skill, updated `package.json`, and generated install output.

## Use `./skills` as the Shared Root

Use this layout when the repository wants `./skills` to be the single source of truth:

- package skills are materialized into `./skills/<skill-name>` by Agents Kit
- local directory skills live in `./skills/<local-skill-name>`
- `.agents/skills`, `.cursor/skills`, and `.claude/skills` contain symlinks to `./skills`

For a version 2 repository consumer, no extra install configuration is needed.
Preview the default layout with `skills install --dry-run`. To opt out, run
`skills install --mode copy`; that persists `{ "mode": "copy" }`.

When adding a local skill in this layout, add the local source as a directory source under `./skills`:

```json
{
  "type": "directory",
  "path": "./skills/my-repo-skill"
}
```

Rules for this layout:

- `./skills/<local-skill-name>` is both the local skill source and the shared-root destination for that skill.
- package skill folders under `./skills` are generated from package sources; update the package dependency and rerun install instead of editing them by hand.
- using `./skills` itself as a skill root is invalid because each skill root must contain one `SKILL.md`.
- always run `skills install --dry-run` after changing sources to catch path overlap or install-name collisions before writing files.
- commit `package.json`, `./skills`, and native target symlinks together.

## Updates

Use [Automation and Updates Reference](automation-and-updates.md) for Renovate-triggered repo updates and user-scope update checks.

## Hooks and Rules

Use [Hook Provider Reference](provider-hooks.md) for consuming and authoring hooks.

Use [Rules Reference](rules.md) for consuming and authoring rules.

## Troubleshooting

Use [Troubleshooting Reference](troubleshooting.md) for package resolution, source collisions, stale generated output, publishability, and symlink issues.
