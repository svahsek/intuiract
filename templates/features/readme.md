# Features Content Type

Features is a long-lived product catalog, not a release-specific changelog.

Bootstrap `features.yaml` from an existing features page and operator input. For later
releases, use `release-evidence.yaml` directly: only shipped `feature` items with confidence
of at least 85 are upserted. Existing groups and refined prose are retained; new items remain
ungrouped until an operator assigns a user-journey or stakeholder group.

Resolve every asset through `standards/content-types/features.yaml`.

For standards-guided remediation of an existing Features page that does not have a
`features.yaml` source, use the manifest's `review_and_remediate_skill`. It audits and safely improves
editorial quality without asserting or changing product facts unless a separate source is
provided.
