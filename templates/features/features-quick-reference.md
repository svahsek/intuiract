# Features Quick Reference

1. Resolve `standards/content-types/features.yaml`.
2. Bootstrap the long-lived `features.yaml` from the existing product page once.
3. On each release, read canonical `release-evidence.yaml`, not rendered release notes.
4. Include only `change_type: feature`, `shipped.included_in_release: true`, and confidence `>= 85`.
5. Upsert by `item_id`; preserve existing groups and refined prose.
6. Validate against the schema and `rules/features/features-rule.yaml` before rendering `features.md`.
