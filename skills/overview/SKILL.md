---
name: overview
description: 'Generate and maintain product/component overview pages using the centralized overview manifest, YAML schema, and rendering rules. One pipeline, two evidence sources: bootstrap-ingest an existing overview page and features page the first time, then patch the feature-coverage table on every subsequent release using the release-notes pipeline output.'
argument-hint: 'Provide the product name and, for ongoing runs, the release-notes evidence to patch against'
---

# Overview

## When To Use

- Bootstrap a product's first `overview.yaml` from its existing overview page and features page.
- Patch an existing `overview.yaml`'s feature-coverage table after a release ships.
- Validate an overview YAML payload before converting it to Markdown.
- Consume shared overview standards from another repository using direct file URLs.

## Canonical Inputs & Standards

**Registry & Manifests**:
- Content registry: [../../standards/content-types.yaml](../../standards/content-types.yaml)
- Overview manifest: [../../standards/content-types/overview.yaml](../../standards/content-types/overview.yaml)
- Upstream release-notes manifest: [../../standards/content-types/release-notes.yaml](../../standards/content-types/release-notes.yaml)

**Canonical Evidence Model**:
- Evidence field glossary: [../../standards/overview-evidence-fields.yaml](../../standards/overview-evidence-fields.yaml)
- Source precedence rules: [../../standards/overview-source-precedence.yaml](../../standards/overview-source-precedence.yaml)

**Schema & Rendering**:
- Schema: [../../templates/overview/overview-schema.yaml](../../templates/overview/overview-schema.yaml)
- Rendering rules: [../../templates/overview/overview-rendering.yaml](../../templates/overview/overview-rendering.yaml)
- Evidence-to-overview mapping: [../../templates/overview/evidence-to-overview-mapping.yaml](../../templates/overview/evidence-to-overview-mapping.yaml)

**Policy & Validation**:
- Policy rules: [../../rules/overview/overview-rule.yaml](../../rules/overview/overview-rule.yaml)
- Confidence scoring rules: [../../rules/overview-evidence-confidence-rule.yaml](../../rules/overview-evidence-confidence-rule.yaml)

**Reference & Examples**:
- Sample evidence file: [../../templates/overview/examples/overview-evidence.sample.yaml](../../templates/overview/examples/overview-evidence.sample.yaml)
- Minimal overview example: [../../templates/overview/examples/minimal-overview.yaml](../../templates/overview/examples/minimal-overview.yaml)
- Quick reference: [../../templates/overview/overview-quick-reference.md](../../templates/overview/overview-quick-reference.md)
- Legacy manual template (superseded, kept for reference): [../../templates/overview/overview-page.template.md](../../templates/overview/overview-page.template.md)

## Procedure

This skill orchestrates one pipeline with a branch at Phase 1. **Do not skip or combine phases.**
Which branch you take at Phase 1 determines which schema sections Phase 3 is allowed to touch —
this is enforced by `evidence-to-overview-mapping.yaml`, not left to judgment call.

### Phase 1: Extraction

**Branch A — Bootstrap Ingestion** (first run for this product, or an explicit full refresh)

1. Locate the existing overview page for this product (if one exists — e.g. this repo's
   `templates/overview/overview-page.template.md`-derived draft, or a consumer repo's
   hand-written page).
2. Locate the features page/doc for this product.
3. (Deferred) Code-repo ingestion is not yet implemented — skip it; do not block on it.
4. Extract per-feature status claims and all narrative/architecture/deployment/configuration
   content.

**Branch B — Release Signal** (every run after the first)

1. Resolve `standards/content-types/release-notes.yaml` and load that release's
   `release-notes.yaml` and/or `release-evidence.yaml`.
2. Filter to items where `change_type == "feature"` and `shipped == true`.
3. Nothing else from that release matters to this pipeline — bug fixes, security updates, and
   technical improvements are out of scope for the overview page.

**Output**: Raw evidence bundle, tagged with `evidence_source: bootstrap_ingestion` or
`release_signal`.

### Phase 2: Reconciliation

1. **Read reconciliation authority**:
   - Load `overview-source-precedence.yaml` (field-level precedence and `excluded_sources`)
   - Load `overview-evidence-fields.yaml` (field semantics)

2. **Apply precedence per field**, per the branch taken in Phase 1. On Branch B, this step is
   mostly matching, not arbitration — see Phase 2b.

2b. **Feature matching (Branch B only)**:
   - Fuzzy-match each shipped feature's name against `upcomingFeatures.items[*].name` first
     (expected common case), then against `featureCoverage.features[*].name`, in that order.
   - No match → treat as net-new.

