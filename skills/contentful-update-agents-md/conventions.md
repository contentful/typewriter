# AGENTS.md Conventions

Reference file for the `contentful-update-agents-md` audit skill. Read this during audits; do not memorize it between sessions.

---

## 1. Philosophy

AGENTS.md is a **compact, always-on contract**. It tells coding agents what they cannot discover from code, config, or tooling — and nothing else.

| Principle                                      | Rationale                                                                                                                            |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Only non-discoverable, high-impact constraints | Agents already read `tsconfig.json`, `package.json`, lint configs, and directory structure. Restating discoverable facts adds noise. |
| Unnecessary context degrades performance       | Research shows bloated context files reduce task success rates and increase inference cost by >20%. Keep AGENTS.md under ~100 lines. |
| Minimal surface area                           | Push procedural instructions, decision trees, and code examples into skills and hooks. AGENTS.md is for constraints, not workflows.  |

**Litmus test:** If removing a line from AGENTS.md would cause an agent to violate a repo constraint it has no other way to learn, the line belongs. Otherwise it does not.

---

## 2. What Belongs in AGENTS.md

Items that are **not discoverable** from code, config, or tooling:

- **Package manager enforcement** — when multiple lockfiles could confuse (e.g., "Use pnpm. Do not run npm install.")
- **Import style constraints** not enforced by linting (e.g., "No barrel re-exports in feature modules")
- **Error handling patterns** specific to this codebase (e.g., "Use `@contentful/errors` classes, not raw `Error`")
- **Auth/security patterns** that are non-obvious (e.g., "Call `authorizeRequest()` before any mutation handler")
- **Anti-patterns** specific to this codebase (e.g., "Do not use legacy CMA clients")
- **Safety rules** — what to never do, what requires asking the user first
- **"Ask first" boundaries** — high-risk changes like modifying shared config, deleting public APIs, or altering CI pipelines

---

## 3. What Does NOT Belong in AGENTS.md

Items that are **discoverable** or **procedural** — put them elsewhere:

| Exclude                                | Why          | Where Instead                   |
| -------------------------------------- | ------------ | ------------------------------- |
| Quick-start / setup commands           | Discoverable | README, `package.json` scripts  |
| File/module naming conventions         | Discoverable | Lint rules + existing code      |
| Detailed feature structure / routing   | Discoverable | Directory structure             |
| Testing taxonomy & framework details   | Discoverable | Test config + `devDependencies` |
| State management decision trees        | Procedural   | Skill or architecture doc       |
| i18n key patterns                      | Discoverable | Lint rules or a skill           |
| Dependency/build/lint tool preferences | Discoverable | Config files                    |
| Code examples longer than one line     | Procedural   | Skill file                      |

---

## 4. Size & Structure Guidelines

- **Target:** root AGENTS.md under ~100 lines.
- Use **nested `AGENTS.md` files** in subdirectories for scope-specific guidance in large repos.
- Prefer **Do/Don't bullets** or guardrail lists over prose paragraphs.
- No setup commands. No multi-line code examples.

**Recommended skeleton:**

```text
1. Brief intro (1-3 lines: repo purpose, monorepo tool, package manager)
2. Guardrails (Do / Don't bullets)
3. Commands (only if non-obvious, e.g., "pnpm exec tsc -b" over "npx tsc")
4. Safety / Permissions (never do X, ask before Y)
```

---

## 5. Staleness Signals

Cross-reference AGENTS.md claims against repo truth. Flag any mismatch as `stale`.

| Repo Signal          | What to Check                                        | Source of Truth                                                                     |
| -------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Package manager      | Does AGENTS.md claim match the lockfile present?     | Presence of `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`                    |
| Test framework       | Does AGENTS.md reference the correct test runner?    | `devDependencies` + config files (`jest.config.*`, `vitest.config.*`, `.mocharc.*`) |
| Build/dev commands   | Do listed commands exist in `package.json` scripts?  | `scripts` field in root and workspace `package.json` files                          |
| Node/runtime version | Does stated version match config?                    | `.nvmrc`, `.node-version`, `.tool-versions`, `engines`, `volta`                     |
| Key dependencies     | Are referenced frameworks/libraries still installed? | `dependencies` / `devDependencies` in `package.json`                                |
| Branch conventions   | Do naming rules match recent usage?                  | `git log --oneline -20` branch prefixes                                             |

---

## 6. Gap Heuristics

If a signal is present in the repo but absent from AGENTS.md, flag it as a **potential gap**. Only flag when the constraint is genuinely non-discoverable.

| Gap                         | Signal                                                                     | Rationale                                                            | Proposed Text                                                                                |
| --------------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Docker required for tests   | `docker-compose.yml` or `Dockerfile` exists                                | Agent won't know tests need running containers unless told           | `Run docker compose up -d before running integration tests.`                                 |
| Environment setup           | `.env.example` exists                                                      | Required env vars are non-discoverable from code alone               | `Copy .env.example to .env and fill in required values before running the app.`              |
| Credentials for test suites | Test config references external services or prod-like endpoints            | Agents may try to run tests that require credentials they don't have | `Integration tests require <SERVICE> credentials. Ask before running integration suites.`    |
| Monorepo tooling            | `nx.json` or `turbo.json` exists                                           | Agents may use wrong commands without knowing the orchestrator       | `This is an Nx monorepo. Use nx run <project>:<target> instead of running scripts directly.` |
| Pre-commit hooks            | `.husky/` or `.git/hooks/` with custom scripts                             | Hooks can reject commits silently; agents need to know they exist    | `Pre-commit hooks run linting and formatting. Do not bypass with --no-verify.`               |
| Generated code              | Codegen scripts in `package.json` or `codegen.yml` / `graphql-codegen.ts`  | Agents may hand-edit generated files that will be overwritten        | `Files in <path> are generated. Run <codegen command> instead of editing them directly.`     |
| Undocumented warnings       | Warnings in `CONTRIBUTING.md`, PR templates, or CI config not in AGENTS.md | Human-targeted docs that agents don't reliably read                  | _(Derive from the specific gotcha found)_                                                    |

Teams may add domain-specific heuristics following this Signal / Rationale / Proposed Text pattern.
