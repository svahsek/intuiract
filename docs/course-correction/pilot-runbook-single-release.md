# Pilot Runbook: Single Release End-to-End

## Purpose

Step-by-step procedure to execute one complete release cycle through the agentic documentation pipeline. Use this runbook to validate that all pieces work together and to identify gaps before full automation.

**Target**: Prove that release notes can be generated from GitHub evidence deterministically (extraction → reconciliation → generation → validation → rendering).

**Timeline**: 1-2 hours per release cycle

---

## Prerequisites

Before starting the pilot:

- [ ] Repository has release tag in semantic format (e.g., `v1.2.3`)
- [ ] Repository has GitHub Issues (milestones, labels)
- [ ] Repository has PRs (linked to issues, conventional titles)
- [ ] Repository has commits (conventional format: feat:, fix:, security:)
- [ ] Repository has QA test results or build logs
- [ ] Vale and Doc Detective installed and configured (optional for pilot v1)
- [ ] All standards files in place:
  - [ ] `release-evidence-contract-v1.md`
  - [ ] `source-precedence.yaml`
  - [ ] `release-evidence-fields.yaml`
  - [ ] `release-notes-schema.yaml`
  - [ ] `evidence-to-release-notes-mapping.yaml`
  - [ ] `release-evidence-confidence-rule.yaml`
  - [ ] `release-notes-rule.yaml`

---

## Phase 1: Extraction (30 minutes)

### Step 1.1: Define Release Scope

**Inputs**:
- Previous release tag (base): `v1.2.0`
- Current release tag (target): `v1.2.3` (or pre-created tag)

**Action**:
```bash
# Navigate to repository
cd /path/to/repository

# List commits between tags
git log v1.2.0..v1.2.3 --oneline | head -20

# Count commits
git log v1.2.0..v1.2.3 --oneline | wc -l

# Record count: ____ commits in scope
```

**Output**: Scope confirmed; commit range known

---

### Step 1.2: Extract GitHub Issues

**Inputs**:
- GitHub milestone for release (e.g., `v1.2.3` or date-based)
- Issue types: feature, bug, enhancement, security, deprecation

**Action**:

Option A: Manual extraction (for pilot)
```bash
# List issues in milestone
# Go to: https://github.com/YOUR_ORG/YOUR_REPO/issues?q=milestone:v1.2.3

# For each issue, note:
# - Issue #
# - Title
# - Labels (type:feature, area:api, etc.)
# - Description (user impact statement)
# - Linked PRs (if any)

# Record: ____ issues found
```

Option B: Scripted extraction (future)
```bash
# GitHub CLI to extract issues
gh issue list \
  --state all \
  --milestone "v1.2.3" \
  --json number,title,labels,body \
  > issues.json

# Count: $(cat issues.json | jq '.[] | select' | wc -l) issues
```

**Output**: `issues.json` (or manual list)

---

### Step 1.3: Extract GitHub PRs

**Inputs**:
- PR merge commits between base_tag and target_tag
- PR title, description, review status, labels

**Action**:

```bash
# GitHub CLI to extract PRs
gh pr list \
  --state all \
  --search "merged:v1.2.0..v1.2.3" \
  --json number,title,body,reviews,labels,commits \
  > prs.json

# Count: $(cat prs.json | jq '.[] | select' | wc -l) PRs

# For each PR, verify:
# - [ ] PR is linked to issue (Fixes #1234)
# - [ ] PR title follows convention (feat:, fix:, security:)
# - [ ] PR has at least 1 approval
# - [ ] PR body explains "what" and "why"
```

**Output**: `prs.json`

---

### Step 1.4: Extract QA Test Results

**Inputs**:
- Build/CI pipeline logs
- QA test case IDs (if using dedicated QA system)
- Security scan results

**Action**:

```bash
# Option 1: From CI workflow logs
# Go to Actions tab → Latest successful build for v1.2.3
# Look for:
# - Build status: ✓ PASSED
# - Test results: Unit tests (N passed), Integration tests (N passed)
# - Security scan: ✓ No critical findings
# - Lint: ✓ Passed

# Copy test summary:
# "15 unit tests passed, 5 integration tests passed, 0 security issues"

# Option 2: From QA system (future)
# Query QA test case IDs from: API, spreadsheet, or dedicated tool
```

