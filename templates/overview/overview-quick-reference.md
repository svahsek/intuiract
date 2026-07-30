# Overview YAML Template — Agent Quick Reference
# =====================================================
# For rapid access to schema structure and validation rules
# Full documentation: overview-schema.yaml

## QUICK REFERENCE: YAML Structure

```yaml
metadata:                # REQUIRED
  productName:            # REQUIRED
  targetUsers:             # REQUIRED: [list]
  coreFunctionality:        # REQUIRED: one sentence
  primaryStandard:           # optional
  standardsReferenced:        # optional: [{name, url}]
  lastUpdated:                 # date this file was last created/patched
  lastUpdateSource:              # bootstrap_ingestion | release_signal | manual_edit

overview:                # REQUIRED
  narrative:               # REQUIRED: opening paragraph — human-authored, never patched by release_signal

featureCoverage:          # optional, but this is what the incremental pipeline maintains
  documentationLink:
  categories:               # optional — named groupings, each its own H3 + table
    - name:                    # REQUIRED, e.g. "Credential Data Models", "Verification Protocols"
      hasStatusColumn:            # default true; set false for purely descriptive reference tables
      items: [{name, description, status, lastConfirmedVersion, lastConfirmedDate, sourceRelease, provenance}]
  features:                 # optional — flat list, or the trailing "Feature Coverage" table if categories is also set
    - name:                 # REQUIRED
      description:           # optional
      status:                  # REQUIRED (unless category has hasStatusColumn: false): supported | unsupported | planned | partial
      lastConfirmedVersion:
      lastConfirmedDate:
      sourceRelease:            # required if provenance == release_signal
      provenance:                # bootstrap_ingestion | release_signal | manual_edit — "partial" can never come from release_signal

tryItOut:                 # optional
  moduleName:
  collabGuideLink:
  sandboxLink:

architecture:              # REQUIRED
  description:               # REQUIRED
  externalSystems:
  interactionMethod:
  components:                  # optional — "Key Components" bullets: [{name, description}]. Bootstrap-only.
  architectureDocumentationLink:

pluginSupport:              # optional — only if product has plugin architecture
  enabled:
  pluginTypes: [{name, description}]
  supportedIntegrations: []
  howToUse: [{title, link, description}]
  customPlugin: {interfaceName, language, codeSnippet, referenceImplementation, deploymentGuideLink}

deployment:                  # REQUIRED
  modes:                      # REQUIRED: [{name, description, link, audience}]
  customPluginDeploymentLink:
  prerequisites:                 # optional: [{name, note}]. Bootstrap-only.

sdkIntegration:                # optional, conditional — only for products shipping embeddable SDK components
  enabled:
  components: [{name, description, status}]
  documentationLink:
  note:

configurations:                # optional: [{name, description, content OR properties, note, documentationLink}]
                                # use `content` (freeform Markdown) for tables/lists; `properties` only for literal properties-file blocks

databases:                      # optional: {scriptsLink, note}

upgrades:                         # optional: [{fromVersion, toVersion, migrationGuideLink}]

upcomingFeatures:                  # optional — mirror image of featureCoverage
  items: [{name, note, targetRelease}]
  note:

documentation:                       # REQUIRED
  apiDocumentation: {baseUrl, link}
  productDocumentation:                # REQUIRED: [{title, url}]

contribution:                          # REQUIRED
  codeContributionLink:
  communityLink:                        # REQUIRED
```

---

## REQUIRED FIELDS (Must Not Be Empty)

| Field | Type | Validation | Example |
|-------|------|-----------|---------|
| `metadata.productName` | string | non-empty | `Inji Certify` |
| `metadata.targetUsers` | array | min 1 item | `[issuers, verifiers]` |
| `metadata.coreFunctionality` | string | 10-200 chars | `issue verifiable credentials...` |
| `overview.narrative` | string | 50-800 chars | opening paragraph |
| `architecture.description` | string | 20-600 chars | architecture summary |
| `deployment.modes` | array | min 1 entry | `[{name, description}]` |
| `documentation.productDocumentation` | array | min 1 entry | `[{title, url}]` |
| `contribution.communityLink` | uri | required | community forum link |

