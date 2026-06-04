---
name: release-notes
description: 'Generate and validate release notes using the centralized release-notes manifest, YAML schema, and rendering rules. Use for release documentation, changelog generation, and consumer repos that fetch standards from this public .github repository.'
argument-hint: 'Provide the release version, source inputs, and output target'
---

# Release Notes

## When To Use

- Generate release notes from issues, PRs, discussions, test evidence, or design inputs.
- Validate a release-notes YAML payload before converting it to Markdown.
- Consume shared release-notes standards from another repository using direct file URLs.

## Canonical Inputs & Standards

**Registry & Manifests**:
- Content registry: [../../standards/content-types.yaml](../../standards/content-types.yaml)
- Release-notes manifest: [../../standards/content-types/release-notes.yaml](../../standards/content-types/release-notes.yaml)

**Canonical Evidence Model**:
- Release evidence contract: [../../docs/course-correction/release-evidence-contract-v1.md](../../docs/course-correction/release-evidence-contract-v1.md)
- Source precedence rules: [../../standards/source-precedence.yaml](../../standards/source-precedence.yaml)
- Field glossary: [../../standards/release-evidence-fields.yaml](../../standards/release-evidence-fields.yaml)

**Schema & Rendering**:
- Schema: [../../templates/release-notes/release-notes-schema.yaml](../../templates/release-notes/release-notes-schema.yaml)
- Rendering rules: [../../templates/release-notes/release-notes-rendering.yaml](../../templates/release-notes/release-notes-rendering.yaml)
- Evidence-to-release-notes mapping: [../../templates/release-notes/evidence-to-release-notes-mapping.yaml](../../templates/release-notes/evidence-to-release-notes-mapping.yaml)

**Policy & Validation**:
- Policy rules: [../../rules/release-notes/release-notes-rule.yaml](../../rules/release-notes/release-notes-rule.yaml)
- Confidence scoring rules: [../../rules/release-evidence-confidence-rule.yaml](../../rules/release-evidence-confidence-rule.yaml)

**Reference & Examples**:
- Target-state architecture: [../../docs/architecture/agentic-doc-pipeline-target-state.md](../../docs/architecture/agentic-doc-pipeline-target-state.md)
- Sample evidence file: [../../templates/release-notes/examples/release-evidence.sample.yaml](../../templates/release-notes/examples/release-evidence.sample.yaml)
- Quick reference: [../../templates/release-notes/agent-quick-reference.md](../../templates/release-notes/agent-quick-reference.md)

## Procedure

This skill orchestrates a 4-layer pipeline. Do not skip or combine layers.

### Phase 1: Extraction (or Load Pre-Extracted Evidence)

**Option A: Extract from GitHub**
1. Extract GitHub issues for release milestone
2. Extract GitHub PRs merged in version range (base_tag → target_tag)
3. Extract commits between tags
4. Extract QA test results (linked to PRs via test_case_ids)
5. Extract build/security scan results

**Option B: Load Pre-Extracted Evidence (Faster for Pilot)**
1. Use sample: [../../templates/release-notes/examples/release-evidence.sample.yaml](../../templates/release-notes/examples/release-evidence.sample.yaml)
2. Or use existing `release-evidence.yaml` from previous run

**Output**: Raw evidence snapshots (or skip if using pre-existing evidence)

### Phase 2: Reconciliation

This layer normalizes and reconciles evidence. **This is the critical innovation: field-level precedence, not prompt-driven guessing.**

1. **Read reconciliation authority**:
   - Load `source-precedence.yaml` (field-level truth)
   - Load `release-evidence-fields.yaml` (field semantics)
   - Load `classification-rules.yaml` (commit-to-type mapping)

2. **Apply precedence for each field**:
   - For each evidence item, determine which source wins using `source-precedence.yaml`
   - Example: `change_type` primary source is issue labels → fallback is PR labels → fallback is commit prefix
   - Track provenance (which source provided each field)

3. **Resolve conflicts**:
   - If issue says "bug" but PR says "enhancement" → apply conflict resolution rule
   - If PR merged but not in tag → shipped status = false
   - If QA tests fail → qa_status = "fail" (overrides PR checks)

4. **Calculate confidence**:
   - Score each item 0-100 based on linkage, labels, QA evidence, shipped status
   - Items >= 85: auto-include
   - Items 60-84: include with warnings
   - Items < 60: manual review required

5. **Generate normalized output**:
   - Populate `normalized_for_release_notes` fields for each item (pre-mapped to schema)
   - Create `unresolved[]` array for conflicts requiring manual review

**Output**: `release-evidence.yaml` (canonical, normalized, scored)

### Phase 3: Content Generation

Generate release-notes structured content from normalized evidence.