**Output**: QA summary string recorded

---

### Step 1.5: Extract Commit Messages

**Inputs**:
- All commits between tags (git log output)

**Action**:

```bash
# Extract commit messages
git log v1.2.0..v1.2.3 --format="%h %s" > commits.txt

# Example output:
# a1b2c3d feat: add OAuth2 flow
# b2c3d4e fix: race condition in registration
# c3d4e5f docs: update API reference

# For each commit, note:
# - Commit hash
# - Message prefix (feat:, fix:, docs:, etc.)
# - What was changed
```

**Output**: `commits.txt`

---

## Phase 2: Reconciliation (20 minutes)

### Step 2.1: Create Release Evidence YAML

**Inputs**:
- `issues.json`
- `prs.json`
- `commits.txt`
- QA summary
- `source-precedence.yaml` (authority)
- `release-evidence-fields.yaml` (field semantics)

**Action**:

Create `release-evidence.yaml` using template structure:

```yaml
# release-evidence.yaml (for v1.2.3 release)

version: "1.0"
release_metadata:
  version: "v1.2.3"
  release_date: "2026-04-23"
  release_type: "Minor"

items:
  
  - item_id: "REL-001"
    intent:
      github_issue_number: 1234
      title: "Self-service OAuth2 integration"
      description: "Partners can configure OAuth2 flow without support"
      user_impact: "Integrators achieve faster onboarding; support ticket volume reduced"
    
    change:
      github_pr_number: 5678
      change_type: "feature"
      pr_title: "feat: OAuth2 self-service configuration"
      pr_body: "Adds UI for OAuth2 configuration with validation and error handling"
      changed_files:
        - "src/auth/oauth2-config.ts"
        - "src/ui/oauth2-form.tsx"
        - "tests/oauth2-e2e.test.ts"
    
    evidence:
      linked_issue_reference: true
      pr_code_review_status: "approved"
      qa_test_case_ids: ["QA-TC-2341", "QA-TC-2342", "QA-TC-2343"]
      qa_tests_passed: 3
      qa_tests_failed: 0
      qa_test_date: "2026-04-22"
      build_status: "passed"
      security_scan_status: "passed"
      regression_test_status: "passed"
    
    shipped:
      included_in_tag: "v1.2.3"
      shipped_date: "2026-04-23"
      shipped_status: true
    
    confidence:
      score: 98
      scoring_breakdown: {linkage: 20, documentation: 20, labeling: 15, tag: 25, qa: 18}
      tier: "auto_include"
    
    normalized_for_release_notes:
      feature_name: "Self-Service OAuth2 Integration"
      feature_description: "Partners can now configure OAuth2 authentication directly in the platform, eliminating manual support tickets."
      use_cases:
        - "Set up OAuth2 without technical support"
        - "Migrate from static credentials to OAuth2"
      capabilities:
        - "OAuth2 configuration UI with validation"
        - "Automatic scope mapping"
        - "Test connection before saving"
      documentation_link: "https://docs.example.com/oauth2"
      related_issue: "https://github.com/ORG/REPO/issues/1234"

  - item_id: "REL-002"
    intent:
      github_issue_number: 5679
      title: "Fix: Race condition in concurrent registrations"
      description: "When multiple registrations happen simultaneously, records may be created in inconsistent state"
      user_impact: "Eliminates data corruption in high-concurrency scenarios"
    
    change:
      github_pr_number: 5680
      change_type: "bug"
      pr_title: "fix: add mutex for concurrent registration state"
      pr_body: "Added distributed lock to prevent concurrent modification of registration state"
      changed_files:
        - "src/registration/state-manager.ts"
        - "tests/registration-concurrent.test.ts"
    
    evidence:
      linked_issue_reference: true
      pr_code_review_status: "approved"
      qa_test_case_ids: ["QA-TC-2344", "QA-TC-2345"]
      qa_tests_passed: 2
      qa_tests_failed: 0
      qa_test_date: "2026-04-22"
      build_status: "passed"
      security_scan_status: "passed"
      regression_test_status: "passed"
    
    shipped:
      included_in_tag: "v1.2.3"
      shipped_date: "2026-04-23"
      shipped_status: true
    
    confidence:
      score: 92
      tier: "auto_include"
    
    normalized_for_release_notes:
      bug_title: "Fixed Race Condition in Concurrent Registration"
      bug_description: "Resolved issue where simultaneous registration requests could result in inconsistent database state due to missing synchronization."
      severity: "High"
      versions_affected: ["v1.2.0", "v1.2.1", "v1.2.2"]
      workaround: "None (critical fix)"
      related_issue: "https://github.com/ORG/REPO/issues/5679"

# ... additional items ...

unresolved: []
```

