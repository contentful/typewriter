# Maintain a Restricted Contentful Fork

## Status

Accepted

## Context

Contentful's bulk AI actions validate many analytics messages in a single operation. The upstream generated clients created an AJV instance and compiled the event schema for every runtime validation. Repeating that work across a bulk action would severely degrade its performance.

Contentful needed generated clients that compile a schema once and reuse its validator. This behavior was not available in the upstream Segment package, while reimplementing the entire CLI and generator stack would duplicate mature upstream functionality. Commit `f9206de` added validator reuse; commit `c1ad641` configured the scoped package and restricted GitHub Packages registry needed to distribute it internally.

## Decision

Maintain `contentful/typewriter` as a narrow fork and publish it as the restricted `@contentful/typewriter` package. Treat cached AJV validator reuse as the fork's primary invariant. Keep the implementation isolated in the analytics-js and analytics-node templates so the delta from upstream remains identifiable.

## Consequences

Bulk AI actions avoid paying schema-compilation cost for every validated event. Contentful can ship that behavior without rebuilding the generator ecosystem, but the team owns manual releases, registry authentication, and reconciliation with upstream changes.

Any upstream sync or template refactor must preserve the cache. Reintroducing AJV construction or schema compilation into the per-event path is a performance regression and defeats the reason for maintaining the fork.
