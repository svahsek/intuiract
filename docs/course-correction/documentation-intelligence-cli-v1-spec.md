# Documentation Intelligence CLI v1 Specification

## Purpose

Define a practical v1 command-line product that organizations can use to automate documentation generation from engineering evidence with governance and validation.

## Product Positioning

### Product name (working)

Documentation Intelligence CLI

### Product goal

Ingest release evidence from GitHub, reconcile it using deterministic precedence rules, and generate validated documentation artifacts.

### Primary users

1. Technical Writers
2. Release Managers
3. DevEx and Platform teams
4. Engineering leads managing release communications

## v1 Scope

### In scope

1. Single repository support
2. GitHub as primary source system
3. Release-focused extraction and generation
4. Evidence model creation per release
5. Release notes generation
6. Validation with rule-based report
7. Markdown and YAML output

### Out of scope (v1)

1. Multi-repo aggregation
2. Full web dashboard
3. Automatic publishing to multiple portals
4. Advanced AI-only autonomous generation

## Core v1 Workflow

1. Extract source evidence for a target release range
2. Reconcile entities and apply field-level precedence
3. Generate canonical evidence artifact
4. Generate release documentation from evidence
5. Validate output and emit warnings and errors

## CLI Commands

### 1) Extract

Command:

```bash
docintel extract \
  --repo owner/repo \
  --base-tag v0.5.0 \
  --target-tag v0.6.0 \
  --out .docintel/runs/0.6.0/
```

Purpose:

1. Pull issues, PRs, commits, tags, releases
2. Build raw evidence snapshots

Outputs:

1. issues.json
2. prs.json
3. commits.json
4. tags.json
5. extract-metadata.json

### 2) Reconcile

Command:

```bash
docintel reconcile \
  --run .docintel/runs/0.6.0/ \
  --precedence .docintel/config/source-precedence.yaml \
  --out .docintel/runs/0.6.0/reconciled/
```

Purpose:

1. Link issues, PRs, commits, QA evidence
2. Resolve conflicts and ownership by field
3. Create canonical release evidence

Outputs:

1. release-evidence.yaml
2. unresolved-items.yaml
3. reconciliation-report.txt

### 3) Generate

Command:

```bash
docintel generate \
  --run .docintel/runs/0.6.0/reconciled/ \
  --content-type release-notes \
  --template .docintel/templates/release-notes.yaml \
  --out docs/releases/
```

Purpose:

1. Transform canonical evidence into content artifact
2. Produce structured YAML and markdown draft

Outputs:

1. 0.6.0-release-notes.yaml
2. 0.6.0-release-notes.md

### 4) Validate

Command:

```bash
docintel validate \
  --input docs/releases/0.6.0-release-notes.yaml \
  --rules .docintel/rules/human-rules.yaml \
  --out docs/releases/0.6.0-validation-report.txt
```

Purpose:

1. Enforce STR, CON, SEC style rules
2. Produce deterministic pass and fail report

Outputs:

1. validation-report.txt
2. non-zero exit code on blocking errors

### 5) Full pipeline

Command:

```bash
docintel run \
  --repo owner/repo \
  --base-tag v0.5.0 \
  --target-tag v0.6.0 \
  --config .docintel/config/docintel.yaml
```

Purpose:

1. Execute extract + reconcile + generate + validate in one shot

## Configuration Model

File:

.docintel/config/docintel.yaml

```yaml
version: 1
repository:
  provider: github
  repo: owner/repo

release:
  base_tag: v0.5.0
  target_tag: v0.6.0
  release_type: Minor

inputs:
  qa_evidence_file: .docintel/inputs/qa-evidence.yaml
  classification_rules: .docintel/config/classification-rules.yaml
  precedence_rules: .docintel/config/source-precedence.yaml

generation:
  content_types:
    - release-notes
  output_dir: docs/releases

validation:
  rules_file: .docintel/rules/human-rules.yaml
  fail_on:
    - ERROR

confidence:
  auto_include_threshold: 80
  warning_threshold: 60
```

## Canonical Evidence Contract (v1)

File:

release-evidence.yaml