**Acceptance**:
- [ ] All items have item_id (REL-001, REL-002, etc.)
- [ ] All items have intent, change, evidence, shipped sections
- [ ] All items have confidence score with tier assigned
- [ ] All confidence scores are calculated per `release-evidence-confidence-rule.yaml`
- [ ] `unresolved[]` array contains any items with confidence < 60 or critical gaps

**Output**: `release-evidence.yaml` (populated)

---

### Step 2.2: Validate Evidence Schema

**Inputs**:
- `release-evidence.yaml`
- `release-evidence-contract-v1.md` (schema definition)

**Action**:

```bash
# Manual validation (for pilot)
# For each item, check:

for each item:
  ✓ item_id exists (REL-NNN format)
  ✓ intent.github_issue_number is number
  ✓ intent.title is string, not empty, no TBD/TODO
  ✓ intent.user_impact is string, one sentence, benefit-driven
  
  ✓ change.github_pr_number is number
  ✓ change.change_type is valid enum (feature, bug, security, enhancement, deprecation)
  ✓ change.pr_title is string, follows convention
  ✓ change.changed_files is array of strings
  
  ✓ evidence.* fields match expected types
  ✓ shipped.included_in_tag matches target tag (v1.2.3)
  ✓ shipped.shipped_status is boolean (should be true for released items)
  
  ✓ confidence.score is 0-100
  ✓ confidence.tier matches score (auto_include/include_with_warning/manual_review)
  ✓ normalized_for_release_notes fields are populated

# If all checks pass:
echo "✓ Evidence schema validation PASSED"
```

**Output**: Validation report (Pass/Fail + field errors if any)

---

## Phase 3: Content Generation (15 minutes)

### Step 3.1: Load Mapping Specification

**Inputs**:
- `release-evidence.yaml`
- `evidence-to-release-notes-mapping.yaml` (mapping authority)

**Action**:

Read mapping file to understand:
- Which evidence fields map to which release-notes schema fields
- Filtering rules (e.g., features must have confidence >= 85)
- Transformations (e.g., trim_v_prefix on versions)

```yaml
# Example mapping (from evidence-to-release-notes-mapping.yaml):
metadata:
  version: "from release_metadata.version | trim_v_prefix()"
  releaseDate: "from release_metadata.release_date"
  
content:
  newFeatures:
    filter: "change_type == 'feature' AND confidence >= 85"
    mapping:
      name: "from normalized_for_release_notes.feature_name"
      description: "from normalized_for_release_notes.feature_description"
      issues: "from [github_issue_number]"
```

**Output**: Mapping understood

---

### Step 3.2: Filter Evidence Items

**Inputs**:
- `release-evidence.yaml`
- Mapping filter rules
- Confidence thresholds

**Action**:

```bash
# Apply filtering rules:

# Features (confidence >= 85):
features = []
for item in release-evidence.items:
  if item.change.change_type == "feature" AND item.confidence.score >= 85:
    features.append(item)
count_features = len(features)  # e.g., 3 features

# Bug Fixes (confidence >= 60):
bugs = []
for item in release-evidence.items:
  if item.change.change_type == "bug" AND item.confidence.score >= 60:
    bugs.append(item)
count_bugs = len(bugs)  # e.g., 2 bugs

# Security Fixes (all):
security = []
for item in release-evidence.items:
  if item.change.change_type == "security" AND item.shipped.shipped_status == true:
    security.append(item)
count_security = len(security)  # e.g., 1 security

# Summary:
echo "Filtering results: $count_features features, $count_bugs bugs, $count_security security"
```

**Output**: Filtered item lists

