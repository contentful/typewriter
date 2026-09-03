# CONTRIBUTING.md — File Format Reference

`CONTRIBUTING.md` owns **specialized repo-specific procedures only** — the steps an engineer cannot get from `README.md`, from the repo's own config files, or from the global `contentful-*` skills.

Two rules govern everything below:

- **No rule tables.** Command catalogues, code-style tables, and CI/CD job tables restate what `package.json`, the linter configs, and the workflow files already carry. They are not generated.
- **No duplication of README content.** Getting started and running tests live in `README.md`. Point at them; never repeat them.

A repo with no specialized procedure gets a short `CONTRIBUTING.md`: the Conventions section, plus File-Level Guidance when the repo has restricted paths.

## Sections (in order)

### 1. Conventions

One line per convention — commit format, branch naming, and pull-request workflow.

Commit format and pull-request workflow point at the global `contentful-*` skill that owns them: `$contentful-git-commit` owns commit format, `$contentful-github-create-pull-request` owns the pull-request workflow.

Confirm each skill exists before naming it: list the installed skills and look for that exact name among the `contentful-*` entries. When a skill is absent, do not emit its bullet. Write the convention inline instead, from repo evidence — commit shape from `git log --oneline -20`, pull-request target from branch protection — in the same one-line-per-convention shape the other bullets use. Name the absent skill in the run output summary so the team knows the pointer was dropped.

Branch naming has no owning skill, so its line states the shape this repo uses — the prefix pattern read from `git log --oneline -20`, or the pattern branch protection enforces when the repo enforces one. This line is unconditional: it is the only place a reader finds the branch shape. When the repo evidences no pattern, omit the line rather than guess one.

Then exactly one fallback paragraph, for environments where those skills are not installed. It states the commit subject shape and the pull-request target branch in prose. State only what the repo evidences — read `git log --oneline -20` for commit shape, `.github/pull_request_template.md` for PR requirements, and branch protection settings for the target branch. Name that evidence in the sentence that states the convention. The paragraph covers only the bullets that point at a skill: a bullet that already states its convention inline is not repeated there, and when neither skill was found the paragraph is omitted.

```markdown
## Conventions

- **Commits** — follow `$contentful-git-commit`.
- **Branches** — `<prefix pattern observed in git log --oneline -20>`.
- **Pull requests** — follow `$contentful-github-create-pull-request`.

Without those skills: commits use `<format observed in git log --oneline -20>`, and pull requests target `<protected branch>`.
```

### 2. Specialized Procedures

Only the procedures this repo actually has. Generate a subsection per procedure; omit the whole section when the repo has none. Candidates:

- **Migrations** — how to write, run, and roll back a migration
- **Terraform / infrastructure changes** — plan/apply flow, which environments, what requires review
- **Code generation** — which files are generated, which command regenerates them, what breaks when they are hand-edited
- **Release process** — only when releases are non-obvious (a manual step, a release branch, a two-repo sequence). A repo whose releases run from CI on merge gets no release subsection.

Every command in this section carries a source citation comment and the context it requires:

```bash
<migration-command>   # source: package.json → scripts.migrate
```

The test for including a procedure: an engineer who has read `README.md` and the repo's config files still cannot perform this task correctly. Anything that fails that test is not generated (see `knowledge/research-basis.md`).

### 3. File-Level Guidance

Which paths are sensitive and which files are generated and must not be hand-edited. This section is kept because the reason a path is restricted is non-derivable — the path list is visible, the "why" is not.

| Path     | Why restricted |
| -------- | -------------- |
| `<path>` | `<reason>`     |

## Disposition of the Old Sections

Phase 1 generates only the three sections above. Many existing `CONTRIBUTING.md` files carry more than that; this table states where each of those sections belongs now, so a run never generates it here:

| Old section                       | Disposition                                                                                                                                                                                                             |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Prerequisites                     | → `README.md` § Getting Started                                                                                                                                                                                         |
| Getting Started                   | → `README.md` § Getting Started                                                                                                                                                                                         |
| Testing                           | → `README.md` § Running Tests                                                                                                                                                                                           |
| Commands (catalogue)              | Not generated — discoverable from `package.json` scripts, `Makefile` targets, and CI config (see `knowledge/research-basis.md`). The install, build, and test commands live in `README.md`; nothing else is catalogued. |
| Code Style & Conventions          | Not generated — discoverable from the linter, Prettier, and `tsconfig` configs (see `knowledge/research-basis.md`)                                                                                                      |
| CI/CD                             | Not generated — discoverable from `.github/workflows/` and `.circleci/config.yml` (see `knowledge/research-basis.md`)                                                                                                   |
| Commit Convention                 | → § Conventions: the `$contentful-git-commit` line plus the fallback paragraph                                                                                                                                          |
| Branch Strategy & Release Process | Branch strategy → § Conventions. Release process → § Specialized Procedures when releases are non-obvious; omitted otherwise.                                                                                           |
| Pull Requests                     | → § Conventions: the `$contentful-github-create-pull-request` line plus the fallback paragraph                                                                                                                          |
| Development Workflow              | Generated only when the repo has a specialized, non-obvious procedure — as a subsection of § Specialized Procedures. Omitted otherwise.                                                                                 |
| File-Level Guidance               | Kept — § File-Level Guidance                                                                                                                                                                                            |
