# README.md — File Format Reference

`README.md` is the repo's front door — the first file both a human and an agent open. It owns orientation: what this repo is and why it exists, how to get it running, how to run its tests, where the other docs live, and where an agent goes first.

Keep it small, concrete, and front-loaded. Every section either states a command or routes the reader to another file.

## Ownership

| README owns                                           | README points at, never duplicates                                                                     |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Purpose and why the repo exists, and who owns it      | System structure, data flows, dependencies → `ARCHITECTURE.md`                                         |
| Getting started: prerequisites, clone, install, build | Specialized repo-specific procedures → `CONTRIBUTING.md`                                               |
| Running tests                                         | Commit, branch, and pull-request conventions → the global `contentful-*` skills, via `CONTRIBUTING.md` |
| The documentation map                                 | Agent guardrails, sharp edges, invariants → `AGENTS.md`                                                |
| Agent routing: one pointer to `AGENTS.md`             | Why decisions were made → `docs/ADRs/`                                                                 |

Content owned by another artifact is pointed to, never restated. The cross-doc conflict scan in `prompts/phase-3-quality-check.md` Step 2.6 flags a conflict when the same claim appears in two files with different wording, and an unresolved conflict fails Phase 3.

## Sections (in order)

### 1. Title + purpose

The repo name as an `# H1`, then one paragraph: what this repo is, who owns it (the `spec.owner` group from `catalog-info.yaml`), and why it exists. One paragraph, not two.

### 2. Getting Started

Prerequisites first, as a table of required tools with versions and where the version comes from:

| Tool                | Version                    | Notes                     |
| ------------------- | -------------------------- | ------------------------- |
| Node.js             | `<version>` (see `.nvmrc`) | Use `nvm use` to switch   |
| `<package-manager>` | `<version>`+               | `<how to install/enable>` |

Then the exact clone → install → build sequence. Every command carries a source citation comment:

```bash
git clone git@github.com:contentful/<repo-name>.git
cd <repo-name>
<install-command>   # source: package.json → packageManager
<build-command>     # source: package.json → scripts.build
```

Additional setup that the commands above require — registry tokens, `.env` values, Docker containers — is stated here with the condition that triggers it: "`<command>` requires `<dependency>`; start it with `<command>`."

### 3. Running Tests

Exact commands, each sourced:

```bash
<test-command>          # source: package.json → scripts.test
<single-test-command>   # source: package.json → scripts.test
<watch-command>         # source: package.json → scripts.test:watch
```

State the test framework and the infrastructure a suite requires: "Integration tests require `<dependency>` running; start it with `<command>`."

### 4. Documentation Map

A table routing to the artifacts this repo has. Omit the row for any artifact the repo does not have.

| Document                               | What it covers                                     |
| -------------------------------------- | -------------------------------------------------- |
| [ARCHITECTURE.md](./ARCHITECTURE.md)   | Internal structure, data flows, integration points |
| [AGENTS.md](./AGENTS.md)               | Agent-facing routing table and guardrails          |
| [CONTRIBUTING.md](./CONTRIBUTING.md)   | Specialized repo-specific procedures               |
| [Architecture Decisions](./docs/ADRs/) | Why things look the way they do                    |

### 5. For AI Agents

Two sentences, verbatim:

```markdown
## For AI Agents

If you are an AI coding agent working in this repository, read [AGENTS.md](./AGENTS.md) first. It tells you where to find architectural context, development setup, decision records, and repo-specific rules.
```

## Section Policy

The evidence behind this policy is recorded in `knowledge/research-basis.md`.

- **Concrete-instruction-shaped sections generate by default.** Quick start, running tests, constraints, and routing all carry a command the reader runs or a destination the reader opens. The five sections above are all of this shape.
- **Generic-overview-shaped sections are omitted.** "Repo at a glance" summaries, architecture-overview prose, and feature tours restate what the code and `ARCHITECTURE.md` already carry. Generate one only when specific evidence justifies it — for example, the repo has no `ARCHITECTURE.md` and no other artifact tells a reader what the components are. When you generate one, record the justification in the run output summary.
- **Owned content is routed, not copied.** A command, constraint, or convention that `AGENTS.md` or `CONTRIBUTING.md` owns appears in README as a link to that file.

## Existing README.md

When the repo already has a `README.md`, the cartographer proposes direct, focused edits to it, delivered as a reviewable PR diff. **The PR is the consent mechanism** — the team approves or rejects each change in review, so the tool does not need a separate draft-only artifact or an in-file marker.

Rules:

1. Draft the **full proposed `README.md`** in the drafts directory (path per `prompts/phase-1-discovery.md` Step 6). The draft is the complete file, not a fragment and not a patch.
2. Preserve existing human-authored content verbatim. Existing wording that is still accurate is copied character-for-character.
3. Merge new and changed sections at the position the section list above assigns them.
4. No wholesale rewrite. No reordering of human-authored content without team approval — raise the reordering as a Phase 2 question and apply it only after the team agrees.
5. No provenance markers. The file carries no "generated by", no "added by the cartographer", and no diff or conflict markers.
6. Before the draft is copied into the target repo, diff it against the repo's current `README.md` and confirm no human-authored content was dropped (`prompts/phase-3-quality-check.md` Step 4).

## Missing README.md

When the repo has no `README.md`, generate the full file from the section list above:

````markdown
# <project-name>

<One paragraph: what this repo is, who owns it, why it exists.>

## Getting Started

| Tool                | Version                    | Notes                     |
| ------------------- | -------------------------- | ------------------------- |
| Node.js             | `<version>` (see `.nvmrc`) | Use `nvm use` to switch   |
| `<package-manager>` | `<version>`+               | `<how to install/enable>` |

```bash
git clone git@github.com:contentful/<repo-name>.git
cd <repo-name>
<install-command>   # source: package.json → packageManager
<build-command>     # source: package.json → scripts.build
```

## Running Tests

```bash
<test-command>          # source: package.json → scripts.test
<single-test-command>   # source: package.json → scripts.test
```

<Framework name and any infrastructure a suite requires.>

## Documentation Map

| Document                               | What it covers                                     |
| -------------------------------------- | -------------------------------------------------- |
| [ARCHITECTURE.md](./ARCHITECTURE.md)   | Internal structure, data flows, integration points |
| [AGENTS.md](./AGENTS.md)               | Agent-facing routing table and guardrails          |
| [CONTRIBUTING.md](./CONTRIBUTING.md)   | Specialized repo-specific procedures               |
| [Architecture Decisions](./docs/ADRs/) | Why things look the way they do                    |

## For AI Agents

If you are an AI coding agent working in this repository, read [AGENTS.md](./AGENTS.md) first. It tells you where to find architectural context, development setup, decision records, and repo-specific rules.
````