---

### Step 3.3: Populate Structured Content (YAML)

**Inputs**:
- Filtered items from Step 3.2
- Mapping transformations
- Operator input (manual fields)

**Action**:

Create `release-notes.yaml` by mapping evidence fields:

```yaml
# release-notes.yaml (for v1.2.3)

metadata:
  version: "1.2.3"  # trimmed from v1.2.3
  releaseDate: "2026-04-23"
  releaseType: "Minor"
  productName: "CRVS Integration Platform"  # manual input
  productAbbreviation: "CRVS"

summary:
  headline: "Self-Service Client Management Accelerates Integrator Onboarding"  # manual input
  paragraphs:
    - "This release focuses on partner self-service capabilities, reducing onboarding time and support burden."  # manual input
  keyBenefits:
    - "Faster partner onboarding with self-service OAuth2 configuration"
    - "Improved system stability with race condition fixes"

content:
  newFeatures:
    - featureName: "Self-Service OAuth2 Integration"
      description: "Partners can now configure OAuth2 authentication..."
      useCase: "Set up OAuth2 without technical support"
      capabilities:
        - "OAuth2 configuration UI with validation"
        - "Automatic scope mapping"
      documentationLink: "https://docs.example.com/oauth2"
      issueLinks:
        - "https://github.com/ORG/REPO/issues/1234"
      confidence: 98
  
  bugFixes:
    - title: "Fixed Race Condition in Concurrent Registration"
      description: "Resolved issue where simultaneous registration requests..."
      severity: "High"
      affectedVersions: ["v1.2.0", "v1.2.1", "v1.2.2"]
      workaround: "None (critical fix)"
      issueLinks:
        - "https://github.com/ORG/REPO/issues/5679"
      confidence: 92
  
  securityFixes:
    - title: "SQL Injection Vulnerability Patched"
      severity: "Critical"
      description: "Fixed SQL injection in query builder..."
      cveId: "CVE-2026-1234"
      affectedVersions: ["v1.2.0", "v1.2.1", "v1.2.2"]
      upgradeRequired: true

callToAction:
  upgradeTimeline: "Recommended: Upgrade within 2 weeks"
  supportLink: "https://support.example.com"
```

**Acceptance**:
- [ ] Metadata populated (version, date, product name)
- [ ] Summary has manual headline and paragraphs
- [ ] All features with confidence >= 85 included
- [ ] All bugs with confidence >= 60 included
- [ ] All security fixes included (no confidence threshold)
- [ ] No placeholder text (TBD, TODO, FIXME)
- [ ] All URLs are valid format

**Output**: `release-notes.yaml` (structured content)

---

## Phase 4: Validation (15 minutes)

### Step 4.1: Schema Validation

**Inputs**:
- `release-notes.yaml`
- `release-notes-schema.yaml`

**Action**:

```bash
# Validate YAML against schema
# Check required fields:

✓ metadata.version
✓ metadata.releaseDate
✓ metadata.productName
✓ summary.headline
✓ summary.paragraphs (at least 1)
✓ content section (features, bugs, or security)

# Check type constraints:
✓ version is semver format
✓ releaseDate is ISO date format
✓ headline is string, not > 100 chars
✓ confidence scores are 0-100

# If all pass:
echo "✓ Schema validation PASSED"
```

**Output**: Validation report (Pass/Fail with field-level errors if any)

---

### Step 4.2: Policy Validation

**Inputs**:
- `release-notes.yaml`
- `release-notes-rule.yaml` (policy rules)

**Action**:

```bash
# Run policy checks:

# Rule STR-001: Release notes must have headline and summary
if metadata.version AND summary.headline AND summary.paragraphs.length >= 1:
  echo "✓ STR-001 PASS: Structure complete"
else:
  echo "✗ STR-001 FAIL: Missing required sections"

# Rule CON-001: No placeholder text
if "TBD" or "TODO" or "FIXME" in content:
  echo "✗ CON-001 FAIL: Placeholder text found"
else:
  echo "✓ CON-001 PASS: No placeholders"

# Rule CON-002: All issue links valid
for issue_link in all_issue_links:
  if issue_link matches regex "https://github.com/.+/issues/\d+":
    echo "✓ Issue link valid: $issue_link"
  else:
    echo "✗ Issue link invalid: $issue_link"

# If all pass:
echo "✓ Policy validation PASSED"
```

