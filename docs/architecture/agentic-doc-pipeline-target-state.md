# Agentic Documentation Pipeline: Target-State Architecture

## Purpose

Define the target operating shape for an agentic documentation pipeline that transforms GitHub evidence into validated, multi-format documentation.

This document bridges the conceptual model in [../course-correction/documentation-intelligence-cli-v1-spec.md](../course-correction/documentation-intelligence-cli-v1-spec.md) with the current standards repository structure.

## High-Level Flow

```
[GitHub Evidence]          [Local QA System]
     ↓                            ↓
  Issues    PRs   Commits   Test Cases   Build Status
     ↓                            ↓
  ┌─────────────────────────────────┐
  │     EXTRACTION LAYER            │
  │  (Extract from sources)         │
  ├─────────────────────────────────┤
  │ Issues/Projects Extractor       │
  │ PR/Commit Extractor             │
  │ QA Test Evidence Importer       │
  │ Build/Security Signal Importer  │
  └─────────────────────────────────┘
              ↓
   [Raw Evidence Snapshots]
     issues.json, prs.json,
     commits.json, tests.json
              ↓
  ┌─────────────────────────────────┐
  │  RECONCILIATION LAYER           │
  │  (Merge, resolve conflicts,     │
  │   apply precedence, score)      │
  ├─────────────────────────────────┤
  │ Merge by issue/PR/commit IDs    │
  │ Apply field-level precedence    │
  │  (source-precedence.yaml)       │
  │ Resolve conflicts               │
  │ Calculate confidence scores     │
  │ Generate normalized mappings    │
  └─────────────────────────────────┘
              ↓
   [Canonical Evidence]
     release-evidence.yaml
   (normalized, reconciled,
    scored, validated)
              ↓
  ┌─────────────────────────────────┐
  │  CONTENT GENERATION LAYER       │
  │  (Multiple renderers)           │
  ├─────────────────────────────────┤
  │ Release Notes Generator         │
  │ Features Page Generator         │
  │ Overview Generator              │
  │ API Changes Generator           │
  │ User Guide Delta Generator      │
  │  (All consume same evidence)    │
  └─────────────────────────────────┘
         ↓ ↓ ↓ ↓ ↓
   Structured Content
   (YAML per content type)
         ↓ ↓ ↓ ↓ ↓
  ┌─────────────────────────────────┐
  │  VALIDATION LAYER               │
  │  (Structure, policy, quality)   │
  ├─────────────────────────────────┤
  │ Schema Validation               │
  │  (release-notes-schema.yaml)    │
  │ Policy Rules                    │
  │  (release-notes-rule.yaml)      │
  │ Quality Checks (Vale, Doc Detect)
  │ Confidence Gate                 │
  └─────────────────────────────────┘
              ↓
       Validation Pass?
        ↙        ↘
      YES        NO
       ↓           ↓
    Render    Fail Report
      ↓           ↓
   Markdown   Manual Review
      ↓           ↓
  Publish    ← → Fix & Retry
              
  [Published Documentation]
     Release Notes
     Features Page
     Overviews
     API Docs
```

## Layer Responsibilities

### 1. Extraction Layer

**Purpose**: Pull raw evidence from GitHub, QA systems, build pipelines, and create atomic snapshots.

**Responsibility**: Faithful replication, no transformation or precedence logic.

**Inputs**:
- GitHub Issues API
- GitHub PRs API
- GitHub Commits API  
- QA system API
- Build/CI logs
- Git tags

**Outputs**:
- `issues.json`: Raw GitHub issues for release milestone
- `prs.json`: Raw GitHub PRs merged in date range
- `commits.json`: Raw commits in tag range
- `qa-evidence.json`: Test case results
- `build-evidence.json`: Build status, security scans
- `extract-metadata.json`: Extraction run details

**NOT handled here**:
- Precedence (which source wins)
- Conflict resolution
- Confidence scoring
- Normalization to release-notes schema

### 2. Reconciliation Layer

**Purpose**: Transform raw evidence into a canonical, normalized intermediate model.

**Responsibility**:
- Merge duplicate items (same issue linked by multiple PRs)
- Apply field-level precedence rules (from `source-precedence.yaml`)
- Resolve conflicts deterministically
- Calculate confidence scores
- Generate pre-mapped fields for content generators
- Identify unresolved items

**Inputs**:
- `issues.json`, `prs.json`, `commits.json`, `qa-evidence.json`, `build-evidence.json` from extraction layer
- `source-precedence.yaml` (field-level authority)
- `release-evidence-fields.yaml` (field semantics)
- `classification-rules.yaml` (change type mapping)

