# Agent Guidance

| What you need | Where to look |
| --- | --- |
| Purpose, setup, and tests | [README.md](./README.md) |
| Structure and data flows | [ARCHITECTURE.md](./ARCHITECTURE.md) |
| Specialized procedures | [CONTRIBUTING.md](./CONTRIBUTING.md) |
| Decision rationale | [docs/ADRs/](./docs/ADRs/) |
| PR review rules | [.bito/guidelines/](./.bito/guidelines/) |

## Guardrails

- Preserve cached AJV validator reuse; it is the reason Contentful maintains this fork and protects bulk AI action performance. Never move `new Ajv()` or `ajv.compile()` into the per-event validation path. Keep cache lookup, compilation, and error behavior equivalent in `analytics-js.hbs` and `node.hbs` ([fork architecture](./ARCHITECTURE.md)).
- Use Yarn for repository work. Follow the install command in README.md and the specialized generation procedures in CONTRIBUTING.md.
- Preserve Typewriter's Node 18 runtime support even though Agents Kit requires a newer Node version. Repository installs intentionally ignore dependency engine declarations; run Agents Kit commands under a compatible newer runtime instead of raising Typewriter's runtime requirement.
- Treat `skills/` and `.agents/skills/` as Agents Kit-managed output. Do not hand-edit installed package skills or generated symlinks; change the configured package sources and reinstall through Agents Kit.
- Do not hand-edit generated analytics clients. Regenerate them from their Tracking Plan; generated files carry the header defined in [the build command](./src/commands/build.ts).
- Keep the generated-file header in every generator output. Cleanup only removes files containing that header, so removing it leaves stale output behind ([build command](./src/commands/build.ts)).
- Treat [the telemetry client](./src/telemetry/segment.ts) as generated output. Follow the regeneration procedure in CONTRIBUTING.md.
- Preserve v7 tracking-plan ID compatibility unless the migration path is intentionally removed. The base command converts legacy IDs in memory and asks before writing config ([base command](./src/base-command.ts)).
- Keep Contentful-specific behavior separate from inherited Segment behavior. The primary delta is cached AJV validation; restricted package publishing distributes it internally (README.md; commits c1ad641 and f9206de).

## Safety and Permissions

- Ask before publishing; publication changes the restricted `@contentful/typewriter` package (package.json, README.md).
- Ask before refreshing Tracking Plans with real credentials; refreshes contact Segment and rewrite local plan files ([Tracking Plan loader](./src/api/trackingplans.ts)).
- Never commit Segment or GitHub tokens. Token resolution supports stdin, a `typewriter.yml` token script, and the global token file; `.npmrc` reads `GITHUB_TOKEN` from the environment.
- Preserve user-authored files in generation directories. Cleanup is intentionally limited to files with the Typewriter-generated header ([build command](./src/commands/build.ts)).
