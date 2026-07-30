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
  features:
    - name:                 # REQUIRED
      description:           # optional
      status:                  # REQUIRED: supported | unsupported | planned
      lastConfirmedVersion:
      lastConfirmedDate:
      sourceRelease:            # required if provenance == release_signal
      provenance:                # bootstrap_ingestion | release_signal | manual_edit

tryItOut:                 # optional
  moduleName:
  collabGuideLink:
  sandboxLink:

architecture:              # REQUIRED
  description:               # REQUIRED
  externalSystems:
  interactionMethod:
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

configurations:                # optional: [{name, description, properties, note, documentationLink}]

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
`deployment`, `configurations`, or `documentation` during a release_signal run — stop. That is
not this pipeline's job on an ongoing run; see `overview-source-precedence.yaml`'s
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

---

## VALIDATION RULES (Deterministic Checks)

**ERROR (blocks patch/render):**
- ❌ STR-001: Missing a required section (metadata, overview, architecture, deployment, documentation, contribution)
- ❌ STR-003: `metadata.lastUpdated` not in `YYYY-MM-DD` format
- ❌ CON-001: Missing or too-short `overview.narrative`
- ❌ CON-002: `featureCoverage.features[*].status` not one of `supported|unsupported|planned`
- ❌ CON-003: A `release_signal`-provenance feature row is missing `sourceRelease` or `lastConfirmedDate`
- ❌ CON-004: Same feature name appears in both `upcomingFeatures.items` and `featureCoverage.features` (status supported)
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
| `templates/overview/examples/minimal-overview.yaml` | Minimal valid example |
| `templates/overview/examples/overview-evidence.sample.yaml` | Sample normalized evidence file |

---

**Questions?** See `overview-schema.yaml` for full documentation with examples.
