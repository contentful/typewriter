# Architecture

## Why the Contentful Fork Exists

The Contentful fork exists to protect bulk AI action performance. These actions can validate many analytics messages in one operation. Upstream Typewriter generated clients created an AJV instance and compiled the event schema during every runtime validation, repeating expensive compilation work for every event.

Contentful changed the generated analytics-js and analytics-node clients to reuse compiled validators. This is the fork's primary architecture and the main behavior maintainers must preserve. The fork is owned by Core AI and published as the restricted `@contentful/typewriter` package.

## Fork-Specific Runtime Architecture

The implementation lives in:

- `src/languages/templates/typescript/analytics-js.hbs`
- `src/languages/templates/typescript/node.hbs`

Both templates generate the same cache structure inside their development/runtime-validation output:

```text
module load
  -> create Ajv({ allErrors: true, verbose: true })
  -> create Map<string, ValidateFunction>

event validation
  -> read schema.$id
  -> cache hit: reuse the compiled validator
  -> cache miss: ajv.compile(schema)
       -> schema has $id: cache the validator
       -> schema has no $id: use it once without caching
  -> validate the event
  -> forward validation errors to the existing onViolation handler
```

The AJV instance and validator map are module-level state in the generated client. They are not recreated for each event. The Handlebars `isDevelopment` guard keeps AJV and runtime validation out of production output.

### Why This Matters for Bulk AI Actions

Without the cache, schema compilation scales with the number of validated events. A bulk action validating the same event type repeatedly would pay the same compilation cost on every message. With the cache, the first event compiles the schema and later events reuse the resulting `ValidateFunction`.

No benchmark number is asserted here; the architectural requirement is qualitative and strict: per-event schema recompilation would severely degrade bulk AI action performance and defeat the purpose of this fork.

## Invariants

Changes to generated runtime validation must preserve all of the following:

1. `new Ajv()` must remain outside the per-event validation function.
2. `ajv.compile()` may run only on a cache miss or when a schema has no `$id`.
3. A validator cached under a schema `$id` must be reused for subsequent events with that ID.
4. analytics-js and analytics-node must implement equivalent caching and error behavior.
5. Validation failures must continue through the existing `onViolation` handler.
6. Production output must not import or instantiate AJV.
7. Snapshot changes for both templates must be reviewed as runtime and performance changes, not accepted mechanically.

Changing schema IDs, cache identity, or template placement can affect validation across every event in a bulk action. Treat those changes as high-impact even when the generated API remains type-compatible.

## Change and Test Surface

| Concern | Source of truth | Required review |
| --- | --- | --- |
| analytics-js cache | `src/languages/templates/typescript/analytics-js.hbs` | Generated analytics-js snapshots |
| analytics-node cache | `src/languages/templates/typescript/node.hbs` | Generated analytics-node snapshots |
| Generated output | `src/__tests__/commands/__snapshots__/build.test.ts.snap` | AJV instance and cache remain module-level |
| Production output | `src/__tests__/commands/__snapshots__/production.test.ts.snap` | No AJV runtime-validation code is emitted |

Run `yarn test` after changing either template. Review both SDK outputs even when the intended change appears target-specific.

## Consumers and Operations

Known examples of repositories using generated Typewriter clients are Semantic Service, Agents API, Feature Preview Service, and Flow Orchestration Execution. This is not an exhaustive inventory. Bulk AI actions are the performance-critical use case behind the fork.

Consumers install or invoke `@contentful/typewriter` from GitHub Packages and commit their `typewriter.yml`, local plan, and generated client. To avoid a bad release, consumers should not update to it; existing consumers can remain pinned to their current package version.

Publishing is manual: increment `package.json`, run `yarn build`, then run `npm publish`. Publication requires explicit approval and an authorized `GITHUB_TOKEN`. For incidents or ownership questions, use `#eng-af-core-ai`.

## Upstream Typewriter Reference

The inherited Typewriter architecture is supporting context, not the main maintenance focus of this fork:

| Area | Role | Reference |
| --- | --- | --- |
| CLI orchestration | oclif commands load configuration and Tracking Plans | `src/commands/`, `src/base-command.ts` |
| Tracking Plans | Segment API and local `plan.json` loading | `src/api/` |
| Client generation | Quicktype plus Handlebars emit typed clients | `src/languages/` |
| Generated-file safety | A standard header identifies replaceable generated files | `src/commands/build.ts` |

For general Typewriter usage and upstream concepts, see [Segment's Typewriter documentation](https://segment.com/docs/protocols/typewriter). Changes in this repository should normally stay within the narrow Contentful fork delta described above.
