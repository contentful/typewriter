# Newcomer Simulation Rubric — 5 Buckets

Each question in the newcomer simulation receives exactly one verdict. Verdicts are not graded on a curve — they apply objectively against the rules below.

## Verdicts

### ANSWERED-WITH-CITATION

The draft files contain a clear, accurate answer **and** the agent recorded a `source: <file>:<heading-or-line>` reference pointing to the exact location.

Example:

- Question: "How do I run the tests?"
- Verdict: `ANSWERED-WITH-CITATION`
- Source: `CONTRIBUTING.md:Testing`
- Notes: "Section says `pnpm test` and `pnpm test:e2e`."

### ANSWERED-NO-CITATION

The draft files contain a clear, accurate answer but no citation was recorded, OR the citation does not point to the actual location of the answer.

The information exists but the agent could not (or did not) prove it. Receives reduced weight (0.7) — does not auto-fail the question. Auto-applies when the agent answers from memory without a verified source.

Example:

- Question: "What's the deploy process?"
- Verdict: `ANSWERED-NO-CITATION`
- Source: (none)
- Notes: "Answer is in CONTRIBUTING.md but I described it from memory without verifying the section."

### PARTIAL

The draft files contain _some_ relevant information but the answer is incomplete, ambiguous, or scattered across multiple files in a way that requires the newcomer to assemble it.

Example:

- Question: "What's the request lifecycle for the primary operation?"
- Verdict: `PARTIAL`
- Source: (none)
- Notes: "ARCHITECTURE.md mentions handler and DB but skips the auth middleware step."

### MISLEADING

The draft files contain a confident-sounding answer that is **wrong** or would lead a newcomer to take a harmful action.

This is the most dangerous verdict. It auto-fails the question AND the category, regardless of other scores.

**Test:** "Would a newcomer who trusts this claim do something they'd have to undo?" If yes, `MISLEADING`. If they'd just be confused or have nothing to act on, `MISSING` or `PARTIAL`.

Examples:

- ARCHITECTURE.md says service X uses Postgres; actual code uses MySQL.
- CONTRIBUTING.md mentions `npm test` but actual command is `pnpm test`.
- README.md describes a deprecated queue that's still partially in use without saying it's deprecated.
- AGENTS.md routes to a file that no longer exists.

### MISSING

The draft files contain no information that addresses the question.

Distinct from `MISLEADING`: missing means the newcomer is left with a question, but won't act on bad information.

Example:

- Question: "What does an alert on the queue-depth alarm mean and what do I do?"
- Verdict: `MISSING`
- Source: (none)
- Notes: "No alerting/runbook content in any draft file."

## Citation format

A `source:` reference for `ANSWERED-WITH-CITATION` uses the format `<file-path>:<anchor>` where anchor is one of:

- A markdown heading text (no leading `#`, preserve hyphens and spaces) — e.g., `CONTRIBUTING.md:Testing`, `docs/ADRs/0003-queue.md:Decision`
- A single line number — e.g., `ARCHITECTURE.md:42`
- A line range — e.g., `ARCHITECTURE.md:42-58`

File paths are repo-relative.

## Decision rules

1. **Citation auto-downgrade:** If the agent claims `ANSWERED-WITH-CITATION` but the citation does not resolve, the verdict drops based on what failed:
   - **Cited file or section does not exist** → `MISLEADING` (fabricated source — the agent invented an authoritative-looking pointer that has no anchor in reality).
   - **Cited section exists but does not contain the claimed answer** → `ANSWERED-NO-CITATION` (incomplete grounding — record the attempted citation in `Source:` and explain the mismatch in `Notes:` so the gap is auditable).
2. **MISLEADING is sticky:** A single MISLEADING verdict auto-fails the question AND the category. Do not roll MISLEADING into PARTIAL to keep aggregate scores high.
3. **PARTIAL has no citation requirement.** It's already a partial credit; demanding citations on partial answers creates noise.
4. **MISSING vs N/A:** A question is `MISSING` if the docs _should_ answer it but don't. `N/A` applies at the **category level only** — never per-question. See `phase-3-quality-check.md` Step 3 for the N/A escape-hatch procedure (justification + audit logging).

## Scoring weights

**This file is the canonical source for the rubric, weights, and per-category gates.** `phase-3-quality-check.md` Step 3 may restate this table for ergonomics — if they diverge, this file wins.

For per-category score calculations:

| Verdict                  | Weight                                         |
| ------------------------ | ---------------------------------------------- |
| `ANSWERED-WITH-CITATION` | 1.0                                            |
| `ANSWERED-NO-CITATION`   | 0.7                                            |
| `PARTIAL`                | 0.4                                            |
| `MISLEADING`             | 0.0 (auto-fails the question AND the category) |
| `MISSING`                | 0.0                                            |

**Per-category gate semantics:**

- **Working Safely** uses a _min-score gate_: every question must score ≥ 0.7 (i.e., no MISSING, no MISLEADING, no PARTIAL). One soft-failed question fails the category.
- **Operating in Production** uses an _aggregate gate_: weighted mean across the category's questions ≥ 0.80.
- **All other categories** use an _aggregate gate_: weighted mean ≥ 0.70.

## What changed (vs. PASS / PARTIAL / FAIL)

The old 3-bucket rubric collapsed two distinct failure modes — "missing" and "actively wrong" — into the same FAIL bucket. The 5-bucket rubric separates them so misleading claims, the most dangerous failure, are surfaced as their own category-failing verdict instead of being lost in aggregate.
