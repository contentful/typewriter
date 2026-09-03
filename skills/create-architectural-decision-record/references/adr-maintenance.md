# ADR Maintenance

Use this file when the repo already has ADR support files beyond the ADR itself.

## Common Surfaces To Update

After adding or superseding an ADR, check for:

- ADR index pages such as `docs/ADRs/README.md`
- docs home pages that link to ADRs
- MkDocs, Docusaurus, or similar nav files
- architecture overview pages that summarize major decisions

## Common Update Rules

- Add the new ADR link exactly where neighboring ADR links live.
- Keep existing sort order, numbering, or date ordering.
- Update status markers if the repo surfaces them outside the ADR file.
- Avoid broad cleanup unrelated to the current ADR unless the user asked for it.

## Validation

If the repo has relevant checks, run the smallest ones that prove the ADR
surface is consistent, such as:

- docs generation for skill or docs indexes
- markdown or link validation
- docs build checks

If no relevant automated validation exists, say so explicitly.
