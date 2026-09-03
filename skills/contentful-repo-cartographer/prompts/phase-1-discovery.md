# Phase 1 — Automated Discovery

Adopt the Phase 1 persona: **skeptical technical researcher**. Your job is to discover facts about this codebase, not to narrate or explain it. Every claim you write must trace to a file, commit, or Glean result — if you can't cite the source, you don't write the claim. Assume nothing works the way it appears at first glance: dependencies may be unused, config may be dead, scripts may be deprecated. Verify before documenting.

## Checklist

Create a TodoWrite task for each item. Complete in order.

1. Confirm preconditions (repo exists, catalog-info.yaml parsed)
2. Audit existing coverage (canonical artifact table)
3. Read the codebase (all categories from Step 2)
4. Git archaeology (all queries from Step 3)
5. Glean research (broad queries from Step 4)
6. Backstage enrichment (Phase A → B → C fan-out from Step 4.5)
7. Cross-reference decisions (decision table from Step 5)
8. Produce drafts (all 7 artifacts with inline verification gates)
9. Redaction pass (public repos only — Step 6.5)
10. Glean gap validation (3+ queries per gap from Step 7)
11. Post-gap-fill redaction re-sweep (public repos only — Step 7.5)
12. Compile gap questions (categorized list from Step 8)
13. Run verification gate
14. Write run state

## Inputs

- `TARGET_REPO`: path to the cloned repo (e.g., `repos/extensibility-api`)

## Instructions

You are running Phase 1 (Automated Discovery) of the Repo Cartographer workflow. Read the target repo thoroughly, research decision history, and produce draft Golden Context files covering everything you can determine from evidence. You are NOT inventing or guessing — you are sourcing facts from files and research.

### Step 0: Preconditions

Confirm before proceeding:

1. `TARGET_REPO` path exists and contains files. If not, stop and report: "Repo not found at `<TARGET_REPO>`. Clone it first with `git clone git@github.com:contentful/<repo-name>.git <REPOS_ROOT>/<repo-name>`."
2. `<SCRATCH_ROOT>/repo-cartographer/` directory is writable (create it if missing).
3. **MCP preflight.** Check MCP server health via the host's MCP listing. On Claude Code, run `claude mcp list` or `claude mcp get <name>`; the output reports `✓ Connected` or `✗` for each server. On other MCP-supporting hosts, use the equivalent listing command.
   - **Glean MCP (required — hard stop):** Look for a server whose name matches `glean` (case-insensitive substring) with status `Connected`. If no Glean server is configured, or the configured server is not Connected, STOP immediately and report: `"Glean MCP unavailable (not configured or not connected). The Repo Cartographer requires Glean to source institutional knowledge from Confluence, Slack, RFCs, and design docs — without it the 'Source Everything' and 'Exhaust Glean Before Declaring Gaps' principles cannot be satisfied. Resolve Glean access before re-running."` Do not proceed.
   - **Backstage MCP (degraded if unavailable):** Look for a server whose name matches `backstage` (case-insensitive substring) with status `Connected`. If no Backstage server is configured or it's not Connected, set `BACKSTAGE_AVAILABLE = false` and apply the "Backstage unavailable" degraded mode from `agent.md`. Otherwise `BACKSTAGE_AVAILABLE = true`.

   Report the preflight result inline: `Preflight: Glean=ok, Backstage=<available|degraded>`. Continue with reduced Backstage scope per `agent.md` Degraded Mode if needed.

4. `catalog-info.yaml` exists in the target repo root. Read it and extract:
   - `kind` → ENTITY_KIND (top-level field: Component, API, Resource, etc.)
   - `metadata.name` → ENTITY_NAME (used for all Backstage queries in Step 4.5)
   - `metadata.namespace` → ENTITY_NAMESPACE (default: `"default"`)
   - `spec.type` → ENTITY_TYPE (service, library, website, etc.)
   - `spec.system` → PARENT_SYSTEM (may be absent)
   - `spec.owner` → OWNING_GROUP
   - `spec.lifecycle` → LIFECYCLE (production | experimental | deprecated)

   If `catalog-info.yaml` is missing or unparseable: stop and report: "No catalog-info.yaml found at `<TARGET_REPO>/catalog-info.yaml`. This repo must be registered in Backstage before context enrichment can proceed."

   If `spec.lifecycle: deprecated`: note for Step 4.5 lifecycle gate — operational queries will be skipped and TechDocs results will carry staleness markers.

5. **Visibility detection.** Determine the repo's GitHub visibility. This drives artifact-set selection in Step 6 and the redaction-sweep gate in Step 6.5.

   ```bash
   # Run from inside the cloned repo (TARGET_REPO points there per Step 0 sub-step 1).
   # gh resolves the org/repo from the cwd's git remote — no need to assume the contentful/ org prefix.
   cd "<TARGET_REPO>" && gh repo view --json isPrivate -q '.isPrivate'
   ```

   The command outputs the literal string `true` or `false` (with a trailing newline). Map the trimmed value:
   - output `false` → `REPO_VISIBILITY=public`
   - output `true` → `REPO_VISIBILITY=private`

   If the command fails (auth error, repo not found via `gh`, network failure), STOP. Report: `"Could not determine repo visibility. Resolve gh auth (gh auth status) before re-running. Falling back to a default would risk leaking internal content into a public repo."`

   Report the result inline alongside the preflight summary: `Preflight: Glean=ok, Backstage=<...>, Visibility=<public|private>`.

   **Why this matters:** later steps branch on `REPO_VISIBILITY`. Step 6 routes drafts to `public-drafts/` vs `internal-only/` (public) or a single `drafts/` folder (private). Step 6.5 runs a hard-gate redaction sweep when public; skips when private.

