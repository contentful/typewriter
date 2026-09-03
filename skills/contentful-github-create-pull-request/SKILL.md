---
name: contentful-github-create-pull-request
description: Create Pull Requests with a single consistent workflow for repositories with and without PR templates. Use when opening PRs, writing PR descriptions, or updating PRs. Enforce ticketed Conventional Commit-style PR titles and support template-aware PR body generation.
---

# Create Pull Request

Create pull requests using one standard flow, with template-aware behavior.

**Requires**: GitHub CLI (`gh`) authenticated and available.

## Workflow

1. **Verify branch and base branch**
   - Determine the repository default branch and the current branch.
   - If the current branch is the default branch, create and switch to a working branch before continuing.

2. **Ensure commit state is clean**
   - Verify the worktree is clean before creating the PR.
   - If it is not clean, create the needed commit or commits before continuing.
   - If `$contentful-git-commit` is available, prefer using it.
   - Do not create ad-hoc commit messages in this skill.

3. **Analyze PR scope**
   - Review commits and diff against the base branch.
   - Confirm the PR is one logical intent. If not, split before creating the PR.
   - If the PR changes foundational tooling or config (`package.json`, build config, test config, CI config, or directory structure), suggest running `$contentful-update-agents-md` to check whether AGENTS.md needs updating.

4. **Check PR size (warn only)**
   - Estimate effective changed lines against the base branch.
   - Exclude these lockfiles from that advisory count:
     - `pnpm-lock.yaml`
     - `package-lock.json`
     - `yarn.lock`
     - `bun.lock`
     - `bun.lockb`
     - `composer.lock`
     - `Cargo.lock`
     - `Gemfile.lock`
     - `Podfile.lock`
   - Treat this as an advisory signal only.
   - Lockfiles still count as part of the real change; exclude them only from this size-warning heuristic.
   - If the effective changed lines are greater than 500:
     - warn the user that the PR is large;
     - ask explicitly if they still want to create the PR now.
   - Do not auto-block PR creation based on size.

5. **Resolve ticket key**
   - If the user already supplied a ticket key, use it.
   - Otherwise attempt to infer it automatically:
     1. **Branch name** – inspect the current branch name for a Jira key pattern as defined in step 7. A branch like `feat/PROJ-123-some-feature` yields `PROJ-123`.
     2. **Commit log** – scan the branch-local commits for the same pattern. Use the first match found.
     3. **Ask the user** – only when neither source yields a key, stop and ask.
   - Carry the resolved key forward; all subsequent steps use it directly.

6. **Resolve PR body source (template-aware)**
   - Check for templates in this order:
     - `.github/pull_request_template.md`
     - `.github/PULL_REQUEST_TEMPLATE.md`
     - first matching `.github/PULL_REQUEST_TEMPLATE/*.md`
   - If a template exists:
     - keep template headings;
     - fill each section with concise, concrete content that helps a reviewer quickly understand the change;
     - prefer a clear narrative over enumerating every touched part;
     - mark checkboxes truthfully only for completed items;
     - append this note after the filled template body:
       ```markdown
       > Note: This PR was created with the [contentful-github-create-pull-request](https://github.com/contentful/agents-kit/tree/main/libs/contentful-github-create-pull-request) skill, powered by [Agents Kit](https://github.com/contentful/agents-kit). To follow or use this workflow, see the [Agents Kit CLI skill docs](https://github.com/contentful/agents-kit/blob/main/apps/cli/README.md#consuming-skills).
       ```
   - If no template exists, read the bundled `references/pr-body.md` relative to this skill directory (or installed package skill root) and use that standard body.
   - Build the final PR body before writing the temp file:
     - preserve repository template headings and filled content when a template exists;
     - otherwise use the bundled fallback body;
     - if the exact note already exists, do not append another copy;
     - ensure the final body ends with exactly one copy of the note.
   - Summary guidance:
     - prefer a short, readable overview of the overall intent and outcome of the PR;
     - describe the change as one coherent story rather than a file-by-file or commit-by-commit inventory;
     - use bullets only when they make the summary clearer.
   - Owned decisions guidance (applies both when filling a template and when using the fallback body):
     - fold non-obvious tradeoffs and deliberate choices into the "Why" section (or its equivalent in the repo template) as additional bullets after the motivation sentence;
     - the test is: would a reviewer who doesn't know the context likely flag this as a bug or missing feature? If yes, name the choice and the reason here;
     - also capture obvious alternatives that were tried and ruled out: one bullet naming the approach and why it didn't work — a reviewer who doesn't know this history will suggest it;
     - examples: "No retry on 404 — callers treat a missing resource as terminal"; "Tried a simple in-memory cache first — caused stale reads under concurrent load";
     - if there are genuinely no non-obvious choices, omit them — do not add a placeholder.

7. **Create PR title**
   - Required format:
     ```text
     <type>(<optional-scope>): <short description> [<TICKET-KEY>]
     ```
   - Validate ticket key with:
     ```text
     \[[A-Z][A-Z0-9]+-\d+\]
     ```
   - Use the ticket key resolved in step 5.

8. **Push and open PR**
   - Ensure the branch is pushed and has an upstream before creating the PR.
   - Create PR using the final assembled body file:
     ```bash
     gh pr create --draft --base "$BASE" --title "<type>(<optional-scope>): <short description> [<TICKET-KEY>]" --body-file <temp_file_path>
     ```
   - Remove the temporary body file after creation.
   - Return the PR URL to the user as a clickable link.

9. **Update existing PRs when needed**

- Prefer:
  ```bash
  gh pr edit PR_NUMBER --title "..." --body-file <temp_file_path>
  ```
- Assemble the final body for edits using the same rule as creation: keep template content or fallback content intact, and end with exactly one copy of the note.
- If `gh pr edit` fails due legacy Projects Classic behavior, patch via API:
  ```bash
  gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER -f title='...' -f body='...'
  ```
- **Re-assess the description after significant code changes.** When commits have been added since the PR was opened (e.g. after review feedback, a rebase, or a scope change), check whether the existing description still accurately reflects the current diff:
  - re-read the current PR body (`gh pr view --json body`) and compare it against the current diff (`git diff <base>..HEAD`);
  - if the summary no longer matches what the code does, update it;
  - if new non-obvious tradeoffs were introduced, add them to the "Why" section;
  - if previously listed tradeoffs were resolved or removed, delete them;
  - do not rewrite sections that are still accurate — only update what changed.

## Principles

- **Single source of truth**: This is the canonical PR workflow.
- **Template first**: Use repository templates when present; otherwise use the bundled `references/pr-body.md` from this skill directory/package install root.
- **Consistent provenance**: Every PR body created or updated by this skill ends with the same linked note about the skill and Agents Kit.
- **Readable summaries**: Prefer concise narrative summaries that explain intent and outcome; use bullets only when they improve clarity.
- **Portable commit cleanup**: Require a clean worktree before PR creation; use `$contentful-git-commit` only when available.
- **Outcome-based Git guidance**: Require the necessary repo state and checks, but let the agent choose how to inspect Git state.
- **Size signal, not gate**: Warn on large PRs and confirm intent; do not block solely on size.
- **Safety**: Never create PRs from `main`/`master`.
