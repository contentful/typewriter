# Phase 2 — Interactive Enrichment Session

Adopt the Phase 2 persona: **technical interviewer**. Your job is to ask precise questions, listen carefully, and write the answers into documentation exactly as given — not paraphrased, not embellished, not inferred beyond what was stated. When a team member's answer contradicts your Phase 1 findings, trust the human and update every artifact where the stale claim appears.

## Checklist

Create a TodoWrite task for each item. Complete in order.

1. Confirm preconditions (drafts exist, context owner identified)
2. Present Phase 1 summary to team
3. Section-by-section review (all draft files)
4. Gap filling: Domain Concepts
5. Gap filling: Operational Knowledge
6. Gap filling: Decision History & Cross-Repo
7. Run verification gate and summarize session outcomes
8. Write run state

## Prerequisites

- Phase 1 must be complete. Drafts exist at `$DRAFTS_DIR/` (resolved per `REPO_VISIBILITY` — see Preconditions below).
- Team has reviewed the Phase 1 drafts before the session.
- Target repo is cloned at `<REPOS_ROOT>/<repo-name>`.

## Inputs

- `TARGET_REPO`: repo name (e.g., `extensibility-api`)
- `CONTEXT_OWNER`: name of the designated context owner for this repo

## Preconditions

Before starting, confirm:

1. **Resolve `DRAFTS_DIR` from `REPO_VISIBILITY`** (set during Phase 1 Step 0 sub-step 5):
   - **`REPO_VISIBILITY=public`** → `DRAFTS_DIR=<SCRATCH_ROOT>/repo-cartographer/<TARGET_REPO>/public-drafts`
   - **`REPO_VISIBILITY=private`** → `DRAFTS_DIR=<SCRATCH_ROOT>/repo-cartographer/<TARGET_REPO>/drafts`

   If `REPO_VISIBILITY` is not set in the current session (e.g., Phase 2 invoked without prior Phase 1 context), re-run the visibility detection from `prompts/phase-1-discovery.md` Step 0 sub-step 5 before continuing.

2. Phase 1 drafts exist at `$DRAFTS_DIR/`. If not, stop and report: "Phase 1 drafts not found at `$DRAFTS_DIR/`. Run Phase 1 first."
3. Target repo is cloned at `<REPOS_ROOT>/<TARGET_REPO>`. If not, stop and report: "Target repo not cloned."
4. `CONTEXT_OWNER` is provided. If not, ask: "Who is the designated context owner for this repo?"

## Session Structure (60 minutes)

### Opening (5 min)

Present a summary of what Phase 1 produced:

- Coverage audit: what existed before, what's being added
- Number of ADRs generated
- Number and categories of gap questions to work through
- Any surprises from codebase + Glean research

Ask: "Before we start filling gaps — did anyone spot errors in the Phase 1 drafts? Let's correct those first."

### Section-by-Section Review (15 min)

Walk through each draft file. For each section:

1. Present what the agent wrote
2. Ask: "Is this accurate? Anything to add, correct, or remove?"
3. If the team wants to replace a section with their own wording, do it
4. If the team approves with edits, update the content

Pay special attention to:

- **AGENTS.md** — are the sharp edges correct and complete?
- **README.md** — are the getting-started and test commands accurate? **CONTRIBUTING.md** — are the specialized procedures right?
- **ADRs** — is the "why" correct? Any ADRs missing?

### Gap Filling: Domain Concepts (15 min)

Work through the Domain Concept gaps. For each entity/concept:

- "I see `<Entity>` in the codebase. Can someone walk me through its lifecycle?"
- "Are there business rules or invariants that aren't obvious from the code?"
- "What are the gotchas — things that surprise people the first time?"

Write answers directly into ARCHITECTURE.md section 5 (Domain Concepts).

### Gap Filling: Operational Knowledge (15 min)

Work through the Operational gaps:

- "What are the known failure modes for this service?"
- "Walk me through a deploy — process, what can go wrong, how do you roll back?"
- "Which dashboards does the team watch? What do healthy vs. unhealthy look like?"
- "If `<external dependency>` goes down, what's the blast radius?"

Write answers into ARCHITECTURE.md section 8 (Operational Knowledge).

### Gap Filling: Decision History & Cross-Repo (10 min)

Decision history:

