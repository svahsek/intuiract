# Documentation Intelligence Approach for Multi-Content Automation

Approach this as a Documentation Intelligence pipeline, not just a bigger release-notes skill.

## Target Architecture

1. Build one shared ingestion layer for git diffs, commit messages, PR titles/descriptions, labels, linked issues, release tags, and changed files.
2. Build one shared classification layer that maps evidence to documentation intent.
3. Build separate renderers per content type: Release Notes, Overview, Features, User Guide, Installation Guide, API Changes.
4. Keep one shared validation/rules engine for style, completeness, security redaction, and link/reference quality.
5. Add confidence scoring and human-approval gates before publish.

## Core Design Principle

1. Separate Facts from Narrative.
2. Facts: what changed, where, risk, compatibility, migration impact, issue refs.
3. Narrative: how those facts are expressed per page type.

If you do this, adding new content types becomes mostly “new rendering + rules,” not redoing extraction every time.

## Recommended Rollout

1. Phase 1: Release Notes + Features Page auto-draft.
2. Phase 2: Overview Page + API Changes.
3. Phase 3: User Guide + Installation Guide deltas.
4. Phase 4: Fully automated PR-to-doc suggestions with approval workflow.

## How to Classify Automatically

1. Use a two-stage classifier.
2. Stage A: Deterministic rules first.
3. Signals: changed paths, file extensions, commit prefixes, PR labels, issue type, keywords.
4. Stage B: LLM classifier only for ambiguous cases.
5. Output for each change unit: content_type, section, confidence, evidence, omission_reason.
6. If confidence is below threshold, route to review queue.

## Suggested Change Unit

1. One PR or one grouped set of commits between tags.
2. Break into atomic “doc events”:
3. Feature added.
4. Behavior changed.
5. API contract changed.
6. Installation prerequisite changed.
7. Config fixed values moved to configuration externalization.
8. Security or compliance update.
9. Deprecation or migration requirement.

## Mapping Strategy (Example)

1. Features Page:
2. Triggers: feat commits, PR label feature, new module/capability files, user-visible behavior.
3. Overview Page:
4. Triggers: multiple cross-module changes, strategic theme labels, roadmap-linked items.
5. User Guide:
6. Triggers: workflow or UX behavior changes, CLI/command flow changes, role-based usage changes.
7. Installation Guide:
8. Triggers: Dockerfile/build scripts/env var/dependency/runtime/version changes.
9. API Changes:
10. Triggers: OpenAPI/proto/schema/controller/signature changes, request/response diffs.

## Quality Gates You Need

1. Structural: required sections present by content type.
2. Content: minimum specificity, no placeholders, issue links valid.
3. Consistency: same change should not conflict across pages.
4. Security: no internal emails/private endpoints/team-only terms.
5. Policy: release type constraints, migration notes required when breaking.
6. Freshness: reject stale PRs outside release window unless explicitly included.

## Human-in-the-Loop Model

1. Auto-generate drafts.
2. Show evidence snippets beside each generated item.
3. Let reviewer accept/edit/reject each item.
4. Feed reviewer decisions back into rules and prompt tuning.
5. Publish only approved sections.

## Metrics to Prove Value

1. Draft coverage: percent of required sections auto-filled.
2. Precision: accepted items over total generated items.
3. Recall proxy: missed changes found in review.
4. Time-to-publish reduction.
5. Rework rate after publication.
6. Confidence calibration accuracy.

## Practical First Implementation

1. Reuse your existing release-notes pipeline for manifest, schema, validation, rendering.
2. Add a new intermediate artifact: docs-events.yaml.
3. Populate docs-events.yaml from git/PR/tag data.
4. Build per-content-type generators that consume docs-events.yaml.
5. Store classification rules in one central file with per-type overrides.
6. Run this in CI on tag push and on release branch updates.

## What to Build Next (Concrete)

1. Define docs-events schema first.
2. Build deterministic classifier for 60 to 70 percent of cases.
3. Add LLM fallback with confidence and evidence.
4. Implement Features Page generator first.
5. Add Overview generator second.
6. Add User Guide and Installation Guide after path-based triggers are stable.
