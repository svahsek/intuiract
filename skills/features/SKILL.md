---
name: features
description: "Maintain a product feature catalog from canonical release evidence."
argument-hint: "Provide the product features.yaml and, for an update, release-evidence.yaml."
---

# Features

## Canonical Inputs

Resolve `standards/content-types/features.yaml` first. Use its schema, mapping, rendering,
rules, and upstream release-evidence contract; do not hardcode those paths.

## Workflow

1. **Bootstrap once:** ingest the existing product features page and operator input into a
   long-lived `features.yaml`. Preserve its user-journey or stakeholder organization as
   `featureGroups`.
2. **Load release evidence:** resolve the release-notes manifest and load that release's
   canonical `release-evidence.yaml`. Never use rendered `release-notes.md` as evidence.
3. **Filter and map:** apply `evidence-to-features-mapping.yaml`. Include only shipped
   `feature` items scoring at least 85. Upsert by `item_id`, retain existing groups and
   non-empty refined prose, and leave net-new items ungrouped.
4. **Validate:** apply `features-schema.yaml` and `rules/features/features-rule.yaml`.
   Report unresolved or excluded evidence; do not render on validation failure.
5. **Render:** apply `features-rendering.yaml` to produce `features.md`.

The features catalog is incremental: do not regenerate or delete existing features solely
because they are absent from one release's evidence.
