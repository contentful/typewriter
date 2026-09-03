# backstage-facts.md — File Format Reference

`backstage-facts.md` is the structured intermediate snapshot produced by Step 4.5 (Backstage Enrichment). It uses YAML blocks for machine-parseable diffing and Markdown prose for human readability.

## Location

The cartographer writes one `backstage-facts.md` per phase: a Phase 1 snapshot, and a fresh Refresh snapshot during drift detection. Exact write paths are defined in `prompts/phase-1-discovery.md` Step 4.5 and `prompts/refresh.md` Step 1 — this format reference describes the file's content, not its workspace location.

## Document Frontmatter

```yaml
---
entity: <metadata.name from catalog-info.yaml>
namespace: <metadata.namespace, default: "default">
type: <spec.type — service, library, website, etc.>
lifecycle: <spec.lifecycle — production, experimental, deprecated>
owner: <spec.owner — owning group ref>
enriched_at: <ISO 8601 timestamp of when this snapshot was taken>
compliance_score: <high|low — based on get-entity-compliance results>
compliance_gaps: [<list of missing metadata fields>]
---
```

## Sections

Each section has a YAML block (diffable, machine-readable) followed by optional Markdown prose (human context). The YAML blocks are the authoritative data — prose is informational only.

### Entity Info

```yaml
description: '<entity description from Backstage>'
tags: [tag1, tag2, ...]
annotations:
  github.com/project-slug: 'org/repo'
  # ... other annotations
links:
  - url: 'https://...'
    title: 'Dashboard'
```

Prose: Human-readable summary of what this entity is.

### Relationships

```yaml
dependencies:
  - entity: 'component:default/service-name'
    type: dependsOn
provides_api:
  - entity: 'api:default/api-name'
consumes_api:
  - entity: 'api:default/other-api'
owned_by: 'group:default/team-name'
part_of_system: 'system:default/system-name'
```

Prose: Narrative describing the dependency topology, suitable for generating mermaid diagrams.

### API Specs (Top 3)

```yaml
specs_retrieved:
  - name: 'api-name'
    type: openapi|asyncapi
    version: '3.0.1'
    endpoint_count: N
specs_referenced:
  - name: 'other-api'
    entity_ref: 'api:default/other-api'
```

Prose: Per-spec summary of key endpoints and notable patterns.

**Constraints:**

- Maximum 3 specs retrieved inline (prefer providesApi > consumesApi, prefer lifecycle: production)
- Remaining APIs listed as references only (name + entityRef)

### TechDocs

```yaml
pages_found: N
pages_summarized:
  - title: 'Page Title'
    path: '/docs/path'
    maps_to_section: 'Operational Knowledge' # or "Domain Concepts", "Data Flow", "unmatched"
```

Prose: 2-3 sentence summary per page with section-mapping notes.

**Constraints:**

- Maximum 5 results from search-techdocs
- Summarize, never inline raw content
- For deprecated entities, add `[MAY BE STALE — service is deprecated]` to each summary

**Placement rule for Step 6:** Match page titles/headings to ARCHITECTURE.md section names by keyword overlap. Unmatched content goes to `§ Additional Context (from TechDocs)` appendix.

### GitHub Metrics

```yaml
avg_merge_time_hours: N
open_prs: N
active_contributors_90d: N
branch_protection: true|false
required_reviews: N
```

### Security

```yaml
snyk_critical: N
snyk_high: N
dependabot_alerts: N
branch_protection_enabled: true|false
```

### Operational (skipped if lifecycle: deprecated)

```yaml
datadog_slo_count: N
datadog_monitor_count: N
pagerduty_incident_count_90d: N
pagerduty_mttr_hours: N
pagerduty_uptime_pct: N
```

### Repository Info

```yaml
file_types:
  docker: N
  yaml: N
  javascript: N
  python: N
  # ...
is_cataloged: true|false
```

### Catalog Health Flags

```yaml
conflicts:
  - type: found_in_code_not_catalog
    detail: 'imports @contentful/auth-client but no relationship declared'
  - type: found_in_catalog_not_code
    detail: 'Backstage declares dependency on event-bus but no import found'
```

## Refresh Diffing Rules

When comparing the previous Phase 1 snapshot against the fresh Refresh snapshot (paths defined in `prompts/refresh.md` Step 1):

1. Compare YAML blocks section-by-section (not raw text diff)
2. Lists: use set diff (added items, removed items)
3. Scalars: use equality comparison
4. Prose blocks: ignored during diff (informational only)
5. Operational section: NOT compared between runs (point-in-time state, not structure)

## Confidence Markers

Based on `compliance_score` and `compliance_gaps` in frontmatter:

| Missing Field | Marker Applied To                                |
| ------------- | ------------------------------------------------ |
| `description` | `[UNVERIFIED]` on § Overview only                |
| `owner`       | `[UNVERIFIED]` on ownership/team references only |
| `tags`        | No marker (supplementary)                        |

Relationship and API spec data remain unmarked regardless of compliance score (structurally enforced by Backstage).
