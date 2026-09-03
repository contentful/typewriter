# PR Body Reference

Use this reference only after confirming the repository does not provide a PR template.

## Template Repositories

- Preserve the repository template headings.
- Fill each section with concrete changes, motivation, and rollout notes.
- Mark checkboxes truthfully only for completed items.

## Standard Body

If no template exists, use:

```markdown
## Summary

<brief overview of what changed and what it enables>

## Ticket

<ticket URL; derive from key when only key is available>

## Why

<why this change is needed — one or two sentences on the root motivation, not a description of what the code does>

<for each non-obvious tradeoff or deliberate choice: one bullet naming the choice and the reason. Omit if there are none.>
<for each obvious alternative that was tried and ruled out: one bullet naming it and why it didn't work. Omit if there are none.>
<examples: "- Skipped retry on 404 — callers treat a missing resource as terminal", "- Tried a simple in-memory cache first — caused stale reads under concurrent load">

## Risk and Rollout

<risk level, rollout/rollback notes>

> Note: This PR was created with the [contentful-github-create-pull-request](https://github.com/contentful/agents-kit/tree/main/libs/contentful-github-create-pull-request) skill, powered by [Agents Kit].
> Learn more: [Agents Kit Documentation](https://contentful.roadie.so/docs/default/component/agents-kit).
```

## Ticket Resolution

- If the user provides a full Jira URL, use it as-is.
- If the user provides only a Jira key like `ABC-123`, render the Ticket section as `https://contentful.atlassian.net/browse/ABC-123`.
- If both a Jira key and a Jira URL are available, prefer the explicit URL for the PR body.
- The ticket key is resolved earlier in the workflow before this step; do not re-run the lookup here.

## Required Ticket Pattern

Keep PR title validation unchanged. The required ticket key pattern remains:

```text
\[[A-Z][A-Z0-9]+-\d+\]
```