**Output**: Policy validation report

---

### Step 4.3: Confidence Gate

**Inputs**:
- `release-notes.yaml` with confidence scores
- Confidence rules from `release-evidence-confidence-rule.yaml`
- Release type: Minor (threshold >= 75 for features, >= 60 for others)

**Action**:

```bash
# Check confidence thresholds per release type (Minor):

for feature in content.newFeatures:
  if feature.confidence >= 75:
    echo "✓ Feature '$feature.name' confidence OK: $feature.confidence"
  else:
    echo "⚠ Feature '$feature.name' low confidence: $feature.confidence (threshold: 75)"

for bug in content.bugFixes:
  if bug.confidence >= 60:
    echo "✓ Bug '$bug.title' confidence OK: $bug.confidence"
  else:
    echo "⚠ Bug '$bug.title' low confidence: $bug.confidence (threshold: 60)"

# If all meet threshold:
echo "✓ Confidence gate PASSED"
```

**Output**: Confidence gate report

---

### Step 4.4: Final Validation Checkpoint

**Inputs**:
- Schema validation report
- Policy validation report
- Confidence gate report

**Action**:

```bash
# If ANY validation failed:
echo "✗ VALIDATION FAILED"
echo "Action: Fix failing items in release-evidence.yaml or release-notes.yaml"
echo "Do NOT proceed to rendering"

# If ALL validations passed:
echo "✓ ALL VALIDATIONS PASSED"
echo "Proceeding to Phase 5: Rendering"
```

**Output**: Final validation checkpoint (Proceed/Block)

---

## Phase 5: Rendering (10 minutes)

### Step 5.1: Apply Rendering Rules

**Inputs**:
- `release-notes.yaml` (validated)
- `release-notes-rendering.yaml` (rendering authority)

**Action**:

Read rendering rules for format, typography, and structure:

```yaml
# Example rendering rules (from release-notes-rendering.yaml):
metadata:
  version: "Display as: 'Version 1.2.3'"
  releaseDate: "Display as: 'Released on 23 April 2026'"
  
typography:
  headline: "H1 (#)"
  section_titles: "H2 (##)"
  feature_names: "Bold (**...**)"
  code_snippets: "Backticks or code blocks"
  
links:
  issue_links: "[GH-1234](url)"
```

**Output**: Rendering rules understood

---

### Step 5.2: Generate Markdown

**Inputs**:
- `release-notes.yaml` (validated)
- Rendering rules

**Action**:

Transform YAML to Markdown:

```markdown
# CRVS Integration Platform v1.2.3

Released on 23 April 2026

## Overview

This release focuses on partner self-service capabilities, reducing onboarding time and support burden.

## Key Benefits

- Faster partner onboarding with self-service OAuth2 configuration
- Improved system stability with race condition fixes

## New Features

### Self-Service OAuth2 Integration

Partners can now configure OAuth2 authentication directly in the platform...

**Use Cases**:
- Set up OAuth2 without technical support

**Capabilities**:
- OAuth2 configuration UI with validation
- Automatic scope mapping

[Documentation](https://docs.example.com/oauth2) | [Issue GH-1234](https://github.com/ORG/REPO/issues/1234)

## Bug Fixes

### Fixed Race Condition in Concurrent Registration

Resolved issue where simultaneous registration requests could result in inconsistent database state...

**Severity**: High  
**Affected Versions**: v1.2.0, v1.2.1, v1.2.2  

[Issue GH-5679](https://github.com/ORG/REPO/issues/5679)

## Security Updates

### SQL Injection Vulnerability Patched

Fixed SQL injection in query builder...

**Severity**: Critical  
**CVE**: CVE-2026-1234  
**Action Required**: Upgrade recommended within 2 weeks

---

## Support

[Contact Support](https://support.example.com)
```

**Output**: `release-notes.md` (publication-ready)

---

### Step 5.3: Generate Table of Contents

**Inputs**:
- `release-notes.md`

**Action**:

```bash
# Extract headings
# Generate numbered TOC

## Suggested TOC:
1. Overview
2. Key Benefits
3. New Features
   3.1. Self-Service OAuth2 Integration
4. Bug Fixes
   4.1. Fixed Race Condition in Concurrent Registration
5. Security Updates
   5.1. SQL Injection Vulnerability Patched
6. Support
```

**Output**: TOC added to release-notes.md (if template includes it)

---

## Phase 6: Quality Review (10 minutes)

### Step 6.1: Manual Review Checklist

**Inputs**:
- `release-notes.md`

**Action**:

Before publishing, review:

- [ ] Headline is benefit-driven (not hype)
- [ ] No internal jargon or acronyms undefined
- [ ] Tone is consistent (professional, clear)
- [ ] Features explain "what" and "why"
- [ ] Bug descriptions are concise but complete
- [ ] All links are valid
- [ ] No broken formatting (bold, lists, etc.)
- [ ] GitHub issue links are present and correct
- [ ] Confidence indicators are visible if needed
- [ ] Spelling and grammar are correct

**Output**: Quality review sign-off

---

## Pilot Results Template

After completing pilot, document findings:

```yaml
pilot_release: "v1.2.3"
pilot_date: "2026-04-23"
pilot_duration_hours: 1.5

phases_completed:
  extraction: "✓ PASS (30 min)"
  reconciliation: "✓ PASS (20 min)"
  generation: "✓ PASS (15 min)"
  validation: "✓ PASS (15 min)"
  rendering: "✓ PASS (10 min)"
  quality_review: "✓ PASS (10 min)"

evidence_quality:
  total_items: 10
  auto_include_items: 8
  include_with_warning_items: 2
  manual_review_items: 0
  average_confidence: 92

metrics:
  precision: "95%"  # (correct items / all items)
  recall: "100%"  # (items we found / all items in release)
  operator_time: "1.5 hours"
  automation_time: "0 hours"  # (once scripts run automatically)
  
operator_workload:
  manual_inputs: 3  # headline, summary, target_audience
  manual_corrections: 0  # no failed validations
  manual_approvals: 0  # all items auto-include

gaps_identified:
  - gap_id: "GAP-001"
    description: "Some PRs lacked linked issues; manual linking required"
    recommendation: "Add GitHub branch protection rule requiring issue link"
    priority: "High"
    
  - gap_id: "GAP-002"
    description: "QA test case IDs not consistently recorded"
    recommendation: "Add QA logging to CI pipeline"
    priority: "Medium"

blockers: []

lessons_learned:
  - "Source precision works well; no ambiguity in field-level precedence"
  - "Confidence scoring filters low-quality items effectively"
  - "Manual inputs (headline, summary) still required; cannot automate"
  - "Evidence structure is sound for all item types"

next_steps:
  - "Run pilot on 2-3 more releases to validate consistency"
  - "Implement branch protection rule for issue linking"
  - "Add QA logging to CI"
  - "Build extraction tool (currently manual)"
  - "Plan next-quarter expansion to features-page content type"

sign_off:
  by: "Technical Writer / Release Manager"
  date: "2026-04-23"
  approval: "Ready for limited automation (extraction + reconciliation still manual)"
```

---

## Success Criteria

Pilot is successful if:

- ✓ All 5 phases complete without errors
- ✓ Release notes generated from evidence deterministically
- ✓ No validation failures
- ✓ Confidence scores assigned correctly
- ✓ Final markdown is publication-ready
- ✓ Operator time < 2 hours per release
- ✓ No gaps in evidence require post-hoc additions

---

## Troubleshooting

**Symptom**: Low confidence scores  
**Diagnosis**: Check linkage (issue #, PR link), QA evidence, tag boundary  
**Fix**: See [release-evidence-coverage-checklist.md](./release-evidence-coverage-checklist.md)

**Symptom**: Validation failures  
**Diagnosis**: Run validation with --verbose; check field-level errors  
**Fix**: Update source data (evidence) or manual input (release notes) per error message

**Symptom**: Rendering issues (broken formatting)  
**Diagnosis**: Check YAML structure; validate against rendering rules  
**Fix**: Fix YAML syntax; re-render

---

## Version History

- **v1.0.0** (2026-04-23): Initial pilot runbook for single release
