# AGENTS.md — File Format Reference

`AGENTS.md` is a **compact, always-on contract**. It tells coding agents what they cannot discover from code, config, or tooling — and nothing else. Heavy content lives in ARCHITECTURE.md, CONTRIBUTING.md, and ADRs.

## Inclusion Litmus Test

**Before writing any line into AGENTS.md, apply this test:**

> If removing this line would cause an agent to violate a repo constraint it has no other way to learn, the line belongs. Otherwise it does not.

The test and its companion bloat rule are recorded in `knowledge/research-basis.md` § The AGENTS.md inclusion litmus test.

Agents already read `tsconfig.json`, `package.json`, lint configs, and directory structure. Restating discoverable facts adds noise and inference cost with no established gain in task success (see `knowledge/research-basis.md` § Context files: a null result on task success; § Instruction count degrades performance monotonically).

## What Belongs

Items that are **not discoverable** from code, config, or tooling:

- **Package manager enforcement** — when multiple lockfiles could confuse (e.g., "Use pnpm. Do not run npm install.")
- **Import style constraints** not enforced by linting (e.g., "No barrel re-exports in feature modules")
- **Error handling patterns** specific to this codebase (e.g., "Use `@contentful/errors` classes, not raw `Error`")
- **Auth/security patterns** that are non-obvious (e.g., "Call `authorizeRequest()` before any mutation handler")
- **Anti-patterns** specific to this codebase (e.g., "Do not use legacy CMA clients")
- **Safety rules** — what to never do, what requires asking the user first
- **"Ask first" boundaries** — high-risk changes like modifying shared config, deleting public APIs, or altering CI pipelines

## What Does NOT Belong

Items that are **discoverable** or **procedural** — put them elsewhere:

| Exclude                                  | Why          | Where Instead                                              |
| ---------------------------------------- | ------------ | ---------------------------------------------------------- |
| Quick-start / setup commands             | Discoverable | README.md                                                  |
| File/module naming conventions           | Discoverable | Lint rules + existing code                                 |
| Detailed feature structure / routing     | Discoverable | Directory structure                                        |
| Testing taxonomy & framework details     | Discoverable | Test config + `devDependencies`                            |
| Commit format, branch strategy           | Discoverable | Global `contentful-*` skills (via CONTRIBUTING.md pointer) |
| Integration points / upstream-downstream | Architecture | ARCHITECTURE.md                                            |
| Code examples longer than one line       | Procedural   | CONTRIBUTING.md or a skill                                 |

## Size Constraint

**Target: under ~100 lines.** If longer, content that fails the litmus test has crept in (see `knowledge/research-basis.md` § Long context buries mid-context information; § AGENTS.md presence reduces agent runtime and output tokens).

Use nested `AGENTS.md` files in subdirectories for scope-specific guidance in large repos.

## Sections (in order)

### 1. Quick Reference Table

A routing table mapping common needs to files:

```markdown
| What you need               | Where to look                            |
| --------------------------- | ---------------------------------------- |
| What this repo does         | [README.md](./README.md)                 |
| How to build, test, and run | [README.md](./README.md)                 |
| How this repo is structured | [ARCHITECTURE.md](./ARCHITECTURE.md)     |
| Specialized procedures      | [CONTRIBUTING.md](./CONTRIBUTING.md)     |
| Why decisions were made     | [docs/ADRs/](./docs/ADRs/)               |
| PR review rules             | [.bito/guidelines/](./.bito/guidelines/) |
```

### 2. Guardrails

Repo-specific rules that agents MUST follow — the "never do X" / "always do Y" / "ask before Z" rules that prevent common mistakes. Use Do/Don't bullets, not prose.

**Rules:**

- Only include constraints evidenced by config, CI, existing docs, or team confirmation
- Each rule should be actionable and verifiable
- Include the "why" when the consequence is non-obvious
- Example: "use pnpm — npm and yarn will corrupt the lockfile"

### 3. Safety & Permissions

