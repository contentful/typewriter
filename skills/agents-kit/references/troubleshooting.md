# Troubleshooting Reference

Use this reference when Agents Kit commands fail or generated output looks wrong.

## Package Source Cannot Resolve

Symptom:

```text
Failed to resolve package source "<package>" from <path>. Is it installed?
```

Checks:

- package is listed in `dependencies` or `devDependencies` where `package.json` config lives
- package manager install has run
- user-scope packages are installed under `~/.config/agents-kit`, not the current repo
- provider bundle dependencies include every `package:<name>` re-export
- provider packages with `exports` expose `./package.json` when consumers need `agentsKit.*.provides` metadata

## Directory Source Fails

Checks:

- path is relative to the config root
- path is inside the repo/config boundary
- single skill roots contain `SKILL.md`
- multi-skill containers have `package.json` with `agentsKit.skills.provides`
- provided roots contain `SKILL.md`

## Duplicate Install Names

Install names come from `SKILL.md` frontmatter `name` after normalization.

Fix by:

- renaming one skill's frontmatter `name`
- removing one source
- avoiding package and local skill roots that intentionally produce the same install name unless override behavior is desired

Always rerun:

```sh
pnpm exec contentful-agents-kit skills install --dry-run
```

## Source/Target Overlap

Do not place source skills under native target roots:

- `.agents/skills`
- `.cursor/skills`
- `.claude/skills`

When using symlink mode with `--shared-root ./skills`, local source skills may live under `./skills/<skill-name>`. Do not use `./skills` itself as a skill root.

## Publishability Failures

`skills validate`, `hooks validate`, and `rules validate` use npm publish rules to determine published files. The validation does not require an `npm` executable.

Fix by checking:

- `package.json.files` includes `SKILL.md`, `HOOK.yaml`, rules files, and supporting directories
- `.npmignore` does not exclude needed files
- generated files are present before validation
- provider roots match `agentsKit.*.provides`

## Stale Generated Output

If installed skills are stale:

```sh
pnpm exec contentful-agents-kit skills install --dry-run
pnpm exec contentful-agents-kit skills install
```

Commit:

- `package.json`
- generated shared root or copied target folders
- native target symlinks or copied skill folders

If generated docs are stale in `agents-kit`:

```sh
pnpm docs:generate-skills
pnpm docs:validate
```

## Symlink Issues

Version 2 repository installs require directory symlink support. On Windows,
enable Developer Mode or grant `SeCreateSymbolicLinkPrivilege`, and ensure Git
checkouts that contain links use `core.symlinks=true`. If the environment
cannot preserve links, use version 2 explicit copy mode or keep the consumer on
version 1. Agents Kit does not silently fall back to copying.

If links are broken:

- rerun `skills install --dry-run` and inspect link targets
- rerun `skills install`
- user-scope installs use absolute targets and migrate older relative links;
  rerun the install after upgrading if native agent directories are symlinked
  into a dotfiles repository
- avoid hand-editing generated native target folders
- confirm the shared root still contains each installed skill

## GitHub Skill Source Issues

For user-scope GitHub sources:

- confirm URL points to `github.com/contentful/*` or `github.com/contentful-labs/*`
- confirm the URL path contains a valid skill root
- for private HTTPS sources, set `GH_TOKEN`, `GITHUB_TOKEN`,
  `GITHUB_PACKAGES_READ_TOKEN`, or `GH_PACKAGES_TOKEN`; authentication failures
  retry the remaining configured tokens in that order
- try `--github-ssh-fallback` if HTTPS credentials fail in an interactive terminal

```sh
contentful-agents-kit skills install --scope user --github-ssh-fallback
```

## Command Routing Mistakes

- use `skills install` for skills
- use `hooks install` for hooks
- use `rules install` for rules
- use `skills update` only for user-scope skill updates
- use provider validation from the provider package root
