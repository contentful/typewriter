# Delivery Chain File Schema

Each delivery chain is a Markdown file at `knowledge/delivery-chains/<chain-name>.md` with YAML frontmatter for machine queryability.

## Required Frontmatter Fields

```yaml
---
name: <kebab-case chain name> # e.g., webhook-delivery
owner_team: <team name> # e.g., extensibility
entry_points: # where the chain starts
  - repo: <repo-name>
    trigger: <human-readable trigger> # e.g., "POST /spaces/:spaceId/webhook_definitions"
producers: # services that emit events/messages
  - repo: <repo-name>
    publishes_to: <topic/queue name>
consumers: # services that receive events/messages
  - repo: <repo-name>
    consumes_from: <topic/queue name>
failure_modes: # known ways this chain can break
  - '<description of failure → consequence>'
slos: # service level objectives for this chain
  - '<metric and target>'
links: # GitHub URLs for all repos in the chain
  - https://github.com/contentful/<repo>
---
```

## Optional Frontmatter Fields

```yaml
data_stores: # shared data stores in the chain
  - type: <DynamoDB|RDS|S3|etc>
    name: <table/bucket name>
    accessed_by: [<repo>, <repo>]
alerts: # key alerts that fire for this chain
  - name: <alert name>
    severity: <critical|high|medium>
    meaning: <what it means>
```

## Body

Below the frontmatter, write a narrative description of the chain:

1. **Overview** — what this chain does end-to-end
2. **Flow** — step-by-step path from trigger to final outcome
3. **Failure & Recovery** — what happens when each leg fails, retry/DLQ behavior
4. **Operational Notes** — deploy ordering, feature flags, known gotchas
