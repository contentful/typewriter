# Refresh Mode — Context Drift Detection

Adopt the Refresh persona: **documentation auditor**. Your job is to find where the docs have drifted from the code — not to validate that docs are still correct. Assume drift has occurred since the last update and look for evidence of it. A "no drift found" result means you didn't look hard enough at areas that change frequently (dependencies, routes, config, CI).

## Checklist

Create a TodoWrite task for each item. Complete in order.

1. Confirm preconditions (existing context files, catalog-info.yaml)
2. Run full discovery (Steps 1–4.5 from Phase 1)
3. Diff against existing files (severity classification)
4. Backstage delta comparison (if prior snapshot exists)
5. Produce drift report
6. PR-eligible allowlist gate (Step 4, fix mode)
7. Apply fixes and create PR (fix mode only)
8. Write run state

## Inputs

- `TARGET_REPO`: repo name (e.g., `extensibility-api`)
- `MODE`: `report` (default) | `fix`
  - `report`: produce a drift report only
  - `fix`: produce report AND open a PR with suggested updates

## Instructions

### Step 0: Preconditions

Confirm before proceeding:

1. Target repo is cloned at `<REPOS_ROOT>/<TARGET_REPO>`. If not, stop and report: "Repo not found. Clone it first."
2. At least one Golden Context file exists in the target repo (AGENTS.md, ARCHITECTURE.md, or CONTRIBUTING.md). If none exist, stop and report: "No existing context files to diff against. Run Phase 1-3 first to seed initial context."
3. **MCP preflight.** Check MCP server health via the host's MCP listing. On Claude Code, run `claude mcp list` or `claude mcp get <name>`; the output reports `✓ Connected` or `✗` for each server. On other MCP-supporting hosts, use the equivalent listing command.
   - **Glean MCP (required — hard stop):** Look for a server whose name matches `glean` (case-insensitive substring) with status `Connected`. If no Glean server is configured, or the configured server is not Connected, STOP immediately and report: `"Glean MCP unavailable (not configured or not connected). The Repo Cartographer requires Glean to source institutional knowledge from Confluence, Slack, RFCs, and design docs — without it the 'Source Everything' and 'Exhaust Glean Before Declaring Gaps' principles cannot be satisfied. Resolve Glean access before re-running."` Do not proceed.
   - **Backstage MCP (degraded if unavailable):** Look for a server whose name matches `backstage` (case-insensitive substring) with status `Connected`. If no Backstage server is configured or it's not Connected, set `BACKSTAGE_AVAILABLE = false` and apply the "Backstage unavailable" degraded mode from `agent.md`. Otherwise `BACKSTAGE_AVAILABLE = true`.

   Report the preflight result inline: `Preflight: Glean=ok, Backstage=<available|degraded>`. Continue with reduced Backstage scope per `agent.md` Degraded Mode if needed.

4. `catalog-info.yaml` exists in the target repo root (required for Backstage queries). Extract entity identifiers per Phase 1 Step 0.

5. **Visibility detection.** Run from inside the cloned repo:

   ```bash
   cd "<TARGET_REPO>" && gh repo view --json isPrivate -q '.isPrivate'
   ```

   The command outputs the literal string `true` or `false`. Map the trimmed value:
   - output `false` → `REPO_VISIBILITY=public`
   - output `true` → `REPO_VISIBILITY=private`

   If the command fails, STOP. Resolve `gh auth status` before re-running. Falling back to a default would risk leaking internal content into a public repo.

   `REPO_VISIBILITY` is referenced by Step 1 (refresh write paths), Step 2 (drift report scope), and Step 4 (fix-mode behavior + PR-body sweep).

### Step 1: Run Discovery

Follow the same codebase reading and research procedure as Phase 1 (`prompts/phase-1-discovery.md`, Steps 1-4.5 and Steps 5-7). This includes Step 4.5 (Backstage Enrichment), the Glean Gap Validation step, **and the redaction sweep (Steps 6.5 and 7.5)** when `REPO_VISIBILITY=public`.

Refresh write paths follow the same `REPO_VISIBILITY` branching as Phase 1, but under the `-refresh` suffix:

| `REPO_VISIBILITY` | Layout                                                                                                                                          |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `public`          | `<SCRATCH_ROOT>/repo-cartographer/<repo-name>-refresh/public-drafts/` and `<SCRATCH_ROOT>/repo-cartographer/<repo-name>-refresh/internal-only/` |
| `private`         | `<SCRATCH_ROOT>/repo-cartographer/<repo-name>-refresh/drafts/`                                                                                  |

The Backstage snapshot is written to `<SCRATCH_ROOT>/repo-cartographer/<repo-name>-refresh/backstage-facts.md` regardless of visibility.

### Step 2: Diff Against Existing Files

Compare the fresh drafts against the current context files in the target repo. The set of files to diff depends on `REPO_VISIBILITY`:

When `REPO_VISIBILITY=public`, diff only the public-allowed artifacts in the target repo against the fresh `public-drafts/`:

- `<REPOS_ROOT>/<repo-name>/AGENTS.md`
- `<REPOS_ROOT>/<repo-name>/ARCHITECTURE.md`
- `<REPOS_ROOT>/<repo-name>/CONTRIBUTING.md`
- `<REPOS_ROOT>/<repo-name>/README.md`
- `<REPOS_ROOT>/<repo-name>/.bito.yaml` and `.bito/guidelines/`

`CLAUDE.md` and `docs/ADRs/` in the target repo (if present from a pre-public-handling era) are flagged as Error severity with the suggested fix "Remove from target repo — public-repo policy disallows these artifacts. If the file contains content worth preserving, relocate it to the internal-only Golden Context layout per phase-1 Step 6's path routing." Do not diff their content.

When `REPO_VISIBILITY=private`, diff all canonical artifacts as before:

- `<REPOS_ROOT>/<repo-name>/AGENTS.md`
- `<REPOS_ROOT>/<repo-name>/ARCHITECTURE.md`
- `<REPOS_ROOT>/<repo-name>/CLAUDE.md`
- `<REPOS_ROOT>/<repo-name>/CONTRIBUTING.md`
- `<REPOS_ROOT>/<repo-name>/docs/ADRs/`
- `<REPOS_ROOT>/<repo-name>/README.md`
- `<REPOS_ROOT>/<repo-name>/.bito.yaml` and `.bito/guidelines/`

For each difference found, classify by severity:

| Severity    | Criteria                                                         | Examples                                                                                                                                                                                |
| ----------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Error**   | Context is actively wrong — would mislead an engineer or AI tool | Command references a script that no longer exists; constraint references a deleted config; file path points to renamed/removed directory; ADR marked Accepted but decision was reversed |
| **Warning** | Context is incomplete — new things exist that aren't documented  | New dependency not in README.md; new entity directory not in ARCHITECTURE.md; new external integration not in dependencies section; decision with no ADR                                |
| **Info**    | Non-critical inconsistencies                                     | Section ordering doesn't match format reference; minor naming drift; version number outdated but still functional                                                                       |

### Step 2.5: Backstage Delta

Compare the fresh Backstage snapshot against the previous one to detect catalog-level drift.

**Inputs:**

- Fresh: `<SCRATCH_ROOT>/repo-cartographer/<repo-name>-refresh/backstage-facts.md`
- Previous: `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/backstage-facts.md`

If no previous snapshot exists (first run with Backstage integration), skip this step and note: "No prior Backstage snapshot — delta comparison not possible. Current snapshot establishes baseline."

**Comparison method:** Section-by-section YAML block comparison (not raw text diff):

- Lists: set diff (identify added and removed items)
- Scalars: equality comparison
- Prose blocks: ignored (informational only)
- Operational section (Datadog/PagerDuty): **NOT compared** — these are point-in-time metrics, not structural catalog data

**Categorize changes:**

| Change Type              | Detection                                                     | Severity | Report Action                                                  |
| ------------------------ | ------------------------------------------------------------- | -------- | -------------------------------------------------------------- |
| Relationship added       | New item in `dependencies`, `provides_api`, or `consumes_api` | Warning  | "New dependency/API `X` declared — not reflected in docs"      |
| Relationship removed     | Item missing from previous snapshot                           | Error    | "Dependency/API `Y` removed — still documented"                |
| Lifecycle changed        | `lifecycle` field differs in frontmatter                      | Error    | "Lifecycle changed from `A` to `B` — triggers doc restructure" |
| Ownership changed        | `owned_by` or frontmatter `owner` differs                     | Warning  | "Ownership transferred — update README.md and AGENTS.md"       |
| API spec changed         | `specs_retrieved` entries differ (endpoint count, version)    | Warning  | "API contract changed — data flow section may be stale"        |
| Security posture changed | `snyk_critical`/`snyk_high` increased                         | Warning  | "New critical vulnerabilities — update AGENTS.md sharp edges"  |
| Compliance regressed     | `compliance_score` went from high to low                      | Info     | "Catalog metadata quality decreased"                           |

**Code↔Backstage conflicts** (from Catalog Health Flags section):
These are never auto-fixed — the fix is a catalog update or code change, not a doc edit. In `MODE=report`, list them under a dedicated "Catalog Update Recommendations" section with specific actions:

- `found_in_code_not_catalog`: "Recommend adding relationship to catalog-info.yaml: `<detail>`"
- `found_in_catalog_not_code`: "Recommend removing stale relationship from catalog-info.yaml, or investigate if dependency was accidentally removed from code: `<detail>`"

In `MODE=fix`, these are included in the PR description as "Catalog health recommendations (manual action needed)" — not applied as file changes.

**Merge into drift report:** Backstage Delta findings are merged with the doc↔code drift from Step 2. Use the same severity classifications (Error/Warning/Info). Backstage Delta items are tagged `[Source: Backstage Delta]` in the report table.

#### Degraded Mode

If Backstage is unavailable during refresh:

- Skip Step 2.5 entirely
- Add to drift report summary: "Backstage unavailable — catalog drift not assessed this run"
- Continue to Step 3 with doc↔code findings only

### Step 3: Produce Report