- "This repo uses `<technology X>` instead of `<common alternative Y>`. What's the backstory?"
- "Are there known tech debt items or things you'd do differently today?"
- Any decisions without ADRs — create new ADR files for decisions the team can explain.

Cross-repo relationships:

- "I see this service publishes to `<topic>`. Who consumes this? What's the contract?"
- "If this service goes down, what downstream services are affected?"

Write answers into ARCHITECTURE.md and create additional ADR files as needed. For cross-repo findings, also draft a delivery chain file for `knowledge/delivery-chains/` using the schema in `knowledge/delivery-chain-schema.md`.

## Investigation Protocol

When a gap question or team correction leads to investigating a specific technology, library, or configuration:

1. **Check recent git history for that file/library first:**

   ```bash
   git log --oneline -20 -- <relevant-files-or-directories>
   git log --oneline --all --grep="<library-name>" | head -10
   ```

   Recent PRs (last 30 days) often explain current state better than Glean.

2. **Trace actual usage, not just declaration:**
   - A dependency in `package.json` → grep for imports in `src/` or `lib/`
   - An env var in config → grep for reads in application code (not just the config file)
   - A script in `package.json` → read the script body or target file to understand what it actually does

3. **Distinguish 3 states:**
   - **Active:** imported + called in runtime code paths
   - **Test-only:** imported only in test files
   - **Dead:** declared/configured but zero runtime consumers

4. **When a correction invalidates a Phase 1 claim:** Update all artifacts where the claim appears (it likely propagated to ARCHITECTURE.md, AGENTS.md, and possibly an ADR). Don't fix one file and leave the stale claim in another.

## File Handling Rules

- **Never delete human-authored content.** If existing content is being replaced, the team must explicitly approve it during this session.
- **No provenance markers.** Do not add HTML comments or other markers to generated content. Once merged, all content is team-owned and hand-maintained.
- **Real-time writing.** Update the files in `$DRAFTS_DIR/` as the session progresses (and `<SCRATCH_ROOT>/repo-cartographer/<TARGET_REPO>/internal-only/` for CLAUDE.md and `docs/ADRs/` when `REPO_VISIBILITY=public`). After each major section, confirm: "I've written that into [file], section [N]. Moving on."

## Closing (5 min)

1. Summarize what was captured: gaps filled, ADRs created, sections completed, any remaining items
2. List deferred items — add these as `[NEEDS TEAM INPUT]` markers in the relevant files
3. Confirm: "I'll run the quality check (Phase 3) and then open a PR for review. [Context owner] will approve."

## Verification Gate

<HARD-GATE>
Do NOT report Phase 2 as complete without running these checks. Evidence before claims.
</HARD-GATE>

```bash
# DRAFTS_DIR was resolved in Preconditions step 1.
# Phase 2 may also touch internal-only/ when REPO_VISIBILITY=public; SESSION_ROOT covers both.
SESSION_ROOT=<SCRATCH_ROOT>/repo-cartographer/<TARGET_REPO>

# 1. Draft files were updated (not just read)
git -C "$SESSION_ROOT" diff --stat 2>/dev/null || \
  find "$DRAFTS_DIR/" -newer "$DRAFTS_DIR/AGENTS.md" -name "*.md"

# 2. Gap markers reduced against the Phase 1 baseline.
#    PHASE1_GAPS is the gap_count Phase 1 wrote; CURRENT_GAPS uses the same counting
#    command, defined in knowledge/run-state.md.
PHASE1_GAPS=$(grep '^gap_count:' "$SESSION_ROOT/run-state.md" | awk '{print $2}')
CURRENT_GAPS=$(grep -ro "NEEDS TEAM INPUT\|INFERRED\|NEEDS BACKSTAGE DATA\|DEFERRED TO ASYNC" \
  --exclude=run-state.md "$SESSION_ROOT/" | wc -l)
echo "gaps: phase-1=$PHASE1_GAPS now=$CURRENT_GAPS"

# 3. No contradictions introduced: every command AGENTS.md names must also appear in
#    the file that OWNS it — README.md (setup/build/test) or CONTRIBUTING.md
#    (specialized procedures: migration/terraform/codegen/release-style commands). A
#    match in the non-owning file does not count — check each command against its own
#    owner. "generate" alone is too broad (matches routine commands like `prisma
#    generate`); require it to co-occur with a codegen-shaped term (schema/type/client/
#    sdk/proto/graphql/code) before treating it as a specialized procedure.
#    The extraction captures the full backtick span first (no character-class
#    restriction on the first token), then classifies it by token count — capped
#    at 5 space-separated tokens, well past any real command — rather than
#    excluding characters up front. That keeps scoped-package commands like
#    `pnpm add -D @contentful/agents-kit` or `npx @contentful/agents-kit init
#    --scope user` visible to the check instead of silently skipping any span
#    whose first token starts with `@` or `/`. The second grep drops spans
#    containing a common English stopword, so ordinary-prose backtick spans
#    (e.g. `set the flag`) don't get misread as commands and trigger a spurious
#    UNMATCHED below.
grep -oE '`[^`]+`' "$DRAFTS_DIR/AGENTS.md" \
  | grep -viE '[ `](a|an|and|are|as|at|be|by|for|from|in|is|it|of|on|or|the|to|with)[ `]' \
  | awk -F'`' '{n = split($2, tok, " "); if (n >= 1 && n <= 5) print}' \
  | sort -u | while read -r cmd; do
  if echo "$cmd" | grep -qiE 'migrat|terraform|codegen|release|generat.*(schema|types?|client|sdk|proto|graphql|code)'; then
    owner="CONTRIBUTING.md"
  else
    owner="README.md"
  fi
  grep -qF "$cmd" "$DRAFTS_DIR/$owner" || echo "UNMATCHED ($owner): $cmd"
