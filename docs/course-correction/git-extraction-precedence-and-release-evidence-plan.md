# Git Extraction Precedence and Release Evidence Plan

You want a rigorous precedence model for Git-based extraction across the full release lifecycle, including Git Issues, branches, commits and PRs, QA test cases, and release tags.

This direction moves from generating only at release time to accumulating evidence through the release lifecycle, which is what makes automated documentation reliable.

## Proposed precedence model

Use three layers, not one linear source:

1. Intent layer
2. Change layer
3. Evidence layer

Apply precedence per field, not per artifact.

### 1. Intent layer

Primary sources:

1. Git Issues such as type, labels, milestone, and linked parent issue
2. GitHub Projects fields such as status, target release, owner, and priority

Why:

1. This is the original what and why and planned scope.
2. It is the best source for feature, enhancement, and bug categorization at business level.

### 2. Change layer

Primary sources:

1. PR title and body, linked issues, labels, and files changed
2. Commit history within the PR, or squash commit message if squash merged

Why:

1. This is where implementation intent is concretized.
2. It is better than raw commits for narrative because the PR has context and review outcome.

### 3. Evidence layer

Primary sources:

1. Release tag diff boundary
2. QA test case execution results and test evidence
3. Build artifacts, deployment notes, and release branch state

Why:

1. This proves what actually shipped.
2. It prevents documenting planned-but-not-shipped work.

## Field-level precedence

Do not say issues beat commits globally. Define precedence by field.

1. Change type such as feature, enhancement, bug, or security:
   1. Issue labels and type
   2. PR labels and type
   3. Commit prefix fallback
2. User impact statement:
   1. Issue description
   2. PR description
   3. Generated summary from diff
3. Technical details:
   1. PR files changed and diff
   2. Commit messages
   3. Issue comments fallback
4. Shipped-in-release inclusion:
   1. Release tag boundary and merged PR SHA inclusion
   2. Release branch inclusion
   3. Issue milestone fallback only as advisory
5. QA validation status:
   1. Linked QA test run results
   2. PR checks
   3. Manual QA notes as fallback
6. Known issues:
   1. Open blocker or high-severity issues in release milestone
   2. Post-merge defects linked to shipped PRs
   3. Manual override list

## Lifecycle extraction model

Treat release extraction as staged aggregation.

1. Stage A: Planning start, driven by Issues and Projects
   1. Capture release candidate scope from milestone and project fields.
   2. Build initial release workset.
2. Stage B: Implementation in progress, driven by branches and PRs
   1. Track development and QA branches mapped to the release.
   2. Continuously enrich the workset with PR metadata and labels.
   3. Detect drift such as PR merged without linked issue, or issue closed without PR.
3. Stage C: Stabilization, driven by QA
   1. Ingest QA test cases and execution outcomes.
   2. Attach pass or fail and coverage evidence to each release item.
   3. Surface missing tests for high-risk changes.
4. Stage D: Freeze and ship, driven by tags
   1. Final truth set from tag range and included merge commits.
   2. Exclude planned-but-unshipped items.
   3. Generate release notes and related content types from final evidence graph.

## How to include QA test cases cleanly

Create a QA evidence contract per release item:

1. test_case_ids
2. test_scope such as functional, regression, integration, or security
3. execution_status such as pass, fail, blocked, or not_run
4. execution_link
5. environment such as staging or preprod
6. last_execution_date

Validation rules to add:

1. Major or Minor release: each feature or enhancement must have at least one executed test case.
2. Any failed critical test blocks ready-for-release state.
3. Patch or Hotfix allows reduced scope but still requires regression evidence for touched areas.

## Suggested GitHub-native structure after JIRA migration

Standardize labels and project fields first, or automation quality will stay noisy.

### Labels

1. type:feature
2. type:enhancement
3. type:bug
4. type:security
5. type:deprecation
6. area:api
7. area:installation
8. area:user-guide
9. qa:required
10. qa:passed
11. release:target-x.y.z
12. severity:high

### Required issue and PR fields

1. release_target
2. change_type
3. user_impact
4. migration_required
5. related_test_cases
6. doc_areas_affected

### Branch conventions

1. release/x.y.z
2. qa/release-x.y.z
3. feature or bug branches must reference issue id

## Conflict resolution policy

When sources disagree, use deterministic rules.

1. Inclusion conflict such as issue says done but tag says not included:
   1. Tag wins for shipped content.
2. Type conflict such as issue says bug but PR says enhancement:
   1. PR label wins if explicitly set.
   2. Otherwise issue type wins.
3. QA conflict such as PR checks pass but QA suite fails:
   1. QA suite wins for release readiness.
4. Missing links:
   1. Keep item as unverified and downgrade confidence.

## Confidence scoring for auto-publish gate

Score each release item out of 100.

1. Linked issue with proper labels: +20
2. PR linked to issue: +20
3. PR has semantic labels and clean description: +15
4. Included in release tag boundary: +25
5. QA evidence attached and passing: +20

Policy:

1. 85 and above: auto-include
2. 60 to 84: include with warning
3. Below 60: manual review required

## Practical implementation plan

1. Define a canonical intermediate model.
   1. Use a file such as release-workset.yaml with item-level evidence fields.
2. Build extractors in order.
   1. Git Issues and Projects extractor
   2. PR and commit extractor
   3. Tag boundary reconciler
   4. QA evidence importer
3. Build reconciler.
   1. Merge records by issue id, PR number, and commit SHA.
   2. Apply field-level precedence and conflict rules.
4. Build validators.
   1. Structural
   2. Consistency
   3. QA completeness
   4. Confidentiality checks
5. Use this reconciled model as single input for all content types.
   1. Release notes
   2. Features page
   3. Overview
   4. User guide delta
   5. Installation guide delta
   6. API changes
6. Start with one release pilot.
   1. Measure precision and missing-item rate.
   2. Tighten labels and required fields.
   3. Then automate further.

## What to change in the current skill next

For the Resolve Input section, split the fallback into explicit extraction phases:

1. Plan extraction using Issues and Projects
2. Delivery extraction using PRs, commits, and branches
3. Verification extraction using QA tests and results
4. Shipment reconciliation using release tag boundary
5. Confidence and unresolved warnings

This keeps the existing standards and template architecture intact while making extraction richer and auditable.
