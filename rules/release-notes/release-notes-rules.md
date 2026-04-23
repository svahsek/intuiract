---
type: rules
id: release-notes
title: Release Notes Validation Rules
domain: documentation
scope: release-notes
purpose: Validate release notes for structure, completeness, and quality
version: 1.0.0
status: active
owner: documentation
audience: agents
format: markdown-rules
appliesTo:

CHANGELOG.md
release-notes.md
docs/releases/**/*.md
validationMode: advisory
ruleIdPrefix: RULE
tags:
release-notes
validation
documentation-quality
---

# Release Notes Validation Rules

**Purpose**: This document defines validation rules for release notes. AI agents should validate against these rules and report violations categorized by severity.

---

## RULE CATEGORIES & SEVERITY LEVELS

- **ERROR**: Must fix before release (blocks validation)
- **WARNING**: Should fix (may impact release quality)
- **INFO**: Nice to have (informational)

---

## SECTION: Structure & Format

### RULE-001: Required Sections
- **Severity**: ERROR
- **Valid if**: Release notes contain ALL of these sections:
  1. Release Header (with version & date)
  2. At least ONE of: New Features, Bug Fixes, Enhancements, or Improvements
- **Invalid examples**:
  - A release with only "New Features" but no actual features listed
  - Missing release date
- **Valid example**:
  ```
  # Version 2.1.0 - Released 16 February 2026
  
  ## New Features
  - Feature 1
  
  ## Bug Fixes
  - Fix 1
  ```

### RULE-002: Version Format
- **Severity**: ERROR
- **Pattern**: Must match `X.Y.Z` (semantic versioning) or `X.Y.Z-suffix` format
  - Valid: `1.0.0`, `2.1.3`, `1.0.0-beta.1`, `2.0.0-rc1`
  - Invalid: `1.0`, `v1.0.0`, `1.0.0.0.0`, `latest`
- **Violation**: Report if version doesn't match format

### RULE-003: Release Date Format
- **Severity**: ERROR
- **Pattern**: Must be `DD Month YYYY` or `D Month YYYY` format
  - Valid: `3rd May, 2026`
  - Invalid: `2024-05-03`, `May 3 2024`, `05/03/2024`, `TBD`, missing date

---

## SECTION: Content Requirements

### RULE-004: Issue References for Bug Fixes
- **Severity**: WARNING
- **Valid if**: Each bug fix entry includes issue ID reference
  - Pattern: `#123` or `issue #123` or `GH-123`
- **Invalid example**: "Fixed memory leak" (no issue reference)
- **Valid example**: "Fixed memory leak (#1234)"

### RULE-005: No Placeholder Text
- **Severity**: ERROR
- **Prohibited keywords**: TBD, TODO, FIXME, WIP, Coming Soon, TK, [REDACTED], [UPDATE ME]
- **Violation**: Report if any prohibited keywords found
- **Rationale**: Release notes should be complete, not drafts

### RULE-006: Breaking Changes Must Be Flagged
- **Severity**: ERROR if breaking changes exist without flag
- **Valid if breaking changes have:
  1. ⚠️ warning emoji
  2. "BREAKING CHANGE:" prefix
  3. Migration steps or guidance
  4. Affected version ranges
- **Valid example**:
  ```
  ⚠️ BREAKING CHANGE: Removed deprecated `legacyAPI()` method
  - Migration: Use `newAPI()` instead
  - Affects: v2.x and earlier
  ```

### RULE-007: Link Validation
- **Severity**: WARNING
- **Valid if**: All hyperlinks are complete
  - Pattern: Links in markdown format `[text](https://...)` 
  - Invalid: Incomplete URLs like `[docs](./docs)` or bare URLs like https://example.com
- **Rationale**: Ensures all reference links work

### RULE-008: Language & Clarity
- **Severity**: WARNING
- **Check for**:
  - Marketing language: "revolutionary", "game-changing", "amazing"
  - Jargon without explanation: Use plain language
  - Unclear wording: Below 120 characters per entry (guideline)
- **Valid example**: "Added pagination API to improve performance with large datasets"
- **Invalid example**: "Implemented revolutionary async paradigm shift"

### RULE-009: Consistency in Formatting
- **Severity**: INFO
- **Rules**:
  - Use consistent bullet style (- or * or •)
  - Use consistent tense (present or past)
  - Follow pattern: `[Action] [What] [Why/Impact]`
- **Example pattern**:
  ```
  - Fixed database query timeout (Issue #456)
  - Added support for custom webhooks
  - Improved error messages for clarity
  ```

---

## SECTION: Completeness Checks

### RULE-010: Entry Descriptions
- **Severity**: WARNING
- **Valid if**: Each entry has sufficient description
  - Minimum: 10 words or one complete sentence
  - Maximum: 200 characters (encourages clarity)
- **Invalid**: "Fixed bugs" (too vague)
- **Valid**: "Fixed race condition in concurrent request handling"

### RULE-011: Version Coordination
- **Severity**: ERROR
- **Valid if**: Version in header matches any version references in content
- **Check**: No conflicting version numbers

### RULE-012: Section Not Empty
- **Severity**: WARNING
- **Valid if**: Sections with headers have at least one entry
- **Invalid**: A `## Bug Fixes` section with no items listed

---

## SECTION: What NOT to Include

### RULE-013: Prohibited Content
- **Severity**: ERROR for HIGH-RISK, WARNING for MEDIUM-RISK
- **Prohibited (HIGH)**:
  - Internal team names
  - Internal Jira/GitHub issue URLs without context
  - Confidential information
  - Personal developer names (use "team" instead)
- **Prohibited (MEDIUM)**:
  - Comparative marketing ("better than competitors")
  - Emotional language without context
  - Incomplete sentences
  - Excessive punctuation!!!

---

## VALIDATION WORKFLOW

When validating a release note, check rules in this order:

1. **Structural** (RULE-001, 002, 003): Does it have proper format?
2. **Content** (RULE-004 through 013): Does content meet quality standards?
3. **Polish** (RULE-009, 010): Is it well-written?

**Output Format for Violations**:
```
❌ RULE-005: No Placeholder Text
   Severity: ERROR
   Found: "TBD - more features coming"
   Location: Line 12, under "Enhancements"
   Fix: Replace with actual content or remove section
```

---

## Example: Valid Release Notes Structure

```markdown
# Version 2.1.0 - Released 16 February 2026

## New Features
- Added webhook support for real-time notifications (#2401)
- Implemented batch API operations to improve performance

## Bug Fixes
- Fixed race condition in concurrent request handling (#2398)
- Corrected date formatting in export output (#2397)

## Enhancements
- Improved error messages for better debugging experience
- Optimized database queries reducing latency by 40%

⚠️ BREAKING CHANGE: Removed deprecated `v1/auth` endpoint
- **Migration**: Use `/oauth/token` endpoint instead
- **Affects**: v2.0.x and earlier
- See [Migration Guide](https://docs.example.com/migration)

## Deprecations (Optional)
- Deprecated `legacyMethod()` - will be removed in v3.0
```

---

## How to Use This File with an AI Agent

**Instructions for validation AI**:
1. Parse the entire release note
2. For each RULE (001-013), check if the condition passes
3. Report all ERRORS first (blocks release)
4. Report WARNINGS (quality issues)
5. Provide specific line numbers and suggestions
