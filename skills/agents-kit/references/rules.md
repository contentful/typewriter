# Rules Reference

Use this reference when consuming, authoring, packaging, validating, or installing Agents Kit rules.

Rules are separate from skills. Skills are task-specific playbooks; rules are ambient instructions for supported rule targets such as Bito AI code review.

## Consumer Config

Rule consumer config lives under `agentsKit.rules.uses`.

Package source:

```json
{
  "type": "package",
  "name": "@your-org/agent-rules"
}
```

Directory source:

```json
{
  "type": "directory",
  "path": "./rules/code-review"
}
```

Preview and install:

```sh
pnpm exec contentful-agents-kit rules install --dry-run
pnpm exec contentful-agents-kit rules install
```

## Provider Config

Rule provider config lives under `agentsKit.rules.provides`.

```json
{
  "name": "@your-org/agent-rules",
  "version": "1.0.0",
  "files": ["rules/code-review/**"],
  "agentsKit": {
    "$schema": "./node_modules/@contentful/agents-kit/schema.json",
    "version": 1,
    "rules": {
      "provides": ["rules/code-review"]
    }
  }
}
```

Each provided rule root must contain its rule definition file as required by the current schema and target renderer.

## Validate

Run from the provider package root:

```sh
pnpm exec contentful-agents-kit rules validate
```

Validation checks provided rule roots, schema shape, and publishability.

## Operating Guidance

- Do not install rules with `skills install`; use `rules install`.
- Do not configure rules under `agentsKit.skills`.
- Keep rule source roots outside native generated target files.
- Preview with `rules install --dry-run` before writing generated outputs.
- If a package provides skills but no rules, `rules install` will tell you to use `skills install`.

## Rules Checklist

- Consumer config uses `agentsKit.rules.uses`
- Provider config uses `agentsKit.rules.provides`
- package `files` includes rule assets
- `rules validate` passes from provider root
- downstream `rules install --dry-run` shows expected target outputs
