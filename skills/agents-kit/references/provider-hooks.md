# Hook Provider Reference

Use this reference when consuming, authoring, packaging, validating, or installing Agents Kit hooks.

## Consumer Config

Hook consumer config lives under `agentsKit.hooks.uses`.

Package source:

```json
{
  "type": "package",
  "name": "@your-org/agent-hooks"
}
```

Directory source:

```json
{
  "type": "directory",
  "path": "./hooks/preflight"
}
```

Supported hook agents are `codex` and `claude-code`.

Preview and install:

```sh
pnpm exec contentful-agents-kit hooks install --dry-run
pnpm exec contentful-agents-kit hooks install
```

User scope:

```sh
contentful-agents-kit hooks install --scope user
```

## Hook Root Shape

A hook root contains a `HOOK.yaml` plus any scripts it invokes:

```text
hooks/skills-update-check/
  HOOK.yaml
  check.mjs
```

Minimal `HOOK.yaml`:

```yaml
name: Skills Update Check
description: Checks user-scoped Agents Kit skills for updates.
events:
  - on: SessionStart
    command: check.mjs
    timeout: 30
    statusMessage: Checking Agents Kit skill updates
```

Keep scripts inside the hook root so they can be copied into native hook folders.

## Provider Config

Hook provider config lives under `agentsKit.hooks.provides`.

```json
{
  "name": "@your-org/agent-hooks",
  "version": "1.0.0",
  "files": ["hooks/preflight/**"],
  "agentsKit": {
    "$schema": "./node_modules/@contentful/agents-kit/schema.json",
    "version": 1,
    "hooks": {
      "provides": ["hooks/preflight"]
    }
  }
}
```

Each provided hook root must contain `HOOK.yaml`.

## Validate

Run from the provider package root:

```sh
pnpm exec contentful-agents-kit hooks validate
```

Validation checks that provided hook roots resolve, include `HOOK.yaml`, and are publishable.

## Native Outputs

Codex install writes:

- `.agents/hooks/<hook-name>/`
- `.agents/hooks/hooks.json`
- `.codex/hooks.json`
- `.codex/config.toml`

Claude Code install writes:

- `.claude/hooks/<hook-name>/`
- `.claude/settings.json`

Agents Kit preserves unrelated native hook config and replaces only generated entries marked as Agents Kit-managed.

## Hook Checklist

- `HOOK.yaml` lives at each provided hook root
- invoked scripts are inside the hook root and executable when needed
- provider `files` includes hook assets
- `hooks validate` passes
- downstream `hooks install --dry-run` shows expected events, commands, and native target files
