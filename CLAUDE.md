# Typewriter

This repository is the Contentful-owned fork of Segment Typewriter. It exists primarily to prevent bulk AI action performance from degrading under repeated runtime schema validation. Ownership is `group:team-core-ai` (`catalog-info.yaml`).

## Primary Fork Invariant

Generated analytics-js and analytics-node clients must reuse compiled AJV validators by schema `$id`.

- Keep the AJV instance and `Map<string, ValidateFunction>` at module scope.
- Never move `new Ajv()` or `ajv.compile()` into the per-event validation path.
- Compile only on a cache miss or when the schema has no `$id`.
- Keep cache and `onViolation` behavior equivalent in `src/languages/templates/typescript/analytics-js.hbs` and `src/languages/templates/typescript/node.hbs`.
- Keep AJV behind the `isDevelopment` template guard; production output must not include it.
- Review both generated SDK snapshots after changing either template.

Read [ARCHITECTURE.md](./ARCHITECTURE.md) for the fork-specific flow. The inherited CLI architecture is supporting context, not the main maintenance focus.

## Commands

| Task | Command | Non-obvious requirement |
| --- | --- | --- |
| Install | `yarn install --frozen-lockfile --ignore-engines` | Match the GitHub Actions install path; Agents Kit is dev-only tooling with a newer Node requirement. |
| Regenerate telemetry | `yarn build:telemetry` | Rewrites the generated `src/telemetry/segment.ts` client. |
| Publish | `npm publish` | Ask first; this publishes a restricted GitHub Packages release. |

Use Node 18 from `.nvmrc`. CI also covers Node 20 and 21 (`.github/workflows/ci.yml`). Run Agents Kit commands under a compatible newer Node runtime; do not raise Typewriter's runtime requirement just for repository tooling.

## Upstream Boundaries

Consult the upstream reference in [ARCHITECTURE.md](./ARCHITECTURE.md) only when a change must cross the narrow fork boundary.

Respect these boundaries:

- Keep oclif lifecycle and command dispatch in `src/commands/` and `src/base-command.ts`.
- Keep Segment transport and plan normalization in `src/api/`.
- Keep target-language behavior behind the `LanguageGenerator` interface.
- Add SDK-specific source through templates or a target renderer; do not mix it into command orchestration.
- Keep Contentful fork changes narrow and separately reviewable from inherited Segment behavior; avoid unrelated upstream refactors.

## Conventions

- Use branches shaped like `<type>/<JIRA-key>-<description>`.
- Preserve the existing mixed quote style unless the touched file's formatter or linter normalizes it.
- Do not restyle unrelated inherited code.

## Generated Content

- Never hand-edit a generated analytics client.
- Treat `src/telemetry/segment.ts` as generated.
- Regenerate repository telemetry through `yarn build:telemetry`.
- Preserve the exact generated-file warning from `src/commands/build.ts`.
- Ensure every new generator emits that warning near the start of every output file.
- Do not delete `plan.json` during output cleanup.
- Do not broaden cleanup to recursively delete directories without a separately reviewed safety design.

## Tracking Plan Compatibility

- Preserve support for local `plan.json` operation without a token.
- Preserve fallback to the local plan when an update request fails.
- Preserve v7 `rs_` Tracking Plan ID migration unless intentionally making a breaking change.
- Keep config updates opt-in; `BaseCommand.finally()` asks before writing migrated IDs.
- Validate `typewriter.yml` changes in both the TypeScript type and Joi schema.
- Keep Tracking Plan schema sanitization isolated in `src/api/trackingplans.ts`.

## Tokens and External Effects

- Never commit a Segment API token or GitHub token.
- Ask before running a real `--update`; it can rewrite local plan data.
- Ask before running `npm publish`.
- Keep token resolution order stable unless migration is explicitly designed: piped input, configured token script, then global token file.
- Do not log token values. Existing debug output in `BaseCommand` is a security-sensitive sharp edge; do not copy that pattern.
- Treat user-configured scripts as arbitrary shell execution.
- Keep the five-second script timeout unless a documented use case justifies changing it.

## Sharp Edges

- Build cleanup identifies generated files by header content, not by filename.
- Removing or changing the header can leave stale files in consumer repositories.
- `start` chooses init or build solely from whether workspace config was loaded.
- Development and production commands delegate to `Build.run()` with an injected mode.
- A missing token is allowed for cached generation but not for initialization.
- Telemetry uses distinct development and production write keys.
- Telemetry errors are intentionally non-blocking.
- JavaScript generation delegates to TypeScript generation and transpiles the returned output.
- React Native TypeScript output changes the extension to `.tsx`.
- Swift output enforces `Codable`.
- Kotlin injects its analytics import outside the template to preserve import order.
- Rules without properties receive a null placeholder before generation.

## Testing

- Run `yarn test`; its `posttest` hook runs lint.
- For validator-cache changes, inspect analytics-js and analytics-node snapshots and confirm the emitted cache lookup occurs before `ajv.compile()`.
- Confirm production snapshots contain no AJV import, instance, or validator cache.
- Snapshot tests cover each supported language/SDK pair.
- Generated typedef tests require build artifacts; `yarn test` runs their build step first.
- Test helpers remove and recreate paths under `test-env/`.
- Snapshot changes must be reviewed as generated public API changes, not accepted mechanically.
- When changing a language generator, verify every supported SDK for that generator.
- When changing shared Quicktype utilities, verify all generator snapshots and typedef tests.
- Do not run update flows with production credentials as a substitute for fixtures.

## Documentation

- Keep setup and test commands in README.md.
- Keep deep data flows and dependencies in ARCHITECTURE.md.
- Keep specialized code-generation and release procedures in CONTRIBUTING.md.
- Record architecture-significant reversals by superseding an ADR rather than rewriting accepted history.