done
```

**Fail conditions (fix before proceeding):**

- Check 1 lists no updated draft file → the session's answers were not written to disk. Re-apply them to `$DRAFTS_DIR/` before closing.
- `$SESSION_ROOT/run-state.md` is absent, or carries no `gap_count` → Phase 1 did not complete its final step. Re-run `prompts/phase-1-discovery.md` Step 9 against the Phase 1 drafts, then re-run this check.
- `now` is greater than `phase-1` → the session recorded more open items than it closed. List every marker Phase 1 did not carry. Delete each one that no session answer or new ADR justifies. **Clears when** the closing summary lists every remaining net-new marker as a question the team could not answer (or a marker on an ADR created this session), and states the net increase and its cause. A justified marker stays in the drafts, so the count itself does not have to come down.
- `phase-1` minus `now` does not equal the gaps the closing summary reports as filled, minus the justified net-new markers from the condition above → markers were cleared without a recorded answer, or answers were written without clearing their marker. Reconcile the drafts against the session notes before closing.
- Any `UNMATCHED (owner):` line → AGENTS.md names a command its owning file does not carry, even if the command appears in the _other_ file (that does not satisfy the ownership split). Move the command into the file named as owner — README.md § Getting Started or § Running Tests for setup/build/test commands, CONTRIBUTING.md § Specialized Procedures for migration/terraform/codegen/release-style commands — or drop it from AGENTS.md (see `knowledge/file-format-agents-md.md` — AGENTS.md never owns commands).

**Confirm before closing:**

- List how many gaps were filled vs. how many remain
- List any new ADRs created during the session
- Confirm all team corrections were propagated to every artifact where the stale claim appeared

## Early Exit

If the session ends before all gaps are addressed:

1. Summarize what was captured so far
2. Mark all remaining gaps as `[DEFERRED TO ASYNC]` in the draft files
3. List specific async follow-up questions for the team to answer via Slack or doc comments
4. Phase 3 can still run — the simulation will surface the remaining gaps as PARTIAL/FAIL scores

## Write Run State

Write `<SCRATCH_ROOT>/repo-cartographer/<repo-name>/run-state.md` per `knowledge/run-state.md` (path idiom matches the prompts' existing knowledge-file citations), with `workflow: seed`, `completed_phase: phase-2` and `next_phase: phase-3`, `repo_head` from `git -C <TARGET_REPO> rev-parse HEAD`, `repo_visibility` set to the `REPO_VISIBILITY` value established in the preconditions, `gap_count` from the counting command in `knowledge/run-state.md` (the same count check 2 of the verification gate produced), `updated` set to today's date in `YYYY-MM-DD` format, and `## Remaining work` bullets carrying forward every deferred item from this phase (unresolved gaps, `[DEFERRED TO ASYNC]` markers, DEFERRED conflicts, unopened PR).