1. **Load mapping authority**:
   - Read `evidence-to-release-notes-mapping.yaml`

2. **Filter evidence items**:
   - Features: change_type == "feature" AND confidence >= 85 AND shipped == true
   - Enhancements: change_type == "enhancement" AND confidence >= 60 AND shipped == true
   - Bugs: change_type == "bug" AND confidence >= 60 AND shipped == true
   - Security: change_type == "security" AND shipped == true

3. **Map evidence fields to schema**:
   - For each feature: extract `normalized_for_release_notes.feature_*` fields
   - Use mapping file to transform into `content.newFeatures[*]` schema
   - Apply transformations (trim_v_prefix, map_severity, etc.)
   - Collect metadata (version, release_date, release_type)

4. **Populate manual input**:
   - Release headline (operator writes, must be benefit-driven)
   - Summary paragraphs (operator writes, 2-4 paragraphs)
   - Target audience (operator specifies)
   - Rollout timeline (operator specifies)

**Output**: `release-notes.yaml` (structured, ready for validation)

### Phase 4: Validation

Validate before rendering. If validation fails, do NOT render.

1. **Schema validation**:
   - Check `release-notes.yaml` matches `release-notes-schema.yaml`
   - Verify all required fields present
   - Verify type constraints, patterns, enum values

2. **Policy validation**:
   - Run checks from `release-notes-rule.yaml`
   - Structural rules (STR-*): sections present, versions formatted
   - Content rules (CON-*): required fields, issue references valid
   - Security rules (SEC-*): no internal content, no leaks
   - Prose rules (PRO-*): apply via Vale linter

3. **Confidence gate**:
   - Check confidence scores from reconciliation phase
   - Items >= 85: proceed
   - Items 60-84: include but flag
   - Items < 60: block unless manually approved

4. **Style checks**:
   - Run Vale on rendered markdown (after rendering)
   - Check for prohibited marketing language, inconsistent tense, etc.

**Failure Handling**:
- If validation fails, report failing field(s) and line numbers
- Do NOT render or publish
- Return failure report to operator for manual review

### Phase 5: Rendering

Convert validated YAML to Markdown.

1. **Apply rendering rules**:
   - Read `release-notes-rendering.yaml`
   - Transform YAML dates (YYYY-MM-DD → "DD Month YYYY")
   - Format lists, tables, links per rules
   - Apply typography (heading levels, spacing)

2. **Generate table of contents**:
   - Extract headings (H2, H3)
   - Create numbered TOC

3. **Output**:
   - `release-notes.md` (publication-ready)

## Expected Inputs (Operator Provides)

```yaml
# Operator input for Phase 3 (manual_input fields)
operator_input:
  release_version: "1.2.3"
  release_type: "Minor"
  release_date: "2026-04-23"
  product_name: "CRVS Integration Platform"
  product_abbr: "CRVS"
  
  # Release headline (benefit-driven, no hype)
  release_headline: "Self-Service Client Management Accelerates Integrator Onboarding"
  
  # Summary paragraphs (2-4)
  summary_paragraphs:
    - text: "This release focuses on partner self-service capabilities..."
      section_label: "ThemesAndFocus"
  
  # Target audience
  target_audience:
    - "Integration Partners"
    - "Platform Administrators"
  
  # Rollout timeline
  rollout_timeline: "Staggered rollout over 2 weeks starting 2026-04-23"
```

## Consumer Repo Pattern

- In a consumer repository, create a local skill that fetches the manifest first.
- Resolve the schema, mapping, and rendering files from the manifest instead of hardcoding every path independently.
- Until version tags exist, prefer pinning to a commit SHA if stability matters.
- **Critical**: Do not embed extraction or precedence logic in the skill. Fetch those from standards files (source-precedence.yaml).

## Output Expectations

1. **Release evidence** (`release-evidence.yaml`): Normalized, reconciled, scored. Single source of truth.
2. **Release notes YAML** (`release-notes.yaml`): Structured, validated, policy-compliant.
3. **Release notes Markdown** (`release-notes.md`): Publication-ready, styled.
4. **Validation report** (if failures): Details on blocking issues, fields to fix.

**Key principle**: If validation fails, all outputs are held. Do NOT publish partial or compromised docs.

## Troubleshooting

- **Low confidence scores**: Check linkage (issue linked? PR linked? QA evidence attached?). See confidence scoring rules.
- **Validation failures**: Run validation separately with `--verbose` flag to see exact failing field.
- **Missing fields**: Check source-precedence.yaml; may require fallback sources or manual input.
- **Schema mismatches**: Use sample `release-evidence.sample.yaml` and `evidence-to-release-notes-mapping.yaml` as templates.
