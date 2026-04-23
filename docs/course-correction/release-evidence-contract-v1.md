# Release Evidence Contract v1

## Purpose

Define the canonical intermediate model for normalized release evidence, serving as the single source of truth for all content generators (release notes, features page, overview, API reference, etc.).

This contract captures evidence across four lifecycle stages: Intent, Change, Evidence, and Shipped.

## Core Principle

**Field-level precedence, not source-level opinion.** Each field has an explicit primary source and fallbacks. Reconciliation is deterministic, not prompt-driven.

## Canonical File Format

- **File name**: `release-evidence.yaml` (or per-release: `release-evidence-v1.2.3.yaml`)
- **Location**: `.docintel/runs/[RELEASE_VERSION]/release-evidence.yaml`
- **Authority**: This file is the authoritative source for all downstream content generation.
- **Consumers**: Release notes skill, features-page generators, overview renderers, validation engines.

## Top-Level Structure

```yaml
version: "1.0.0"
release:
  version: "1.2.3"
  codename: "Optional codename"
  release_date: "2026-04-23"
  release_type: "Minor"  # Major, Minor, Patch, Hotfix, LTS
  base_tag: "v1.2.0"
  target_tag: "v1.2.3"

items: []  # Array of release items (see below)
unresolved: []  # Conflicts and missing evidence
metadata:
  extraction_timestamp: "2026-04-23T10:30:00Z"
  reconciliation_timestamp: "2026-04-23T11:00:00Z"
  generated_by: "docintel v0.1.0 or agent workflow name"
```

## Release Item Schema

Each item in `items[]` represents one cohesive unit of work: a feature, enhancement, bug fix, or security update.

```yaml
items:
  - item_id: "REL-001"
    change_type: "feature"  # feature|enhancement|bug|security|deprecation|technical_improvement
    
    # ========== INTENT LAYER (GitHub Issues, Projects) ==========
    intent:
      github_issue_number: 1234
      github_issue_url: "https://github.com/org/repo/issues/1234"
      issue_title: "Self-service OAuth2 client provisioning"
      issue_description: "Partners need automated way to create clients"
      issue_labels: ["type:feature", "area:auth", "priority:high"]
      issue_milestone: "v1.2.3"
      project_fields:
        status: "Done"
        priority: "High"
        target_release: "v1.2.3"
        owner: "alice@example.com"
      user_impact: "Users can now self-provision OAuth2 clients without manual intervention"
    
    # ========== CHANGE LAYER (PRs, Commits, Branches) ==========
    change:
      pr_number: 5678
      pr_url: "https://github.com/org/repo/pull/5678"
      pr_title: "feat: self-service OAuth2 client provisioning"
      pr_description: "Implements ability for partners to create clients via Admin Portal"
      pr_labels: ["type:feature", "area:auth", "qa:passed"]
      merged_at: "2026-04-22T15:30:00Z"
      merged_by: "bob@example.com"
      
      commits:
        - sha: "abc1234def5678"
          message: "feat: add self-service client creation endpoint"
          author: "alice@example.com"
        - sha: "xyz9876abc1234"
          message: "test: add integration tests for client provisioning"
          author: "bob@example.com"
      
      files_changed:
        - path: "src/api/oauth/client-provisioning.ts"
          lines_added: 250
          lines_deleted: 0
        - path: "src/ui/admin/client-management.vue"
          lines_added: 180
          lines_deleted: 0
      
      branch_name: "feature/oauth-self-service"
      base_branch: "main"
    
    # ========== EVIDENCE LAYER (QA, Tests, Validation) ==========
    evidence:
      qa_status: "pass"  # pass|fail|partial|not_tested
      qa_test_cases:
        - test_id: "QA-TC-2341"
          test_name: "Client creation with valid scopes"
          scope: "functional"
          status: "pass"
          executed_at: "2026-04-22T10:00:00Z"
          environment: "staging"
        - test_id: "QA-TC-2342"
          test_name: "Error handling for duplicate client"
          scope: "functional"
          status: "pass"
          executed_at: "2026-04-22T10:15:00Z"
          environment: "staging"
        - test_id: "QA-TC-2343"
          test_name: "Regression: existing client operations"
          scope: "regression"
          status: "pass"
          executed_at: "2026-04-22T10:30:00Z"
          environment: "staging"
      
      pr_checks_passed: true
      build_status: "pass"
      security_scan_status: "pass"
      security_scan_warnings: []
      
      documentation_links:
        feature_docs: "https://docs.example.com/features/oauth-provisioning"
        api_docs: "https://api-docs.example.com/clients/post"
        integration_guide: null
    
    # ========== SHIPPED LAYER (Release Tags, Deployment) ==========
    shipped:
      included_in_release: true
      release_tag: "v1.2.3"
      tag_commit_sha: "main-abc1234def5678"
      deployed_to_production: false  # Set true after production deployment
      deployment_timestamp: null  # Set after production deployment
    
    # ========== CONFIDENCE & METADATA ==========
    confidence:
      score: 95  # 0-100; auto-include if >= 85, warning if 60-84, manual if < 60
      score_breakdown:
        linked_issue: 20
        pr_linked: 20
        semantic_labels: 15
        tag_boundary: 25
        qa_evidence: 15
      flags:
        - "Complete issue linkage"
        - "All QA tests passing"
      warnings: []
    
    # ========== NORMALIZATION OUTPUT (for schema mapping) ==========
    normalized_for_release_notes:
      feature_name: "Self-Service OAuth2 Client Management"
      feature_description: "Partners can now create and manage OAuth2 clients directly through the Admin Portal without manual intervention."
      feature_use_case: "Reduce partner onboarding time by enabling self-service client provisioning."
      feature_key_capabilities:
        - "Create OAuth2 clients with custom scopes"
        - "Generate and rotate client secrets automatically"
        - "View audit logs of client activity"
      feature_documentation_link: "https://docs.example.com/features/oauth-provisioning"
      feature_related_issues:
        - "#1234"
        - "#1235"
```