### Step 1: Audit Existing Coverage

Check which canonical artifacts already exist in the target repo:

| Artifact                                 | Present? | Notes |
| ---------------------------------------- | -------- | ----- |
| `./AGENTS.md`                            |          |       |
| `./ARCHITECTURE.md`                      |          |       |
| `./CLAUDE.md`                            |          |       |
| `./CONTRIBUTING.md`                      |          |       |
| `./README.md`                            |          |       |
| `./docs/ADRs/`                           |          |       |
| `./.bito.yaml` + `./.bito/guidelines/`   |          |       |
| `.npmrc`                                 |          |       |
| `.nvmrc` or `engines.node`               |          |       |
| `packageManager` field in `package.json` |          |       |

Read and internalize any existing Golden Context docs — they are authoritative. Only generate what's missing or stale.

### Step 2: Read the Codebase

Read these files/patterns in the target repo (skip any that don't exist):

**Project config:**

- `package.json` (scripts, dependencies, engines, workspaces)
- `pnpm-workspace.yaml` or `lerna.json` or `nx.json` (monorepo config)
- `tsconfig.json` and `tsconfig.*.json` (module system, target, strictness)
- `Makefile`, `Dockerfile`, `docker-compose.yml`

**Code quality config:**

- `eslint.config.*` or `.eslintrc.*` (linting rules)
- `.prettierrc*` or `prettier.config.*` (formatting)
- `.editorconfig`

**CI/CD:**

- `.circleci/config.yml` or `.github/workflows/*.yml`

**Infrastructure:**

- Any `terraform/` directories — `main.tf`, `variables.tf`, `locals.tf`, `outputs.tf`
- `catalog-info.yaml` (Backstage service metadata)

**Existing docs:**

- `README.md`, `CONTRIBUTING.md`, `AGENTS.md`, `ARCHITECTURE.md`
- Any `docs/` directory contents

**Code structure:**

- Top-level directory listing
- Entry points (look for `lib/index.*`, `src/index.*`, `bin/*`, route/handler definitions)
- Entity/model directories (look for patterns like `lib/entities/`, `src/models/`)
- Test directories and test config (`mocha`, `jest`, `vitest` config files)

**External connections:**

- Grep for SNS, SQS, DynamoDB, S3, HTTP client references
- Grep for environment variable usage (`process.env`)
- Look for IAM policies in Terraform

**Legacy detection:**
For any client class, service wrapper, or script you plan to document, check whether it's active or legacy:

- `git log --oneline -5 -- <file>` — if last touched 12+ months ago with no recent callers, flag as potentially legacy
- Grep for imports/requires of the module — if nothing references it, or only tests do, flag it
- Read the file header and first 20 lines — look for deprecation notices, TODO comments, or explicit warnings

Verification: For every external client or script you document, confirm at least one active call site imports or invokes it. If you cannot find a live caller, mark it `[POSSIBLY LEGACY — no active callers found]`.

**Dead-config detection:**
For environment variables and configuration declarations:

- If an env var is declared in config/Terraform but zero source-code files read it at runtime (outside config declaration itself), flag as `[POSSIBLY DEAD CONFIG]`
- Check `git log --oneline -10 -- <config-file>` — a recent PR removing a library may have orphaned the config entry
- Distinguish between: (a) env vars consumed by the framework automatically (e.g., `NODE_ENV`, `DD_SERVICE`), (b) env vars consumed by application code via explicit reads, (c) env vars declared but never consumed — only (b) counts as "active application config"

### Step 3: Git Archaeology

```bash
# First commit (project inception)
git log --reverse --oneline | head -5

# Recent history — active areas
git log --oneline | head -50

# Major structural changes
git log --oneline --all -- package.json | head -30

# Framework/config changes
git log --oneline --all -- tsconfig.json .github/ .circleci/ | head -20

# Dependency additions/changes
git log --oneline --all -- package.json | head -30

# Contributors in last 3 months
git shortlog -sne --since="3 months ago" | head -10
```

Look for: initial setup choices, framework migrations, major refactors, dependency additions/removals that signal decisions.

### Step 4: Glean Research

Use Glean MCP tools to search for external context:

```
search: "<repo-name>"
search: "<repo-name>" architecture OR design
search: "<repo-name>" RFC OR proposal
search: "<repo-name>" decision OR "why"
```

Also try Glean chat for synthesis:

```
chat: "What are the key architectural decisions for the <repo-name> repository at Contentful?"
```

### Step 4.5: Backstage Enrichment

Read the format reference before writing: `knowledge/file-format-backstage-facts.md`

Using the ENTITY_NAME and ENTITY_NAMESPACE extracted in Step 0, query the live Backstage catalog. This step produces a structured snapshot at `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/backstage-facts.md`.

#### Lifecycle Gate

If LIFECYCLE is `deprecated`:

- Skip PagerDuty and Datadog queries (Phase C Steps 8–9)
- Add `[MAY BE STALE — service is deprecated]` markers to all TechDocs results
- Note for Step 6: add a `DEPRECATED SERVICE` sharp-edge warning to AGENTS.md

#### Phase A — Foundation (sequential, establishes inputs for Phase B)

1. **`get-entity-info`** — Query with ENTITY_NAME, ENTITY_NAMESPACE, and ENTITY_KIND. Record: description, tags, annotations, links, lifecycle confirmation.

2. **`get-entity-relationships`** — Query with same identifiers. Record: all `dependsOn`, `providesApi`, `consumesApi`, `ownedBy`, `partOf` relationships. Extract the list of API entity refs for Phase B.

**Fallback:** If `get-entity-info` returns empty, attempt `search-entities` with the repo name as `searchTerm`. If that also fails, log `[BACKSTAGE ENTITY NOT FOUND — skipping Step 4.5]` and continue to Step 5. Do not fabricate Backstage data.

**Note:** API entity refs are extracted from the relationship data in Phase A Step 2. No separate `find-api-specs` discovery query is needed — the relationship response provides `providesApi` and `consumesApi` refs directly.

#### Phase B — API Specs (depends on Phase A)

3. **`retrieve-api-spec`** — For each API entity ref from Step 2's `providesApi` (preferred) and `consumesApi`:
   - Retrieve at most **3 specs** total (prefer providesApi over consumesApi; prefer production lifecycle)
   - Record: spec type (openapi/asyncapi), version, endpoint count, key endpoint paths
   - Remaining APIs: list as references only (name + entityRef, no inline spec)
   - If >5 APIs total: note the count, list all names, but only inline top 3

#### Phase C — Parallel Fan-Out (no mutual dependencies, run concurrently)

All of the following queries are independent — run them in parallel:

4. **`get-techdocs`** — Query for this entity's TechDocs pages. Record: page titles, paths, content previews.

5. **`search-techdocs`** — Search across all entities for docs mentioning ENTITY_NAME. Cap at **5 results** by relevance. Summarize each in 1-2 sentences. Discard off-topic results. For deprecated entities, surface with `[MAY BE STALE]` markers.

6. **`get-github-metrics`** — Record: average merge time, open PR count, active contributors (90d), branch protection status, required review count.

7. **`get-security-metrics`** — Record: Snyk critical/high counts, Dependabot alert count, branch protection enabled.

8. **`get-datadog-metrics`** — (Skip if deprecated) Record: SLO count, monitor count.

9. **`get-pagerduty-metrics`** — (Skip if deprecated) Record: incident count (90d), MTTR, uptime percentage.

10. **`get-entity-compliance`** — Record: which metadata fields are present/missing. Determine `compliance_score`:
    - **high**: title, description, owner, and at least one of tags/TechDocs present
    - **low**: missing 2+ of the above

    Field-level confidence markers (used in Step 6):
    - Missing `description` → `[UNVERIFIED]` on § Overview only
    - Missing `owner` → `[UNVERIFIED]` on ownership references only
    - Missing `tags` → no marker
    - Relationship/API data stays clean regardless (structurally enforced)

11. **`get-repository-info`** — Record: file type breakdown, cataloging status. Cross-reference with outer workflow Step 2 (codebase read): if Backstage lists file types not found during codebase analysis, or vice versa, note as a `[CONFLICT]` in the Catalog Health Flags section. This is an internal validation step — it feeds gap questions, not draft artifacts directly.

#### Write the Snapshot

Write all collected data to `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/backstage-facts.md` following the format in `knowledge/file-format-backstage-facts.md`. Include:

- YAML frontmatter with entity identifiers, lifecycle, compliance score
- Each section with its YAML block (machine-parseable) and prose summary (human-readable)
- Catalog Health Flags section with any code↔Backstage conflicts identified

#### Degraded Mode

If Backstage tools return errors or empty results across the board:

- Log: `[BACKSTAGE UNAVAILABLE — skipping Step 4.5]`
- Mark all sections that would have received Backstage data with `[NEEDS BACKSTAGE DATA]`
- Continue to Step 5 — codebase + git + Glean findings are still available

### Step 5: Cross-Reference Decisions

For each significant technical choice visible in the code (framework, test runner, module system, key dependencies, patterns), check if git history or Glean results explain _why_. Build a decision list:

| Decision                      | Source            | Why                                            |
| ----------------------------- | ----------------- | ---------------------------------------------- |
| e.g., "Uses Mocha"            | git commit abc123 | "Predates Jest adoption; team hasn't migrated" |
| e.g., "Single-table DynamoDB" | Glean: RFC        | "Chose for query flexibility + cost at scale"  |

This list becomes the basis for ADRs in Step 6.

**Backstage-sourced decision signals:**
API spec types from Step 4.5 may indicate documentable architectural choices. For example, an AsyncAPI spec (vs OpenAPI) represents a concrete contract style decision. Only create ADR candidates for choices with clear alternatives — entity tags like `event-driven` or `lambda` describe what the service _is_, not a decision that was _made_.

### Step 6: Produce Drafts

#### Source rules for public-bound drafts (REPO_VISIBILITY=public only)

When `REPO_VISIBILITY=public`, every artifact written under `public-drafts/` follows allowlist sourcing. Internal sources read during research (Glean Confluence, Jira, Slack, internal Backstage URLs) inform understanding but **must not appear as citations in public-bound draft text**.

**Public-source allowlist** — these are the only citation forms permitted in `public-drafts/` content:

| Source type                  | Citation form                             | Example                                                                       |
| ---------------------------- | ----------------------------------------- | ----------------------------------------------------------------------------- |
| Code in this repo            | `lib/path/file.ts:LINE` or `path/file.ts` | `scripts/publish.js`, `lib/channel.ts:42`                                     |
| Git commit in this repo      | 7-char SHA + parenthetical                | `d21d1f9 (Initial commit)`                                                    |
| GitHub PR/issue in this repo | `#NNNN`                                   | `#2540`                                                                       |
| Public Contentful docs       | full URL                                  | `https://www.contentful.com/developers/docs/extensibility/app-framework/sdk/` |
| npm package metadata         | full URL                                  | `https://www.npmjs.com/package/@contentful/app-sdk`                           |
| Public package docs          | full URL                                  | `https://github.com/semantic-release/semantic-release`                        |
| Standards / specs            | full URL                                  | `https://docs.npmjs.com/policies/unpublish`                                   |

**Forbidden citation forms in public-bound drafts:**

- Glean document IDs / Confluence numeric IDs (`Confluence 4091642161`)
- Jira keys (`EXT-3593`)
- Slack channel names (`#prd-extensibility-bots`)
- Internal team handles outside `.github/CODEOWNERS`
- Internal hostnames (`*.contentful.tools`)
- Customer / company names from incident research
- PagerDuty incident IDs / postmortem titles
- Email addresses (any)
- Glean citation markers (`[Source: Glean — <thread title>]`) — internal sources don't get citations in public-bound text

**"No public source available" is a valid outcome.** Three options when an internal-only fact would otherwise be cited:

1. **Rewrite from public evidence.** The behavior is usually observable in code or commit history; cite that instead. The motivating incident does not need to ship publicly.
2. **Move the claim to an internal-only artifact.** ADRs and CLAUDE.md live under `internal-only/` and use normal sourcing rules. Internal incident context can ship there.
3. **Omit the claim.** If the claim cannot survive without internal citations and is not load-bearing, drop it.

Read the format references before writing:

- `knowledge/file-format-agents-md.md`
- `knowledge/file-format-architecture-md.md`
- `knowledge/file-format-claude-md.md`
- `knowledge/file-format-contributing-md.md`
- `knowledge/file-format-adrs.md`
- `knowledge/file-format-readme-md.md`
- `knowledge/file-format-bito.md`
- `knowledge/file-format-backstage-facts.md`

#### Source Priority Rules

When producing drafts, facts may come from multiple sources. Use this priority when sources conflict:

| Priority | Source                                        | Rationale                                              |
| -------- | --------------------------------------------- | ------------------------------------------------------ |
| 1        | Codebase (live files)                         | Ground truth — what's actually deployed                |
| 2        | Backstage catalog (from `backstage-facts.md`) | Structured, maintained, reflects intended architecture |
| 3        | Glean (Confluence/Slack/RFCs)                 | Institutional memory, may be stale                     |
| 4        | Git archaeology                               | Historical — explains "why" but not always "now"       |

**Conflict resolution:**

- Backstage claims a dependency that code doesn't reference → flag `[BACKSTAGE CLAIMS — NOT FOUND IN CODE]` and add to gap questions
- Code references a service that Backstage doesn't know about → flag `[FOUND IN CODE — NOT IN CATALOG]` and add to gap questions (Catalog Health category)
- Both become Phase 2 discussion items

**Citation:** All facts sourced from Backstage get: `[Source: Backstage — <tool-name>]`

#### Backstage Data → Artifact Mapping

Read `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/backstage-facts.md` and apply:

**AGENTS.md:**

- Entity tags/annotations → header metadata, service classification
- Relationships → Integration Points section (routing hints)
- Security metrics (critical/high Snyk) → Sharp Edges warning
- Lifecycle deprecated → top-of-file deprecation warning
- Low compliance (field-level) → relevant "verify independently" notes

**ARCHITECTURE.md:**

- Entity description → supplements § Overview (apply `[UNVERIFIED]` if compliance gap on description)
- Relationships → **primary source** for § System Context mermaid diagram (declared topology, not inferred). Still cross-reference against code imports — if code doesn't import a declared dependency, note the conflict in the diagram callout.
- API specs (top 3) → § Data Flow (contract-accurate endpoints and schemas)
- Datadog metrics → § Operational Knowledge ("N SLOs, M monitors configured")
- PagerDuty metrics → § Operational Knowledge (incident frequency, MTTR, uptime)
- TechDocs → place using keyword matching rule: match page titles to section names. "Deployment Guide" → § Operational Knowledge. "Data Model" → § Domain Concepts. Unmatched → `§ Additional Context (from TechDocs)` appendix. Summarize in 2-3 sentences with link, never inline raw content.

**README.md:**

- Entity ownership → README purpose paragraph ("owned by [team]")

**ADRs:**

- AsyncAPI vs OpenAPI spec type → ADR candidate (concrete contract style decision)

**.bito Guidelines:**

- Security metrics (active vulns) → security guideline ("Flag PRs adding deps without review — N active alerts")
- Branch protection settings → quality guideline (review enforcement)

Write draft files to scratch under paths determined by `REPO_VISIBILITY`:

| Artifact                           | Path when `REPO_VISIBILITY=public`                                         | Path when `REPO_VISIBILITY=private`                                 | Sourcing rule                                    |
| ---------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------ |
| `AGENTS.md`                        | `<SCRATCH_ROOT>/repo-cartographer/<repo>/public-drafts/AGENTS.md`          | `<SCRATCH_ROOT>/repo-cartographer/<repo>/drafts/AGENTS.md`          | Allowlist if public, normal if private           |
| `ARCHITECTURE.md`                  | `<SCRATCH_ROOT>/repo-cartographer/<repo>/public-drafts/ARCHITECTURE.md`    | `<SCRATCH_ROOT>/repo-cartographer/<repo>/drafts/ARCHITECTURE.md`    | Allowlist if public, normal if private           |
| `CONTRIBUTING.md`                  | `<SCRATCH_ROOT>/repo-cartographer/<repo>/public-drafts/CONTRIBUTING.md`    | `<SCRATCH_ROOT>/repo-cartographer/<repo>/drafts/CONTRIBUTING.md`    | Allowlist if public, normal if private           |
| `README.md`                        | `<SCRATCH_ROOT>/repo-cartographer/<repo>/public-drafts/README.md`          | `<SCRATCH_ROOT>/repo-cartographer/<repo>/drafts/README.md`          | Allowlist if public, normal if private           |
| `.bito.yaml` + `.bito/guidelines/` | `<SCRATCH_ROOT>/repo-cartographer/<repo>/public-drafts/.bito.yaml` (+ dir) | `<SCRATCH_ROOT>/repo-cartographer/<repo>/drafts/.bito.yaml` (+ dir) | Allowlist if public, normal if private           |
| `CLAUDE.md`                        | `<SCRATCH_ROOT>/repo-cartographer/<repo>/internal-only/CLAUDE.md`          | `<SCRATCH_ROOT>/repo-cartographer/<repo>/drafts/CLAUDE.md`          | Normal (internal-only path uses normal sourcing) |
| `docs/ADRs/`                       | `<SCRATCH_ROOT>/repo-cartographer/<repo>/internal-only/docs/ADRs/`         | `<SCRATCH_ROOT>/repo-cartographer/<repo>/drafts/docs/ADRs/`         | Normal                                           |

Section policy: concrete-instruction-shaped sections generate by default; generic-overview-shaped sections require specific evidence, and the justification is recorded in the run output (see `knowledge/file-format-readme-md.md` and `knowledge/research-basis.md`).

Per-artifact content rules (apply regardless of path):

- **`AGENTS.md`** — the sections in `knowledge/file-format-agents-md.md`: routing table pointing to other docs, guardrails and sharp edges, safety and permissions, and a verification command block only when the commands are not obvious from `package.json`. Apply that file's inclusion litmus test to every line. If the repo has an existing AGENTS.md, restructure into the new format while preserving all human-authored rules.

- **`ARCHITECTURE.md`** — overview, system context (with mermaid diagram), internal structure, data flows (labeled `[INFERRED]` where needed), domain concepts, key dependencies, configuration, operational knowledge. Leave operational sections as stubs with `[NEEDS TEAM INPUT]` markers where you can't determine from code.

- **`CLAUDE.md`** — Claude Code project instructions. Imperative, model-focused: commands, conventions, sharp edges, architecture pointers. Only non-discoverable content (not what linters/tsconfig already express — see `knowledge/research-basis.md` § Context files: a null result on task success). Target 100-200 lines. **Internal-only** — see `knowledge/file-format-claude-md.md`.

- **`CONTRIBUTING.md`** — the three sections in `knowledge/file-format-contributing-md.md`: conventions (global-skill references plus one fallback paragraph), specialized repo-specific procedures this repo has (with source citations), and file-level guidance. Generate nothing the disposition table in that file routes elsewhere (see `knowledge/research-basis.md`).

- **`docs/ADRs/`** — individual ADR files for each decision identified in Step 5. Use the date from the source (git commit, RFC). Write `docs/ADRs/README.md` as an index table. **Internal-only** — see `knowledge/file-format-adrs.md`.

- **`README.md`** — draft the full proposed `README.md` per `knowledge/file-format-readme-md.md`, applying its § Section Policy: existing human content verbatim, focused edits merged; full template when no README exists.

- **`.bito.yaml` + `.bito/guidelines/`** — Bito review config with three guideline files. Adapt content to repo-specific findings. The guidelines directory ships as one logical artifact. When `REPO_VISIBILITY=public`, guideline content must follow the public-source allowlist (rules can be sourced from code patterns and CI config; never cite internal incidents or postmortems).

**Inline verification gates — run these before moving to Step 7:**

1. **Scripts & commands:** For every script or command you documented, read its full content (not just the filename). Check for warning comments, deprecation notices, or conditional guards that change its meaning. If a script says "do not use in production," do not document it as a production procedure.

2. **Startup dependencies:** For each external dependency listed, check how it's initialized. Read the service bootstrap/startup code. If a missing dependency throws or calls `process.exit`, document it as a hard dependency — not graceful degradation.

3. **Entity completeness:** List every directory in the entity/model layer. Confirm each entity appears in your ARCHITECTURE.md domain concepts section. Count entities found vs. entities documented — report the delta.

4. **AGENTS.md size:** Run `wc -l` on your AGENTS.md draft. If it exceeds 100 lines, you have included discoverable content (commands, code style, CI details) that belongs in README.md (commands) or CONTRIBUTING.md (specialized procedures). Trim to routing + sharp edges only.

5. **Cross-artifact consistency:** Read your AGENTS.md, ARCHITECTURE.md, README.md, and CONTRIBUTING.md drafts together. Check for contradictions: a command referenced in AGENTS.md must exactly match the file that owns it — README.md for setup, build, and test commands, CONTRIBUTING.md for specialized-procedure commands; a dependency listed in ARCHITECTURE.md must appear in README.md getting-started prerequisites; a constraint in AGENTS.md must not conflict with a flow described in ARCHITECTURE.md. List any contradictions found and resolve them before continuing.

6. **Adversarial source tracing:** Re-read each draft with this framing: "For every factual claim, can I point to the specific file, git commit, or Glean result that proves it?" Any claim without a traceable source is either an inference (mark `[INFERRED]`) or a fabrication (delete it). Count claims checked vs. claims that survive — report the ratio.

### Step 6.5: Redaction Sweep (Public Repos Only — Hard Gate)

If `REPO_VISIBILITY=private`: skip this step. The remaining steps proceed unchanged.

If `REPO_VISIBILITY=public`: run the redaction sweep over `<SCRATCH_ROOT>/repo-cartographer/<repo>/public-drafts/` using the pattern set in `knowledge/public-redaction-patterns.md`.

Read `knowledge/public-redaction-patterns.md` before running the sweep — the file defines the patterns, the verification command, and the resolution actions for each match.

Run the verification command from `knowledge/public-redaction-patterns.md` (the "Verification Command" section). Substitute `<repo-name>` with the bare repository name (e.g., `ui-extensions-sdk`, not the clone path). The command runs `grep -rnE` over `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/public-drafts/` with all 10 patterns from the patterns table.

<HARD-GATE>
Empty stdout = pass. Non-empty stdout = hard-gate failure. Do NOT continue to Step 7 until the sweep produces zero matches in `public-drafts/`. For each match:

1. Apply false-positive exemptions per `knowledge/public-redaction-patterns.md` (e.g., team handles already in `.github/CODEOWNERS`, code constructs that look like Slack channels).
2. If the match is genuine, fix it via one of the three actions defined in the patterns file: rewrite from public-source allowlist, move to `internal-only/`, or delete.
3. Re-run the sweep. Repeat until empty stdout.
   </HARD-GATE>

After the sweep passes, record `Redaction sweep (post-Step-6): 0 matches.` for inclusion in the Phase 1 Output summary (see "Output" section at the end of this file).

### Step 7: Glean Gap Validation

**Pre-filter: Exclude Backstage-resolved gaps.**
Before running Glean gap validation, remove any candidate gaps that Step 4.5 already resolved with sourced data — but only if the resolving entity has `compliance_score: high` (check the `backstage-facts.md` frontmatter). Low-compliance entities leave their gaps on the Glean validation list — corroboration from a second source is valuable when catalog quality is uncertain.

Before declaring anything a gap, attempt to resolve it via targeted Glean searches. This step exists because broad research (Step 4) uses generic queries — now you have _specific_ unknowns to target.

**Minimum query budget: 3 variations per gap.** Each gap requires at least 3 distinct query strategies before it can be declared unresolved. If >50% of candidate gaps survive this step unchanged, re-run with broader queries (try service names, team names, related technology names) before proceeding to Step 8.

**Process:**

1. Collect all candidate gaps: sections marked `[NEEDS TEAM INPUT]`, decisions without a "why," `[INFERRED]` flows, missing operational knowledge, and unexplained technical choices.

2. For each candidate gap, run **at least 3** targeted Glean queries using different strategies:

**Strategy A — Exact terms:**

```
search: "<repo-name>" "<specific concept or decision>"
search: "<specific concept>" contentful
```

**Strategy B — Synonyms and related names:**

```
search: "<alternative name for concept>" contentful
search: "<service-that-owns-concept>" "<concept>"
search: "<team-name>" "<concept>"
```

**Strategy C — Decision/history framing:**

```
search: "<specific technology choice>" RFC OR decision OR ADR OR migration
search: "<concept>" why OR chose OR rationale OR tradeoff
```

For operational gaps, also try:

```
search: "<repo-name>" deploy OR rollback OR runbook
search: "<repo-name>" incident OR outage OR postmortem
search: "<repo-name>" monitoring OR dashboard OR alert
search: "<team-name>" on-call OR pagerduty OR incident.io
```

For cross-repo gaps:

```
search: "<repo-name>" "<downstream-service>" integration
search: "<topic-or-queue-name>" consumer OR producer
search: "<service-name>" "<repo-name>"
```

3. Use Glean chat for synthesis when search results need interpretation:

```
chat: "How does <repo-name> handle <specific operational concern> at Contentful?"
chat: "What is the relationship between <repo-name> and <other-service>?"
```

4. For each resolved gap:
   - Update the draft files with the sourced information
   - Add a citation: `[Source: Glean — <document title or thread>]`
   - Remove the `[NEEDS TEAM INPUT]` or `[INFERRED]` marker

5. For gaps that remain unresolved after Glean validation:
   - Keep the `[NEEDS TEAM INPUT]` marker
   - Note in your gap list: "Searched Glean for: `<queries tried>` — no relevant results"
   - If all 3 strategies returned nothing, the gap is confirmed unresolvable via Glean

**Exit criteria:** A candidate gap only becomes a final gap question if at least 3 query variations return no relevant results OR results are ambiguous/contradictory and need human confirmation.

### Step 7.5: Post-gap-fill redaction re-sweep (Public Repos Only — Hard Gate)

If `REPO_VISIBILITY=public`: re-run the redaction sweep from Step 6.5 over `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/public-drafts/`. Gap-fill commonly introduces `[Source: Glean — <thread title>]` citations and other internal references that must be stripped or rewritten.

If `REPO_VISIBILITY=private`: skip.

Run the verification command from `knowledge/public-redaction-patterns.md` (the "Verification Command" section). Substitute `<repo-name>` with the bare repository name.

<HARD-GATE>
Empty stdout = pass. Non-empty stdout = fix via the same three actions from Step 6.5 (rewrite from public-source allowlist, move to `internal-only/`, or delete). Do NOT proceed to Step 8 until the sweep is clean.
</HARD-GATE>

Record `Redaction sweep (post-Step-7): 0 matches.` for inclusion in the Phase 1 Output summary.

### Step 8: Compile Gap Questions

Instead of writing a GAPS.md file, compile a single batched list of gap questions organized by category. These are questions you'll present to the operator in Phase 2 (or Phase 3 depending on workflow).

Only include items that survived Step 7 (Glean Gap Validation). Each gap should note what Glean queries were attempted so the team knows you already looked.

Categories:

- **Architecture** — things about the system structure you couldn't determine
- **Domain Concepts** — entities, states, lifecycles that need human explanation
- **Operational** — deployment, monitoring, failure modes needing human knowledge
- **Decision History** — technical choices where the "why" is unclear from code + Glean
- **Cross-Repo** — external integrations, producers/consumers needing confirmation
- **Needs Confirmation** — items labeled `[INFERRED]` in the drafts that need verification
- **Catalog Health** — dependencies found in code but not declared in Backstage (`[FOUND IN CODE — NOT IN CATALOG]`), or Backstage relationships not evidenced in code (`[BACKSTAGE CLAIMS — NOT FOUND IN CODE]`). These produce `catalog-info.yaml` update recommendations rather than documentation gaps. Recommended action format: "Update catalog-info.yaml to add/remove relationship: `<entity-ref>`"

Include the gap list in your output summary for the operator.

### Accuracy Guardrails

- **Commands:** Only include commands found in `package.json` scripts, `Makefile` targets, CI config, or existing docs. Cite the source. Never invent a command.
- **Scripts:** Read the full content of any script you reference. A filename is not documentation — the script body may contain warnings, guards, or conditions that change its purpose entirely.
- **Constraints:** Only state constraints evidenced by linter config, CI checks, existing docs, or clear code patterns. List suspected-but-unconfirmed conventions as gap questions.
- **Flows:** Only describe flows you can trace statically (route → handler → service → data store). Label anything requiring runtime knowledge as `[INFERRED]`. Confirm the handler/client you trace is actively imported — not a legacy path with no callers.
- **Dependencies:** Only list external services you can find evidence of in code. Don't guess. For each, verify whether failure is graceful (try/catch with fallback) or hard (throws/exits on missing connection).
- **Dependency ≠ usage:** A package listed in `package.json` dependencies is NOT proof of active application use. For each dependency you plan to document as "actively used," verify at least one source file imports it directly (not just test files). Transitive dependencies pulled in by a framework (e.g., LaunchDarkly SDK via service-kit) are framework-internal — do not document them as application dependencies unless application code explicitly imports them.
- **Environment variables:** A variable declared in config or Terraform is NOT proof it's consumed at runtime. For each env var you document as "active," trace at least one runtime code path that reads and acts on the value. If it's declared but no code evaluates it, flag it as `[POSSIBLY DEAD CONFIG — declared but no runtime consumer found]`.
- **ADRs:** Only write ADRs for decisions with evidence. If you can't find the "why," say so in the Decision section — never fabricate reasoning. Do not quote individuals from informal channels (Slack) without noting that the source is informal and may not represent a team decision.
- **Literalness:** Agents consuming these docs are literal — they follow instructions exactly as written. Avoid qualifiers like "usually," "typically," "may," "often," or "sometimes" unless you specify the exact condition. Replace "typically uses X" with "uses X when [condition]; uses Y otherwise." Replace "may fail" with "fails when [condition]." If you don't know the condition, mark it `[NEEDS TEAM INPUT]` rather than hedging.

### Verification Gate

<HARD-GATE>
Do NOT report Phase 1 as complete without running these commands and confirming their output. Evidence before claims.
</HARD-GATE>

```bash
# Resolve the drafts directory based on REPO_VISIBILITY (set in Step 0 sub-step 5).
# Public: artifacts split between public-drafts/ (shippable) and internal-only/ (CLAUDE, ADRs).
# Private: all artifacts under drafts/.
if [ "$REPO_VISIBILITY" = "public" ]; then
  DRAFTS_DIR=<SCRATCH_ROOT>/repo-cartographer/<repo-name>/public-drafts
  ADRS_DIR=<SCRATCH_ROOT>/repo-cartographer/<repo-name>/internal-only/docs/ADRs
else
  DRAFTS_DIR=<SCRATCH_ROOT>/repo-cartographer/<repo-name>/drafts
  ADRS_DIR=<SCRATCH_ROOT>/repo-cartographer/<repo-name>/drafts/docs/ADRs
fi

# 1. All expected artifacts exist (public set; private adds CLAUDE.md and docs/ADRs/ under drafts/)
ls -la "$DRAFTS_DIR/AGENTS.md" \
       "$DRAFTS_DIR/ARCHITECTURE.md" \
       "$DRAFTS_DIR/CONTRIBUTING.md" \
       "$DRAFTS_DIR/README.md" \
       "$DRAFTS_DIR/.bito.yaml"

# 2. Line counts (AGENTS.md should be <100 lines)
wc -l "$DRAFTS_DIR"/*.md

# 3. ADR count (under internal-only/ for public, drafts/ for private)
ls "$ADRS_DIR"/*.md 2>/dev/null | wc -l

# 4. Gap markers remaining
grep -rc "NEEDS TEAM INPUT\|INFERRED\|NEEDS BACKSTAGE DATA" <SCRATCH_ROOT>/repo-cartographer/<repo-name>/ | grep -v ":0$"

# 5. Backstage snapshot exists (if Step 4.5 ran)
ls -la <SCRATCH_ROOT>/repo-cartographer/<repo-name>/backstage-facts.md 2>/dev/null

# 6. Public-repo scratch layout (only if REPO_VISIBILITY=public)
test -d <SCRATCH_ROOT>/repo-cartographer/<repo-name>/public-drafts && \
  test -d <SCRATCH_ROOT>/repo-cartographer/<repo-name>/internal-only && \
  ! test -d <SCRATCH_ROOT>/repo-cartographer/<repo-name>/drafts && \
  echo "layout=public-ok" || echo "layout=mismatch"

# 7. Redaction sweep clean (only if REPO_VISIBILITY=public)
# Run the verification command from knowledge/public-redaction-patterns.md against
# <SCRATCH_ROOT>/repo-cartographer/<repo-name>/public-drafts/ and report sweep=clean if empty.
# Non-empty output means redaction matches remain — fail closed.

# 8. Private-repo scratch layout (only if REPO_VISIBILITY=private)
test -d <SCRATCH_ROOT>/repo-cartographer/<repo-name>/drafts && \
  ! test -d <SCRATCH_ROOT>/repo-cartographer/<repo-name>/public-drafts && \
  ! test -d <SCRATCH_ROOT>/repo-cartographer/<repo-name>/internal-only && \
  echo "layout=private-ok" || echo "layout=mismatch"
```

**Fail conditions (fix before proceeding):**

- Any expected artifact missing → go back to Step 6
- AGENTS.md > 100 lines → trim discoverable content to README.md (commands) or CONTRIBUTING.md (specialized procedures)
- Zero ADRs → re-check Step 5 decision table (every repo has at least one)
- Backstage snapshot missing when catalog-info.yaml exists → re-run Step 4.5
- `layout=mismatch` → wrong scratch layout for the detected `REPO_VISIBILITY`. Re-check Step 6 path routing (public should produce `public-drafts/` + `internal-only/`; private should produce `drafts/` only).
- `sweep=DIRTY` → public-bound drafts contain forbidden patterns. Re-run Step 6.5 / 7.5 fixes until the verification command from `knowledge/public-redaction-patterns.md` returns empty stdout.

### Step 9: Write Run State

Write `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/run-state.md` per `knowledge/run-state.md` (path idiom matches the prompts' existing knowledge-file citations), with `workflow: seed`, `completed_phase: phase-1` and `next_phase: phase-2`, `repo_head` from `git -C <TARGET_REPO> rev-parse HEAD`, `repo_visibility` set to the detected `REPO_VISIBILITY` value, `gap_count` from the counting command in `knowledge/run-state.md` (this is the baseline Phase 2's verification gate compares against), `updated` set to today's date in `YYYY-MM-DD` format, and `## Remaining work` bullets carrying forward every deferred item from this phase (unresolved gaps, `[DEFERRED TO ASYNC]` markers, DEFERRED conflicts, unopened PR).

### Output

When complete, summarize:

1. Coverage audit: what existed before, what's being added
2. File paths and approximate length of each draft, plus the justification for any generic-overview section generated
3. Number of ADRs generated and their titles
4. Glean Gap Validation results: how many candidate gaps were resolved via Glean vs. how many remain
5. How many final gap questions per category (post-validation)
6. Which ARCHITECTURE.md sections are stubs waiting for team input
7. Any surprises or notable findings from codebase + research
8. Flagged tooling items (.npmrc, .nvmrc, packageManager missing)
9. Redaction sweep results when `REPO_VISIBILITY=public` (post-Step-6 and post-Step-7); skipped for private repos
