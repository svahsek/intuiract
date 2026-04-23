# Release-Time Content Automation — Strategy Notes

_Date: 12 April 2026_

---

## Context

Release notes schema, YAML payload, and validation scripts are working. The next phase is extending automation to identify and update all documentation that must change at release time.

---

## 1. Must-Update Content Types at Release Time

In CI/CD SDLC / Agile, these fall into three tiers:

**Tier 1 — Every release (mandatory)**

| Content Type | Why |
|---|---|
| Release Notes / Changelog | Record of what changed |
| What's New / Features page | User-facing new capabilities |
| API Reference | Any endpoint/interface change |
| Configuration Guide | New/changed config keys, env vars, defaults |
| Upgrade / Migration Runbook | Breaking changes, schema migrations |

**Tier 2 — Major/minor releases**

| Content Type | Why |
|---|---|
| Developer Setup Guide | New tooling, env vars, dependencies |
| Deployment / Operations Runbook | Changed infra, steps, prerequisites |
| Architecture Overview / ADRs | Significant design decisions |

**Tier 3 — Signal-triggered**

| Content Type | Signal |
|---|---|
| Security Advisory | CVE fix, auth change |
| Deprecation Notice | Anything marked deprecated |
| Known Issues | Shipped defects |
| Test Report / QA Evidence | Traceability to release |

---

## 2. Commit/PR Signal to Doc-Impact Matrix

The existing classification layer (New Features, Bug Fixes, Breaking Changes, Internal) is the **signal layer**. A **doc-impact matrix** maps those signals to content types.

```
Commit signal              → Content Types triggered
─────────────────────────────────────────────────────
new_feature                → release-notes, features-page, api-reference*, config-guide*
breaking_change            → release-notes, upgrade-runbook, api-reference, config-guide
config_key_added/changed   → release-notes, config-guide, dev-setup*
dev_tooling_change         → dev-setup-guide, deployment-runbook*
security_fix               → release-notes, security-advisory
deprecation                → release-notes, api-reference, deprecation-notice
db_schema_change           → upgrade-runbook, deployment-runbook
bug_fix                    → release-notes (only)
```

`*` = conditional on whether the diff touches those areas

This matrix is deterministic and encodable as YAML — following the same pattern as the release notes schema.

---

## 3. Proposed Pipeline Extension

**Current pipeline:**

```
git diff + commits → classify → release-notes YAML → validate → render Markdown
```

**Extended pipeline:**

```
git diff + commits → classify → release-notes YAML
                                       ↓
                              doc-impact-matrix.yaml
                                       ↓
                     per-content-type update task list
                                       ↓
                    content-type skill (one per doc type)
```

---

## 4. Next Steps (in order)

### Step A — Build `doc-impact-matrix.yaml`

A new standard in `standards/`. Maps change categories + file-path signals (e.g., `config/**`, `src/api/**`) to a list of content types that must be updated.

### Step B — Extend the release YAML payload

Add a `doc_impact` block to the existing release notes schema output:

```yaml
doc_impact:
  triggered:
    - content_type: config-guide
      reason: "3 new config keys added"
      priority: must
    - content_type: upgrade-runbook
      reason: "Breaking change in AuthConfig"
      priority: must
    - content_type: features-page
      reason: "2 new features shipped"
      priority: must
```

### Step C — Register new content types in `content-types.yaml`

Register `features-page`, `config-guide`, `upgrade-runbook`, `dev-setup-guide` following the same pattern as `release-notes`.

### Step D — Build per-content-type skills

One skill per content type. Each skill knows its template, its trigger signals, and how to map release YAML → content update for that doc type.

### Step E — Build an orchestration skill

A `release-doc-coordinator` skill that:
1. Reads the release YAML + `doc_impact` block
2. Iterates triggered content types
3. Dispatches to the right per-content-type skill

This keeps the system composable — each content type is independently automatable, and the coordinator ties them together at release time.

---

## Starting Point

**Steps A and B first** — the doc-impact matrix and the extended YAML payload are the contracts everything else builds on.