```yaml
release:
  version: 0.6.0
  base_tag: v0.5.0
  target_tag: v0.6.0
  release_date: 2026-04-23

items:
  - item_id: REL-001
    issue_refs: [142]
    pr_refs: [233]
    commit_refs: ["abc1234"]
    change_type: feature
    user_impact: Users can filter items by category.
    technical_scope:
      modules: [ui]
      files:
        - src/filter.js
        - src/home.html
    qa_status: pass
    shipped_status: shipped
    doc_areas_affected:
      - release-notes
      - features-page
    confidence_score: 90
    warnings: []

unresolved:
  - item_id: REL-009
    reason: Missing linked issue for merged PR #240
    severity: WARNING
```

## Source Precedence Rules (machine-readable)

File:

.docintel/config/source-precedence.yaml

```yaml
version: 1
field_ownership:
  change_type:
    primary: issue_labels
    fallback:
      - pr_labels
      - commit_prefix

  user_impact:
    primary: issue_description
    fallback:
      - pr_summary

  technical_scope:
    primary: pr_files_changed
    fallback:
      - commit_diff

  shipped_status:
    primary: release_tag_boundary

  qa_status:
    primary: qa_evidence

conflict_policy:
  shipped_conflict: tag_wins
  type_conflict: pr_label_over_issue_if_explicit
  qa_conflict: qa_wins
```

## QA Evidence Input Contract (v1)

File:

.docintel/inputs/qa-evidence.yaml

```yaml
release: 0.6.0
results:
  - issue_ref: 142
    pr_ref: 233
    test_case_id: QA-88
    test_scope: regression
    environment: staging
    status: pass
    executed_at: 2026-04-20T10:40:00Z
    report_url: https://example.test/report/qa-88
```

## Output Artifacts

For a release 0.6.0:

1. docs/releases/0.6.0-release-evidence.yaml
2. docs/releases/0.6.0-release-notes.yaml
3. docs/releases/0.6.0-release-notes.md
4. docs/releases/0.6.0-validation-report.txt
5. docs/releases/0.6.0-reconciliation-report.txt

## Validation Policy

### Blocking errors

1. Missing release metadata
2. Missing shipped boundary (base or target tag)
3. Major or Minor release item with no QA evidence
4. Invalid output schema

### Warning-level checks

1. Missing issue link on merged PR
2. Missing user impact text
3. Low confidence item below warning threshold

## Recommended Tech Stack for CLI

### Primary recommendation

Node.js + TypeScript

Why:

1. Strong GitHub API ecosystem
2. Easy packaging and distribution
3. Good YAML and validation libraries
4. Good maintainability for config-driven tools

### Suggested libs

1. Commander or Yargs for CLI
2. Octokit for GitHub API
3. js-yaml for YAML parsing
4. Zod or Ajv for schema validation
5. Chalk for output formatting

## Security and Governance (v1)

1. Token-based GitHub auth via environment variable only
2. No token or secret logging
3. Optional redaction pass for sensitive content
4. Audit log for extraction and reconciliation steps

## CI Integration Pattern

### GitHub Actions example flow

1. Trigger on tag push or release branch push
2. Run docintel run command
3. Upload generated artifacts
4. Fail build on validation errors
5. Open PR with generated docs changes (optional)

## 4-Week Build Plan

### Week 1: Foundation

1. CLI skeleton and command framework
2. Config loader and validator
3. Extract command with GitHub API integration

### Week 2: Reconciliation engine

1. Entity linking logic
2. Field-level precedence implementation
3. release-evidence.yaml output generation

### Week 3: Generate and validate

1. release-notes generator from evidence
2. validation engine with error and warning reporting
3. output artifact writer

### Week 4: Hardening and demo

1. End-to-end pipeline command
2. CI workflow integration
3. sample repository demo run
4. documentation and starter templates

## Success Criteria for v1

1. 80 percent of release items auto-linked with no manual intervention
2. deterministic regeneration from same tags yields same artifacts
3. validation catches missing critical evidence before publishing
4. one-command pipeline works in CI for at least one production-like repo

## v2 Expansion Path

1. Additional content types: features page, overview, API changes
2. Multi-repo and portfolio release support
3. Organization-level policy packs
4. Optional dashboard with review workflow

## Final Recommendation

Start narrow and make v1 dependable. A deterministic evidence and validation core is more valuable than broad but unstable generation. Build trust first, then expand coverage.
