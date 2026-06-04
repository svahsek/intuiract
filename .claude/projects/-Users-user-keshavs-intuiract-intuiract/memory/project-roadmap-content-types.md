---
name: project-roadmap-content-types
description: Planned content type build order for the multi-output documentation pipeline
metadata:
  type: project
---

Planned progression for expanding the documentation intelligence framework beyond release notes:

**Phase 1 — Easier (render new content from evidence):**
- Features page
- User guide delta (harder than features page — needs locate+patch pattern)

**Phase 2 — Harder (surgical updates to existing long-form docs):**
- Deployment Guide — infra/ops audience, config/infra change signals
- Upgrade Runbook — procedural, ordering-sensitive, rollback-aware
- SDK Guide — developer audience, API method signatures, code examples, migration notes

**Why:** The Phase 2 docs require a "locate existing section → diff → patch" pattern, not just generation from scratch. The `user-guide-delta` skill in Phase 1 is the prototype for this pattern — design it carefully as it will be reused.

**How to apply:** When building user-guide-delta, treat it as the foundation for the harder content types, not just a one-off. See [[known-issues-table-fallback]] and docs/plan/future-path-2.md for related context.