High-risk operations that require human confirmation:

- What files/directories are never safe to auto-modify
- What operations must always be confirmed first
- What external integrations have blast radius

### 4. Build & Quality

A single copy-paste verification command block (only if non-obvious from `package.json` scripts):

```bash
# Quick verification loop
<install> && <build> && <test> && <lint>
```

## Staleness Signals

When auditing or refreshing an existing AGENTS.md, cross-reference claims against repo truth. Flag any mismatch as stale:

| Repo Signal          | What to Check                                        | Source of Truth                                                                     |
| -------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Package manager      | Does AGENTS.md claim match the lockfile present?     | Presence of `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`                    |
| Test framework       | Does AGENTS.md reference the correct test runner?    | `devDependencies` + config files (`jest.config.*`, `vitest.config.*`, `.mocharc.*`) |
| Build/dev commands   | Do listed commands exist in `package.json` scripts?  | `scripts` field in root and workspace `package.json` files                          |
| Node/runtime version | Does stated version match config?                    | `.nvmrc`, `.node-version`, `.tool-versions`, `engines`, `volta`                     |
| Key dependencies     | Are referenced frameworks/libraries still installed? | `dependencies` / `devDependencies` in `package.json`                                |
| Branch conventions   | Do naming rules match recent usage?                  | `git log --oneline -20` branch prefixes                                             |

## Gap Heuristics

During Phase 1 discovery, check for these signals. If present in the repo but absent from AGENTS.md, flag as a potential gap. Only add if the constraint is genuinely non-discoverable (apply the litmus test).

| Gap                         | Signal                                                 | Rationale                                                            | Proposed Text Pattern                                                                        |
| --------------------------- | ------------------------------------------------------ | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Docker required for tests   | `docker-compose.yml` or `Dockerfile` exists            | Agent won't know tests need running containers unless told           | `Run docker compose up -d before running integration tests.`                                 |
| Environment setup           | `.env.example` exists                                  | Required env vars are non-discoverable from code alone               | `Copy .env.example to .env and fill in required values before running the app.`              |
| Credentials for test suites | Test config references external services               | Agents may try to run tests that require credentials they don't have | `Integration tests require <SERVICE> credentials. Ask before running integration suites.`    |
| Monorepo tooling            | `nx.json` or `turbo.json` exists                       | Agents may use wrong commands without knowing the orchestrator       | `This is an Nx monorepo. Use nx run <project>:<target> instead of running scripts directly.` |
| Pre-commit hooks            | `.husky/` or `.git/hooks/` with custom scripts         | Hooks can reject commits silently; agents need to know they exist    | `Pre-commit hooks run linting and formatting. Do not bypass with --no-verify.`               |
| Generated code              | Codegen scripts in `package.json` or `codegen.yml`     | Agents may hand-edit generated files that will be overwritten        | `Files in <path> are generated. Run <codegen command> instead of editing them directly.`     |
| Undocumented warnings       | Warnings in CI config or PR templates not in AGENTS.md | Human-targeted docs that agents don't reliably read                  | _(Derive from the specific gotcha found)_                                                    |

## Handling Existing AGENTS.md

Many repos already have an AGENTS.md with team-authored constraints, permissions, and file-level guidance. When one exists:

1. Read it first — it is authoritative
2. Preserve all human-authored content
3. Apply the litmus test to existing content — flag `bloat` and `misplaced` items but do NOT remove without team approval
4. Move detailed file-level guidance to CONTRIBUTING.md where it belongs
5. Move detailed commands to the file that owns them — setup/build/test commands to README.md, specialized-procedure commands to CONTRIBUTING.md; drop command catalogues that only restate `package.json` scripts

## Verification

After generating AGENTS.md, check:

1. `wc -l` — is it under 100 lines? If not, re-apply the litmus test to every section.
2. For each line, ask: "Could an agent discover this from reading config files?" If yes, remove it.
3. Confirm no integration points, no commit conventions, no test framework details remain — those belong elsewhere.
