---
name: features-consumer
description: "Maintain this repository's incremental features catalog from pinned central standards and canonical release evidence."
---

# Features Consumer Skill

## Standards Configuration

```yaml
standards_mode: "remote"                  # remote | vendored; remote is required by default
standards_revision: ""                    # required: Intuiract commit SHA or immutable release tag
standards_vendor_root: ""                 # required only when standards_mode is vendored
```

In `remote` mode, fetch
`https://raw.githubusercontent.com/svahsek/intuiract/{standards_revision}/standards/content-types/features.yaml`
and all resolved entrypoints only into the agent's ephemeral session workspace. Resolve an
immutable tag to a commit SHA. Never copy standards into the consumer repository. If the pinned
revision cannot be loaded, stop with an error; do not skip validation or fall back locally.

`vendored` mode is an explicit offline exception. Load standards only from
`standards_vendor_root`; never switch modes implicitly.

## Workflow

1. Resolve the Features manifest and its entrypoints from the configured mode.
2. On first use, map the existing product features page into `features.yaml`.
3. For each shipped release, load `release-evidence.yaml` through the resolved release-notes
   manifest. Filter to shipped features with confidence at least 85.
4. Upsert qualifying features by `item_id`. Preserve prior groups and non-empty editorial
   fields; append net-new features without a group for operator classification.
5. Validate `features.yaml` before rendering `features.md`. Do not render invalid output.
   Record the standards mode, manifest URL, resolved commit SHA, and SHA-256 hash of every
   loaded entrypoint in the validation report.
