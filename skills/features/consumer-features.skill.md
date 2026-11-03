---
name: features-consumer
description: "Maintain this repository's incremental features catalog from canonical release evidence."
---

# Features Consumer Skill

1. Fetch `standards/content-types/features.yaml` from the central standards repository and
   resolve every entrypoint from its `entrypoints` section.
2. On first use, map the existing product features page into `features.yaml`.
3. For each shipped release, load its `release-evidence.yaml` through the resolved
   release-notes manifest. Filter to shipped features with confidence at least 85.
4. Upsert qualifying features by `item_id`. Preserve prior groups and non-empty editorial
   fields; append net-new features without a group for operator classification.
5. Validate `features.yaml` before rendering `features.md`. Do not render invalid output.