**Outputs**:
- `release-evidence.yaml`: Canonical normalized evidence
- `unresolved-items.yaml`: Conflicts requiring manual review
- `reconciliation-report.txt`: Process audit trail

**Logic**:
For each field in each item:
1. Identify available sources (issue data, PR data, commit data, etc.)
2. Look up field precedence in `source-precedence.yaml`
3. Select value from first non-empty source
4. Tag with provenance (which source provided it)
5. If no source available or sources conflict → add to unresolved

### 3. Content Generation Layer

**Purpose**: Convert normalized evidence into structured content for specific formats.

**One Generator Per Content Type**:
- **Release Notes Generator**: evidence → release-notes.yaml
- **Features Page Generator**: evidence → features.yaml
- **Overview Generator**: evidence → overview.yaml
- **API Changes Generator**: evidence → api-changes.yaml

**Each Generator**:

1. Reads `release-evidence.yaml`
2. Filters items by relevance (features for features page, all for release notes, etc.)
3. Maps fields using content-type-specific mapping file:
   - `templates/release-notes/evidence-to-release-notes-mapping.yaml`
   - `templates/features/evidence-to-features-mapping.yaml`
   - etc.
4. Outputs structured YAML matching content-type schema

**Key**: All generators consume the same evidence. No extraction or precedence logic in generators.

### 4. Validation Layer

**Purpose**: Ensure generated content meets quality and policy standards before rendering.

**Validators**:

1. **Schema Validator**: Structured content matches schema
   - `templates/release-notes/release-notes-schema.yaml`
   - Catches missing required fields, invalid types
2. **Policy Rules Validator**: Content passes policy checks
   - `rules/release-notes/release-notes-rule.yaml`
   - Structural rules (STR-*), content completeness (CON-*), security (SEC-*), prose (PRO-*)
3. **Quality Checks**: Style and test results
   - Vale: prose style and language
   - Doc Detective: documentation quality
   - Custom checks: no placeholder text, markdown rendering
4. **Confidence Gate**: Only auto-publish if confidence >= 85
   - 85-100: Auto-include
   - 60-84: Include with warnings
   - <60: Manual review required

**Failure Handling**:
- If validation fails on any item, report failing field(s)
- Stop rendering
- Move item to unresolved for manual review
- Do NOT publish partial or compromised docs

### 5. Rendering Layer

**Purpose**: Convert validated structured content to publishable Markdown.

**Responsibility**:
- Apply deterministic rendering rules (e.g., `release-notes-rendering.yaml`)
- Format dates, lists, links, tables
- Generate table of contents
- Ensure consistent typography and structure

**NOT handled here**:
- Content decisions (which items to include) — decided in validation
- Prose quality — checked in validation
- Precedence or extraction — handled in reconciliation

## Standards Repository Organization (Current → Target)

### Current State (Imported)

```
standards/
  content-types.yaml          ← registry
  content-types/
    release-notes.yaml        ← manifest
  classification-rules.yaml   ← classification
  references.yaml

templates/
  release-notes/
    release-notes-schema.yaml
    release-notes-rendering.yaml
    release-notes-quick-reference.md
    examples/

rules/
  release-notes/
    release-notes-rule.yaml

skills/
  release-notes/
    SKILL.md                  ← current approach: all logic in skill

.github/                      ← partial mirror (drift risk)
  skills/release-notes/
  rules/release-notes/
```

### Target State (Execution Plan)

```
standards/
  content-types.yaml          ← registry (stable)
  content-types/
    release-notes.yaml        ← manifest (stable)
  classification-rules.yaml   ← classification rules (stable)
  source-precedence.yaml      ← NEW: field-level precedence
  release-evidence-fields.yaml ← NEW: field semantics
  references.yaml

templates/
  release-notes/
    release-notes-schema.yaml  ← structure authority (stable)
    release-notes-rendering.yaml ← rendering authority (stable)
    evidence-to-release-notes-mapping.yaml ← NEW: evidence → schema mapping
    release-notes-quick-reference.md
    examples/
      release-evidence.sample.yaml ← NEW: sample evidence

rules/
  release-notes/
    release-notes-rule.yaml   ← policy rules (stable)
  release-evidence-confidence-rule.yaml ← NEW: confidence scoring
  release-evidence-structural-rule.yaml ← NEW: structural validation

skills/
  release-notes/
    SKILL.md                  ← UPDATED: orchestration only
    extraction.md             ← NEW: extraction procedure
    reconciliation.md         ← NEW: reconciliation procedure
    generation.md             ← NEW: generation procedure
    validation.md             ← NEW: validation procedure

docs/
  course-correction/
    release-evidence-contract-v1.md ← NEW: canonical evidence model
    release-evidence-coverage-checklist.md ← NEW: QA requirements
    evidence-signals-vale-doc-detective.md ← NEW: signal integration
    pilot-runbook-single-release.md ← NEW: first release workflow
    decisions-log-standards-vs-product.md ← NEW: governance

  architecture/
    agentic-doc-pipeline-target-state.md ← THIS FILE
    registry-manifest-architecture.md    ← existing (link to)
```

