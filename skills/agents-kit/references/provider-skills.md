# Skill Provider Reference

Use this reference when authoring, packaging, validating, or publishing reusable Agents Kit skills.

## Provider Setup

Skills are shipped from npm packages. A provider package may be:

- a standalone skill package under `libs/`
- any npm package that includes one or more skill roots
- the `@contentful/agents-kit` CLI package itself

Install the CLI in the provider workspace first:

```sh
pnpm add -D @contentful/agents-kit
```

## Single-Skill Package

Use this when the package root is the skill root:

```json
{
  "name": "@your-org/agents-skill-release",
  "version": "1.0.0",
  "files": ["SKILL.md", "references/**", "scripts/**", "templates/**"]
}
```

The package root must contain `SKILL.md`.

## Explicit Skill Roots

Use `agentsKit.skills.provides` when:

- the skill lives in a subdirectory
- the package provides multiple skills
- the package re-exports other skill packages
- the provider wants the published skill root to be explicit

Example:

```json
{
  "name": "@your-org/agent-skills",
  "version": "1.0.0",
  "files": ["skills/release/**", "skills/testing/**"],
  "agentsKit": {
    "$schema": "./node_modules/@contentful/agents-kit/schema.json",
    "version": 1,
    "skills": {
      "provides": ["skills/release", "skills/testing"]
    }
  }
}
```

Each provided root must contain its own `SKILL.md`.

## Re-Export Package Skills

Use `package:<name>` entries when creating a bundle package:

```json
{
  "name": "@contentful/agents-skill-workflow-bundle",
  "dependencies": {
    "@contentful/agents-skill-git-commit": "1.0.0"
  },
  "agentsKit": {
    "$schema": "./node_modules/@contentful/agents-kit/schema.json",
    "version": 1,
    "skills": {
      "provides": ["package:@contentful/agents-skill-git-commit"]
    }
  }
}
```

Package references must resolve from the provider package and should be declared in `dependencies`.

## SKILL.md Shape

Each skill root needs frontmatter:

```md
---
name: release-helper
description: Release this package using the team workflow.
---

# Release Helper

Explain when to use the skill and what the agent must do.
```

The `name` field becomes the installed folder name after normalization.

Keep `SKILL.md` focused. Move long details into supporting files.

## Supporting Files

Place supporting material inside the skill root:

- `references/` for detailed workflow variants, provider/consumer splits, or long background
- `scripts/` for executable helper scripts
- `templates/` for reusable generated artifacts
- `assets/` for static files

Include those paths in `files` so npm publishes them.

When a skill has distinct use modes, keep `SKILL.md` as a router and place the detailed guidance under `references/`. For example:

```text
skills/agents-kit/
  SKILL.md
  references/
    consumer.md
    provider-skills.md
```

## Validate

Run from the provider package root:

```sh
pnpm exec contentful-agents-kit skills validate
```

Validation checks:

- provided roots resolve
- every root contains valid `SKILL.md`
- files are publishable according to npm publish rules
- package references resolve

## Test as a Consumer

Before publishing:

1. install the package in a consumer repo or use a local dependency
2. add the package to `agentsKit.skills.uses.sources`
3. run `contentful-agents-kit skills install --dry-run`
4. run `contentful-agents-kit skills install`
5. inspect installed folder names and copied supporting files

This catches install-name collisions, missing `files` entries, and broken references before release.

## Provider Checklist

- `package.json.files` includes every file the skill needs
- `agentsKit.skills.provides` matches the intended root shape
- no source skill lives under native install targets like `.agents/skills`
- `skills validate` passes from the provider package root
- downstream install dry-run shows the expected skill names and sources
- Roadie docs are preferred for internal links, with GitHub docs as fallback when needed
