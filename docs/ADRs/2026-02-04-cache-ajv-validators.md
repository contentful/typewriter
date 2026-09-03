# Cache AJV Validators

## Status

Accepted

## Context

Generated JavaScript and TypeScript analytics clients validate events at runtime in development and test modes. Bulk AI actions can send many events through this validation path in one operation. The upstream templates created a new AJV instance and compiled the same JSON Schema for every event, so compilation cost grew with the number of validated events and would severely degrade bulk-action performance.

Commit `f9206de` introduced validator reuse as the Contentful fork's key behavioral change.

## Decision

Create one module-level AJV instance and a `Map<string, ValidateFunction>` in generated development/runtime-validation clients. Use the schema `$id` as the cache key. On a cache miss, compile once and store the validator; on a cache hit, reuse it. Schemas without a `$id` are compiled on demand and are not cached.

Apply identical behavior to the analytics-node and analytics-js templates. Preserve the existing `onViolation` behavior and keep AJV out of production output.

## Consequences

Repeated validation of the same event schema no longer recompiles it, protecting bulk AI action performance. Generated snapshots are the primary regression evidence for the implementation.

Changes to schema identity, template scope, or validation flow must preserve all of these properties:

- `new Ajv()` is not executed per event.
- `ajv.compile()` runs only on a cache miss or for a schema without `$id`.
- analytics-js and analytics-node remain behaviorally equivalent.
- validation errors continue through the existing `onViolation` path.
- production output remains free of the AJV dependency.