```markdown
**Context Drift Report — <repo-name>**
**Date:** YYYY-MM-DD

## Summary

- Errors: N
- Warnings: N
- Info: N

## Errors

| File | Section | Finding | Suggested Fix |
| ---- | ------- | ------- | ------------- |

## Warnings

| File | Section | Finding | Suggested Fix |
| ---- | ------- | ------- | ------------- |

## Info

| File | Section | Finding |
| ---- | ------- | ------- |
```

### Step 4: Fix Mode (only if MODE=fix)

**Auto-fix scope for Backstage-sourced drift:**

| Safe to auto-fix                                 | Requires human review                |
| ------------------------------------------------ | ------------------------------------ |
| New dependency added to mermaid diagram          | Dependency _removed_ from diagram    |
| Ownership name update in README.md               | Lifecycle transition (any direction) |
| Security alert count update in AGENTS.md         | API contract breaking changes        |
| New contributor added to active contributor list | Compliance regression                |

**Public-repo redaction gate (REPO_VISIBILITY=public only).** Before applying fixes to the target repo, run the verification command from `knowledge/public-redaction-patterns.md` over `<SCRATCH_ROOT>/repo-cartographer/<repo-name>-refresh/public-drafts/` to confirm no new leaks are about to ship.

<HARD-GATE>
Empty stdout = pass. Non-empty stdout = fix the matches before applying any patches to the target repo. The PR cannot be opened with redaction violations in the proposed changes.
</HARD-GATE>

Code↔Backstage conflicts (Catalog Health items) are NEVER auto-fixed. They appear in the PR description as recommendations only.

If invoked with `--fix`:

1. Apply suggested fixes — all context files are team-owned, so all sections are candidates for update
2. Use `git blame` to identify who last edited a section. If a section was recently hand-edited by a team member, flag it for review in the PR description; do not overwrite hand-edited content.
3. Check if any new decisions warrant ADRs — create them
4. **Allowlist gate.** Before committing, list every path about to be committed (`git status --porcelain` on the branch). Check each against `agent.md` § PR-Eligible Files for the detected `REPO_VISIBILITY`.

   <HARD-GATE>
   A deletion-only status (`D ` or ` D`) for a path that is not on the eligible list is permitted when the proposed tree removes that forbidden artifact. Any status that adds, modifies, copies, or renames a path must still match the eligible list; a rename is not deletion-only. For every non-deletion status, remove an ineligible path from the branch and re-check. GAPS.md, backstage-facts.md, decisions.md, run-state.md, *.append.md, and internal-only/ content may be deleted but never added or modified in the PR.
   </HARD-GATE>

5. **Commit each file change separately using the `$contentful-git-commit` workflow:**
   - Resolve the ticket key once (from branch name, commit log, or user input) and use it for all commits
   - Format: `<type>(<optional-scope>): <description> [<TICKET-KEY>]`
   - One commit per file for reviewability
6. **Create the PR using the `$contentful-github-create-pull-request` workflow:**
   - Branch: `chore/context-refresh`
   - Title format: `chore: refresh Golden Context for <repo-name> [<TICKET-KEY>]`
   - Resolve the ticket key from branch name, commit log, or ask the user
   - Check for a repo PR template and fill it if present
   - Body should include: drift report summary (error/warning counts), list of changes made, and recently-hand-edited sections flagged as "needs manual review"
   - **Public-repo PR-body sweep (REPO_VISIBILITY=public only — warn-and-confirm).** Before invoking `gh pr create`, render the proposed PR body to a temp file and run the verification command from `knowledge/public-redaction-patterns.md` against it. Empty stdout: proceed to `gh pr create`. Non-empty stdout: print the matches and ask the operator `"These patterns matched in the proposed PR body. Proceed anyway? (y/N)"`. Default is no — the operator must explicitly type `y` to proceed. If they decline, the operator has three options: (a) edit the body to remove or rewrite the matches, re-run the sweep, and re-prompt; (b) abandon the PR creation if the body cannot be made safe; (c) override on a legitimate false positive by typing `y` at the prompt. This is **warn-and-confirm**, not a hard gate (per `agent.md` "PR-body strictness (universal)") — the operator is in the loop.
   - Always create as `--draft`
   - Return the PR URL for visibility

### Step 5: Write Run State

Write `<SCRATCH_ROOT>/repo-cartographer/<repo-name>-refresh/run-state.md` per `knowledge/run-state.md` (path idiom matches the prompts' existing knowledge-file citations), with `workflow: refresh`, `completed_phase: refresh` and `next_phase: none`, `repo_head` from `git -C <TARGET_REPO> rev-parse HEAD`, `repo_visibility` set to the detected `REPO_VISIBILITY` value, `gap_count` from the counting command in `knowledge/run-state.md`, `updated` set to today's date in `YYYY-MM-DD` format, and `## Remaining work` bullets carrying forward every deferred item from this phase (unresolved gaps, `[DEFERRED TO ASYNC]` markers, DEFERRED conflicts, unopened PR).

### GitHub Check Mode (optional, future)

When wired as a CI check, this mode:

1. Runs Steps 1-4 only (never auto-fixes)
2. Posts the report as a PR comment
3. Returns exit code 0 (info/warnings) or 1 (errors found) for CI gating
