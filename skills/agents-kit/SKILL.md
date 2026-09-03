---
name: agents-kit
description: Use Agents Kit to configure, install, update, validate, provide, and publish agent skills, rules, and hooks. Use when a user asks how to adopt Agents Kit, add package/local/GitHub skill sources, manage repo or user scoped installs, use the version 2 canonical symlink layout, wire update automation, author provider packages, or troubleshoot installed skill output.
---

# Agents Kit

Use this skill when helping with Agents Kit usage.

## Primary References

- Roadie TechDocs: https://contentful.roadie.so/docs/default/component/agents-kit
- Fallback GitHub docs: https://github.com/contentful/agents-kit/tree/main/docs
- CLI package: https://github.com/contentful/agents-kit/tree/main/apps/cli

Prefer Roadie TechDocs for current Contentful-internal guidance. If Roadie is unavailable or the user needs a public link, use the GitHub docs fallback.

## Route First

Before acting, identify whether the user is a consumer, a provider, or both.

Use [Consumer Reference](references/consumer.md) when the user wants to:

- adopt Agents Kit in a repository or user profile
- add package, local directory, or GitHub skill sources
- install, update, list, or remove skills
- use `./skills` as a shared root or source-of-truth layout

Use [Skill Provider Reference](references/provider-skills.md) when the user wants to:

- create or publish a reusable skill package
- add a skill shipped by `@contentful/agents-kit` or another npm package
- configure `agentsKit.skills.provides`
- add supporting `references/`, `scripts/`, `templates/`, or `assets/`
- validate producer package publishability
- test a package as a downstream consumer

Use [Hook Provider Reference](references/provider-hooks.md) when the user wants to:

- consume, author, package, validate, or install hooks
- configure `agentsKit.hooks.uses` or `agentsKit.hooks.provides`
- work with `HOOK.yaml`, hook scripts, or native hook activation files

Use [Rules Reference](references/rules.md) when the user wants to:

- consume, author, package, validate, or install rules
- configure `agentsKit.rules.uses` or `agentsKit.rules.provides`
- work with rule targets such as Bito AI code review

Use [Automation and Updates Reference](references/automation-and-updates.md) when the user wants to:

- wire Renovate-triggered skill refreshes
- configure the shared GitHub Actions workflow
- install user-scope skill update checks
- understand update branch protection or Vault requirements

Use [Troubleshooting Reference](references/troubleshooting.md) when installs, validation, package resolution, symlinks, publishability, or generated output are failing.

If the request spans both provider and consumer sides, use the provider reference first to shape the package contract, then the consumer reference to verify install behavior.

## Current Consumer Defaults

- New repository-scope `init` writes consumer schema version 2. With no `uses.install` block, skills are materialized once in `./skills` and projected to native agent folders through relative symlinks.
- Version 1 remains supported: omitted `uses.install` means copy mode. Version 2 consumers choose `{ "mode": "copy" }` only when they need that opt-out.
- User-scope `init` keeps its existing version 1 config with an explicit `./skills` canonical root under the resolved Agents Kit home. Its printed package-manager step installs the CLI package there before skill sources resolve.
- Default `init` sources include the scope's Contentful bundle and `@contentful/agents-kit`, so consumers receive this guidance after installing dependencies.

## Scope Rules

- Consumer config lives under `agentsKit.skills.uses`.
- Provider config lives under `agentsKit.skills.provides`.
- Hook consumer config lives under `agentsKit.hooks.uses`; hook provider config lives under `agentsKit.hooks.provides`.
- Rule consumer config lives under `agentsKit.rules.uses`; rule provider config lives under `agentsKit.rules.provides`.
- Do not mix up source roots and install targets: `.agents/skills`, `.cursor/skills`, and `.claude/skills` are generated target folders.
- Always preview writes with `contentful-agents-kit skills install --dry-run` before changing installed output.
- In the `agents-kit` repository itself, do not add this `agents-kit` skill to root `agentsKit.skills.uses`; it is shipped and documented from the CLI package, not consumed as an active repository skill.
