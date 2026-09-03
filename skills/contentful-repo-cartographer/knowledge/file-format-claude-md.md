# CLAUDE.md — Format Reference

> **Scope: internal-only.** CLAUDE.md is never proposed to public repos — agent guidance for public repos lives in `AGENTS.md`, which is the agnostic standard. CLAUDE.md typically carries internal Slack channels, Jira project keys, and team workflow references that cannot ship publicly. For private repos, it is proposed to the target repo root.

Claude Code project instructions file. Loaded automatically into every Claude Code conversation within the repo. Provides the model with project-specific conventions, tool configuration, and workspace context that isn't discoverable from code alone.

## Placement

- `CLAUDE.md` at repo root (loaded automatically by Claude Code)
- Additional files at `.claude/docs/*.md` for overflow (manually referenced)

## Core Sections

| Section              | Purpose                                                                                                                                              |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Project Overview** | One-paragraph summary — what this repo is, who maintains it                                                                                          |
| **Commands**         | Only invocations the model would get wrong from `package.json` alone — non-obvious flags, required env, ordering. Omit when nothing qualifies.       |
| **Architecture**     | Brief pointer to ARCHITECTURE.md; mention key patterns the model must respect                                                                        |
| **Conventions**      | Commit format, branch naming, code style rules not enforced by linters                                                                               |
| **Testing**          | Only what the model would get wrong from `package.json` and the test config alone — required services, ordering, flags. Omit when nothing qualifies. |
| **Sharp Edges**      | Things that break silently or behave unexpectedly — same spirit as AGENTS.md                                                                         |

## Optional Sections

| Section              | When to Include                                                                        |
| -------------------- | -------------------------------------------------------------------------------------- |
| **MCP Tools**        | If the repo uses MCP servers or has `.mcp.json` configuration                          |
| **Workspace Layout** | Monorepos or non-obvious directory structures                                          |
| **Environment**      | Required env vars for local dev (NOT secret values — just names and where to get them) |
| **Dependencies**     | Non-obvious setup steps (Docker, localstack, native extensions)                        |
| **CI/CD**            | What the model should know about pipeline behavior                                     |

## Rules

1. **No secrets.** Never include API keys, tokens, passwords, or internal URLs that would be sensitive in public repos. Reference `.env.example` or vault paths instead.
2. **Actionable over descriptive.** Every section should tell the model what to DO, not what the codebase IS. "Run `npm test`" not "Tests exist in the `test/` directory."
3. **Non-discoverable content only.** Don't repeat what `package.json` scripts, `tsconfig.json`, or linter configs already express. Focus on things the model would get wrong without guidance (see `knowledge/research-basis.md` § Context files: a null result on task success).
4. **Concise.** Target 100-200 lines. Claude Code loads this into every conversation — bloat wastes context window and pushes load-bearing rules into the middle of the file, where they are followed least reliably (see `knowledge/research-basis.md` § Long context buries mid-context information).
5. **Imperative tone.** Write as instructions to the model: "Always use...", "Never...", "When changing X, also update Y."

## Relationship to Other Artifacts

- **AGENTS.md** — routing table for agents (which files matter, sharp edges, invariants). CLAUDE.md is broader: project identity + workflow + conventions.
- **CONTRIBUTING.md** — specialized repo-specific procedures plus pointers to the global convention skills. CLAUDE.md is model-focused: terser, imperative, assumes the reader is an LLM following instructions.
- **ARCHITECTURE.md** — deep system knowledge. CLAUDE.md just points to it and highlights what the model must respect.

## Anti-Patterns

- Copying the entire CONTRIBUTING.md into CLAUDE.md (redundant, bloated)
- Including file trees that go stale within a week
- Documenting every command from `package.json` (the model can read that itself)
- Adding "personality" instructions unrelated to code quality
- Exceeding 300 lines (signals scope creep — split into `.claude/docs/`)

## Template

Two sections in the template are conditional. Generate **Commands** only for invocations the model would get wrong from `package.json` alone, and **Testing** only for what the model would get wrong from `package.json` and the test config alone. Omit either section from the generated file when nothing qualifies.

```markdown
# <Project Name>

<One paragraph: what this is, who owns it, primary language/framework.>

## Commands

| Task                 | Command     | Why it is not obvious from `package.json` |
| -------------------- | ----------- | ----------------------------------------- |
| `<non-obvious task>` | `<command>` | `<required flag, env var, or ordering>`   |

## Conventions

- <Commit format>
- <Branch naming>
- <Import ordering or other non-linter-enforced rules>

## Architecture

See [ARCHITECTURE.md](./ARCHITECTURE.md) for full details.

Key patterns to respect:

- <Pattern 1>
- <Pattern 2>

## Sharp Edges

- <Thing that breaks silently>
- <Non-obvious constraint>

## Testing

- <Service or fixture a suite requires before it runs>
- <Ordering or flag the test scripts do not express>
- <What must pass before PR>
```