---

## THE ONE THING THAT MAKES THIS DIFFERENT FROM RELEASE-NOTES

There is **no per-run YAML**. `overview.yaml` is a single long-lived file per product.

- **First run for a product** (`evidence_source: bootstrap_ingestion`): populate every section
  from the existing overview page + features page.
- **Every run after that** (`evidence_source: release_signal`): patch **only**
  `featureCoverage` and `upcomingFeatures`, using the release-notes pipeline's own output for
  the release that just shipped. Every other section is read from the current file and written
  back byte-for-byte unchanged.

If you find yourself about to rewrite `overview.narrative`, `architecture.description`,
`architecture.components`, `deployment` (including `deployment.prerequisites`), `pluginSupport`,
`sdkIntegration`, `configurations`, or `documentation` during a release_signal run — stop. That
is not this pipeline's job on an ongoing run; see `overview-source-precedence.yaml`'s
`excluded_sources` for each field.

---

## FEATURE STATUS TRANSITIONS (the core mechanic)

```yaml
# Before a release ships:
upcomingFeatures:
  items:
    - name: "SD-JWT VC Issuance Support"
      targetRelease: "0.15.0"

# release-notes.yaml for v0.15.0 confirms it shipped (change_type: feature, shipped: true)
# → After the release_signal patch:

featureCoverage:
  features:
    - name: "SD-JWT VC Issuance Support"
      status: "supported"
      sourceRelease: "0.15.0"
      lastConfirmedDate: "2026-07-30"
      provenance: "release_signal"

upcomingFeatures:
  items: []   # entry removed — this happens atomically with the featureCoverage write
```

**If the product uses categorized `featureCoverage.categories` instead of (or alongside) the
flat `features` list**: the release_signal match search still checks `upcomingFeatures` first,
then walks every category's `items` plus the flat `features` list before concluding "net-new."
A matched row is patched in place, in whatever category it already lives in — a release_signal
patch never moves a row between categories, and never creates a new category. A **net-new**
shipped feature always lands in the flat `features` list, never invented into an existing
category, since picking a category is a bootstrap-time judgment call.

`partial` status (e.g. "JSON-LD and SD-JWT supported; mDoc/mDL not yet") can only be asserted
by `bootstrap_ingestion` or `manual_edit` — `release_signal` only ever flips a row to `supported`
(CON-002b). Whether a release that ships one of several sub-capabilities behind a `partial` row
means the row is now fully `supported` or still `partial` is a call for the next bootstrap pass,
not the automatic patch.

---

## VALIDATION RULES (Deterministic Checks)