3. **Calculate confidence** using `overview-evidence-confidence-rule.yaml`:
   - Items >= 80: auto-apply
   - Items 50-79: apply with warning
   - Items < 50: hold — do not patch, add to `unresolved[]`
   - Any item flagged as contradicting existing content is added to `unresolved[]` regardless
     of score (see the confidence rule's override note).

4. **Generate normalized output**: `overview-evidence.yaml`, with every item carrying a
   `provenance` tag.

**Output**: `overview-evidence.yaml` (canonical, normalized, scored)

### Phase 3: Patch / Generate

1. **Load mapping authority**: `evidence-to-overview-mapping.yaml`.
2. **Load the existing `overview.yaml`** if one exists (empty object if this is a true first run).
3. Apply the mapping scoped to the Phase 1 branch:
   - `bootstrap_mapping`: populate every section. Never overwrite a field that already has
     non-empty content unless the operator explicitly requested a full refresh.
   - `release_signal_mapping`: touch only `featureCoverage` and `upcomingFeatures`. For each
     auto-applied or warned item: upsert the `featureCoverage` row (status, sourceRelease,
     lastConfirmedDate, provenance) and remove the matching `upcomingFeatures` entry in the same
     write.
4. Items held at Phase 2 (confidence < 50, or flagged contradictions) are left out of this
   patch — the existing `overview.yaml` row for them, if any, stays as-is.

**Output**: Patched `overview.yaml`

### Phase 4: Validation

Validate before rendering. If validation fails, do NOT render.

1. **Schema validation**: Check `overview.yaml` matches `overview-schema.yaml` — required
   sections present, enum values valid, field types correct.
2. **Policy validation**: Run checks from `overview-rule.yaml`.
   - STR-*: required sections, date formats
   - CON-*: overview narrative present, feature status enum valid, **CON-003** (release_signal
     rows carry sourceRelease + lastConfirmedDate), **CON-004** (no feature listed in both
     upcomingFeatures and featureCoverage-as-supported simultaneously), documentation/deployment
     completeness, no placeholder text
   - SEC-*: no internal content
   - PRO-*: apply via Vale linter on rendered Markdown

**Failure Handling**:
- If validation fails, report the failing rule ID(s) and field path(s).
- Do NOT render or publish.
- On a `release_signal` run, a validation failure should be scoped to the specific item(s) that
  caused it where possible — don't block an otherwise-clean patch over one bad row; move that
  row to `unresolved[]` and proceed with the rest.

### Phase 5: Rendering

Convert validated YAML to Markdown.

1. Apply `overview-rendering.yaml`'s section templates in order (HDR, FEA, TRY, ARC, PLG, DEP,
   CFG, DB, UPG, UPC, DOC, CTR), omitting any section whose top-level data is absent.
2. **Output**: `overview.md` (publication-ready)

## Consumer Repo Pattern

- In a consumer repository, create a local skill that fetches the manifest first.
- Resolve the schema, mapping, and rendering files from the manifest instead of hardcoding
  every path independently.
- **Critical**: Do not embed source-precedence or confidence-scoring logic in the skill. Fetch
  those from standards files (`overview-source-precedence.yaml`,
  `overview-evidence-confidence-rule.yaml`).
- **Critical**: A release_signal run must resolve the release-notes manifest
  (`standards/content-types/release-notes.yaml`) to find that release's output — do not assume
  a fixed file path for it.

## Output Expectations

1. **Overview evidence** (`overview-evidence.yaml`): Normalized, reconciled, scored. Ephemeral —
   regenerated fresh each run, not committed long-term (unlike `overview.yaml` itself).
2. **Overview YAML** (`overview.yaml`): The long-lived, structured source of truth for this
   product's overview page. Patched, never replaced wholesale on a release_signal run.
3. **Overview Markdown** (`overview.md`): Publication-ready, styled.
4. **Validation report** (if failures): Details on blocking issues, fields to fix.

**Key principle**: If validation fails, all outputs are held. Do NOT publish partial or
compromised docs.

## Troubleshooting

- **A shipped feature didn't get patched in**: Check its confidence score — likely fell below
  50 (manual_review) or was flagged as a contradiction. See `unresolved[]` in the evidence file.
- **Validation failures on CON-003**: A `release_signal` row is missing `sourceRelease` or
  `lastConfirmedDate` — check the mapping actually ran `release_signal_mapping`'s full item list,
  not a partial one.
- **Validation failures on CON-004**: The `upcomingFeatures` removal didn't happen atomically
  with the `featureCoverage` write — check the mapping's `upcoming_features_removal` rule ran.
- **Bootstrap run looks incomplete**: Confirm both the existing overview page and the features
  page were actually located — a missing features page degrades feature-status accuracy but
  should not silently produce an empty `featureCoverage` section without a log line saying so.