## Critical Design Decisions

### 1. Extraction Is Separate from Precedence

**Decision**: Extraction (pulling from GitHub) and precedence (choosing which source wins) are separate layers.

**Why**:
- Extraction is straightforward and low-risk.
- Precedence is policy and can change per organization.
- Separating them makes precedence machine-readable and testable.

**Implication**:
- Extract all sources, even if redundant.
- Apply precedence in reconciliation, not extraction.

### 2. Canonical Evidence Model Is Normalized But Not Schema-Specific

**Decision**: `release-evidence.yaml` is organization-independent. Content-type-specific normalization happens during generation (via mapping files), not during reconciliation.

**Why**:
- One evidence file serves multiple content types (release notes, features page, API changes, etc.).
- Don't lock evidence format to one schema.
- Mapping files can evolve without re-reconciling.

**Implication**:
- Evidence includes fields for all content types.
- Mapping files select relevant fields and transform them per schema.

### 3. Validation Happens Before Rendering

**Decision**: Validate structured YAML before rendering Markdown. If validation fails, don't render.

**Why**:
- Easier to debug and fix YAML than published Markdown.
- Can report failing fields with line numbers.
- Confidence gates applied before render, not after.

**Implication**:
- Validation is deterministic (pass/fail, no warnings in output).
- Failed items don't make it to published docs.

### 4. Confidence Scoring Is Observable

**Decision**: Every item has a confidence score with component breakdown. Score determines auto-publish gate.

**Why**:
- Makes automation quality visible.
- Auto-publish threshold can be tuned.
- Low-confidence items are flagged, not silently included.

**Implication**:
- Operators can see why an item is/isn't auto-published.
- 60-84 range includes warnings; <60 requires manual review.

## Transition Plan (Weeks)

### Week 1: Foundation
- Define release-evidence contract ✓
- Define source-precedence rules ✓
- Define field glossary ✓
- Create target-state architecture (this document) ✓

### Week 2: Pipeline Refactor (Release Notes First)
- Create sample evidence file
- Create evidence-to-release-notes mapping
- Update release-notes skill to consume evidence instead of extracting

### Week 3: Quality Gates
- Define release-evidence coverage checklist
- Define confidence scoring rules
- Document Vale + Doc Detective signal integration

### Week 4: Pilot & Governance
- Run one complete release cycle end-to-end
- Document decisions and next steps
- Publish pilot results

## Success Criteria

By end of Week 4:

1. **Separation is clear**: Extraction, reconciliation, generation, validation are distinct and auditable.
2. **Evidence model works**: One evidence file successfully feeds multiple content generators.
3. **Precedence is deterministic**: Field-level rules in YAML, not embedded in prompts.
4. **Validation is effective**: Failed items don't make it to published docs.
5. **Confidence is observable**: Operators understand auto-publish vs. manual review decisions.
6. **Pilot is successful**: One complete release cycle end-to-end, with measurable precision and recall.

## Future Evolution

Once this 4-week cycle is complete:

1. Build extractors for other content types (features page, overview, API changes).
2. Package as CLI tool (docintel) once extraction/reconciliation are stable.
3. Add multi-repo aggregation.
4. Integrate with deployment pipelines for automated updates.
5. Extend to non-GitHub sources (JIRA, Slack, email).

## References

- [../course-correction/release-evidence-contract-v1.md](../course-correction/release-evidence-contract-v1.md)
- [../course-correction/documentation-intelligence-cli-v1-spec.md](../course-correction/documentation-intelligence-cli-v1-spec.md)
- [../course-correction/git-extraction-precedence-and-release-evidence-plan.md](../course-correction/git-extraction-precedence-and-release-evidence-plan.md)
- [../../standards/source-precedence.yaml](../../standards/source-precedence.yaml)
- [../../standards/release-evidence-fields.yaml](../../standards/release-evidence-fields.yaml)