**ERROR (blocks patch/render):**
- ❌ STR-001: Missing a required section (metadata, overview, architecture, deployment, documentation, contribution)
- ❌ STR-003: `metadata.lastUpdated` not in `YYYY-MM-DD` format
- ❌ CON-001: Missing or too-short `overview.narrative`
- ❌ CON-002: A feature status (flat list or any category's items) not one of `supported|unsupported|planned|partial`
- ❌ CON-002b: A `release_signal`-provenance row has status `partial` (release_signal may only set `supported`)
- ❌ CON-003: A `release_signal`-provenance feature row (flat or categorized) is missing `sourceRelease` or `lastConfirmedDate`
- ❌ CON-004: Same feature name appears in both `upcomingFeatures.items` and featureCoverage (flat list or any category) with status supported
- ❌ CON-005: `documentation.productDocumentation` empty
- ❌ CON-007: `deployment.modes` empty
- ❌ CON-009: Unresolved placeholder text (`TBD`, `{PRODUCT_NAME}`, etc.)
- ❌ SEC-001: Internal team/personal names present

**WARNING (quality issues):**
- ⚠️ CON-006: No `architecture.architectureDocumentationLink`
- ⚠️ CON-008: `deployment.customPluginDeploymentLink` set without `pluginSupport.enabled`
- ⚠️ CON-010: Feature row missing a `provenance` tag
- ⚠️ SEC-002: Comparative marketing language
- ⚠️ PRO-001–PRO-003: Prose style violations (Vale, on rendered Markdown)

---

## AGENT WORKFLOW

**Bootstrap run (first time for a product):**
1. Ingest existing overview page (if any) + features page.
2. Normalize into `overview-evidence.yaml` (`evidence_source: bootstrap_ingestion`).
3. Map every section into `overview.yaml` via `evidence-to-overview-mapping.yaml`'s `bootstrap_mapping`.
4. Validate against `overview-rule.yaml`. Fix ERRORs before proceeding.
5. Render `overview.md` via `overview-rendering.yaml`.

**Ongoing run (every release after that):**
1. Load the release-notes pipeline's output for the release that just shipped.
2. Normalize into `overview-evidence.yaml` (`evidence_source: release_signal`).
3. Score each item via `overview-evidence-confidence-rule.yaml`. Auto-apply ≥80, warn 50-79, hold <50.
4. Patch **only** `featureCoverage`/`upcomingFeatures` via `evidence-to-overview-mapping.yaml`'s `release_signal_mapping`.
5. Validate (CON-003 and CON-004 are the ones that matter most here). Render `overview.md`.

---

## COMMON MISTAKES TO AVOID

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| Rewriting prose on a release_signal run | Regenerating `overview.narrative` because a feature shipped | Only touch `featureCoverage`/`upcomingFeatures`; leave prose untouched |
| Dropping the roadmap entry | Adding a `featureCoverage` row without removing the matching `upcomingFeatures` item | Remove the `upcomingFeatures` entry atomically with the `featureCoverage` write (CON-004) |
| Silent status flip | Flipping `status` to `supported` with no `sourceRelease` | Always set `sourceRelease` + `lastConfirmedDate` when `provenance == release_signal` (CON-003) |
| Deleting unmatched rows during bootstrap | Removing a feature row because `features_page` doesn't mention it | Retain it and flag to `unresolved[]` for human confirmation instead |
| Placeholder leftovers | `{PRODUCT_NAME}`, `TBD` in shipped output | Resolve every placeholder before rendering (CON-009) |
| Inventing a category | Guessing which named category a net-new shipped feature belongs in during a release_signal run | Net-new rows always land in the flat `features` list; category assignment is bootstrap-only |
| Asserting `partial` from a release signal | Setting `status: partial` with `provenance: release_signal` | Only `bootstrap_ingestion`/`manual_edit` may assert `partial` (CON-002b) |

---

## FILE LOCATIONS

| File | Purpose |
|------|---------|
| `templates/overview/overview-schema.yaml` | **Schema authority** |
| `templates/overview/overview-rendering.yaml` | YAML → Markdown conversion rules |
| `templates/overview/evidence-to-overview-mapping.yaml` | Evidence → schema field mapping (source-scoped) |
| `standards/overview-source-precedence.yaml` | Which source may touch which field |
| `standards/overview-evidence-fields.yaml` | `overview-evidence.yaml` field glossary |
| `rules/overview-evidence-confidence-rule.yaml` | Confidence scoring for feature-status patches |
| `rules/overview/overview-rule.yaml` | Deterministic validation rules (STR/CON/SEC/PRO) |
| `templates/overview/examples/minimal-overview.yaml` | Minimal valid example (flat featureCoverage only) |
| `templates/overview/examples/comprehensive-overview.yaml` | Full example: categorized featureCoverage, partial status, components, prerequisites, sdkIntegration |
| `templates/overview/examples/overview-evidence.sample.yaml` | Sample normalized evidence file |

---

**Questions?** See `overview-schema.yaml` for full documentation with examples.