## Unresolved Items

Items with missing or conflicting evidence go into `unresolved[]`:

```yaml
unresolved:
  - item_id: "REL-009"
    reason: "Missing linked issue for merged PR #240"
    severity: "warning"  # warning|error
    pr_number: 240
    pr_url: "https://github.com/org/repo/pull/240"
    detected_at: "2026-04-22T18:00:00Z"
    resolution_action: "Manual review required: Does PR #240 belong in v1.2.3?"
  
  - item_id: "REL-010"
    reason: "QA test failure: QA-TC-2450 failed but PR merged"
    severity: "error"
    pr_number: 241
    qa_test_id: "QA-TC-2450"
    qa_test_status: "fail"
    detected_at: "2026-04-22T18:15:00Z"
    resolution_action: "Investigate failed test; may block release."
```

## Field Ownership & Precedence

See [../standards/source-precedence.yaml](../../standards/source-precedence.yaml) for machine-readable precedence rules.

High-level summary:

| Field | Primary Source | Fallback 1 | Fallback 2 |
|-------|---|---|---|
| `change_type` | Issue labels + type | PR labels + type | Commit prefix |
| `user_impact` | Issue description | PR description | Generated from diff |
| `technical_scope` | PR files changed + diff | Commit messages | Issue comments |
| `shipped_status` | Release tag boundary | Release branch | Issue milestone |
| `qa_status` | QA test execution | PR checks | Manual notes |

## Lifecycle Stages Reference

1. **Intent Layer**: Issues + Projects. Answers "What" and "Why".
2. **Change Layer**: PRs + commits + branches. Answers "How" and "What was implemented".
3. **Evidence Layer**: QA tests + build status + security scans. Answers "Does it work?"
4. **Shipped Layer**: Release tags + deployment records. Answers "Is it live?"

## Quality Assurance

### Validation Rules (Applied Before Rendering)

1. **Structural**: Every required field is populated.
2. **Linkage**: Each item has at least one link to GitHub (issue, PR, or commit).
3. **QA Coverage**: Features and enhancements have at least one QA test result (for Major/Minor releases).
4. **Confidence Threshold**: Items below confidence score 60 must be manually reviewed.
5. **Shipped Consistency**: If `shipped.included_in_release == true`, then `shipped.release_tag` must be set.

### Validation Failure Handling

If validation fails on any item:

1. Report failing field(s) with reason.
2. Stop rendering.
3. Move item to `unresolved[]` for manual review.
4. Do not publish partial or compromised release notes.

## Example Usage

### Extraction Phase

Extract from GitHub Issues, PRs, QA systems, and release tags; populate fields per precedence rules.

### Reconciliation Phase

Merge duplicate items (same issue linked by multiple PRs), resolve conflicts (PR says bug, issue says feature), apply confidence scoring.

### Generation Phase (Release Notes Example)

For each item in `items[]` with confidence >= 60:
1. Read `normalized_for_release_notes` section.
2. Use [../templates/release-notes/evidence-to-release-notes-mapping.yaml](../../templates/release-notes/evidence-to-release-notes-mapping.yaml) to map fields to release-notes schema.
3. Validate mapped payload against release-notes schema.
4. Render Markdown.

### Validation Phase

Run [../rules/release-evidence-confidence-rule.yaml](../../rules/release-evidence-confidence-rule.yaml) and [../rules/release-notes/release-notes-rule.yaml](../../rules/release-notes/release-notes-rule.yaml) before publishing.

## Extending This Contract

For new content types (features page, API reference, overview):

1. Add new `normalized_for_[CONTENT_TYPE]` fields to the item schema.
2. Create new mapping files in `templates/[CONTENT_TYPE]/`.
3. Update reconciliation and confidence scoring logic if needed.

The core evidence contract stays the same; content generators extend it.

## Version History

- **v1.0.0** (2026-04-23): Initial canonical contract aligned to release-notes automation pilot.
