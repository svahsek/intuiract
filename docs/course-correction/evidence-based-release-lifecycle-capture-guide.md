# Evidence-Based Release Lifecycle Capture Guide

## Purpose

This guide explains each stage of a GitHub-based software release lifecycle from planning to post-release, including:

1. Who is involved
2. What typically happens
3. Nuances that create documentation risk
4. What evidence should be captured for automation
5. Recommended quality gates before moving forward

The goal is to help you design content automation on top of trustworthy lifecycle evidence, not only on generated summaries.

## Core principle

Use this model for all stages:

1. Intent: What was planned and why
2. Change: What was implemented
3. Evidence: What was verified and shipped

Automation should combine all three before generating publishable documentation.

## Lifecycle map

1. Product planning and release framing
2. Issue creation and backlog refinement
3. Release scoping and milestone commitment
4. Branch strategy and environment readiness
5. Development execution and commits
6. Pull request review and merge governance
7. QA design, execution, and defect loop
8. Release hardening and freeze
9. Tagging, release creation, and publication
10. Post-release monitoring and corrections
11. Documentation generation and governance layer across all stages

## Stage 1: Product planning and release framing

### Actors

1. Product Manager
2. Engineering Lead or Architect
3. Technical Writer
4. QA Lead

### What happens

1. Product goals are defined
2. Scope themes are identified (feature, enhancement, bug reduction, compliance)
3. Target release window is set
4. High-level acceptance expectations are discussed

### Nuances

1. Teams often use strategy language, not implementation language
2. Scope can look stable but still has unknowns
3. Priority can shift after technical discovery

### Capture points

1. Release objective statement
2. Theme tags (for example performance, onboarding, API stability)
3. Initial risk assumptions
4. Target release identifier (for example 1.8.0)
5. Intended audience segments

### Why it matters for docs automation

This gives context for Overview and Summary sections and prevents release notes from being only a list of code changes.

## Stage 2: Issue creation and backlog refinement

### Actors

1. Product Manager
2. Developers
3. QA
4. Technical Writer (for doc-impact tagging)

### What happens

1. Work items are created as GitHub Issues
2. Labels, type, priority, and area fields are assigned
3. Acceptance criteria are defined or refined
4. Dependencies are identified

### Nuances

1. Inconsistent labeling is the biggest quality killer for automation
2. Some teams close issues through PRs without clean linkage
3. Issue descriptions may mix business intent and technical detail

### Capture points

1. Issue type (feature, enhancement, bug, security, deprecation)
2. Product area label
3. Documentation impact field (overview, features, user guide, install guide, API)
4. Acceptance criteria
5. Dependency links
6. Severity and priority

### Recommended gate

No issue enters committed scope unless required metadata fields are present.

## Stage 3: Release scoping and milestone commitment

### Actors

1. Product Manager
2. Engineering Manager
3. QA Lead
4. Release Manager

### What happens

1. Issues are assigned to a release milestone
2. Scope is baselined
3. Stretch items are separated from committed items

### Nuances

1. Teams often keep optional work in the same milestone, creating ambiguity
2. Late additions are sometimes not tracked as scope changes

### Capture points

1. Milestone assignment history
2. Committed vs stretch flag
3. Scope change log (added, removed, deferred)
4. Rationale for scope changes

### Recommended gate

Any scope change after commitment must include reason and owner approval.

## Stage 4: Branch strategy and environment readiness

### Actors

1. Developers
2. QA
3. DevOps or Platform team

### What happens

1. Release branch is created
2. QA or stabilization branch may be created
3. Environment configuration baseline is established

### Nuances

1. Branch naming drift causes extraction errors
2. Hotfixes may bypass normal flow
3. Environment parity gaps can invalidate test evidence

### Capture points

1. Branch naming map
2. Branch purpose (feature, release, qa, hotfix)
3. Environment and config snapshot
4. Feature flags status per branch

### Recommended gate

Standard branch conventions must be enforced before automated extraction starts.

## Stage 5: Development execution and commits

### Actors

1. Developers
2. Engineering Lead

### What happens

1. Code is implemented on work branches
2. Commits are pushed with incremental changes
3. Work is linked to issues

### Nuances

1. Commit messages vary heavily in quality
2. Squash merges can lose commit granularity
3. Code may include docs-impactful changes with no explicit metadata

### Capture points

1. Commit SHA and timestamp
2. Commit message classification hints
3. Linked issue id
4. Changed file paths and modules
5. API or config surface touched

### Recommended gate

Require issue references in commit or PR metadata for traceability.

## Stage 6: Pull request review and merge governance

### Actors

1. Developers
2. Reviewers or Maintainers
3. QA (for validation context)
4. Security reviewers when needed

### What happens

1. PR created with summary and linked issues
2. CI checks run
3. Review comments and requested changes happen
4. Merge decision and strategy applied

### Nuances

1. PR title and description quality strongly affects doc quality
2. Labels may be added late or inconsistently
3. Merge strategy affects how history is interpreted later

### Capture points

1. PR title and body
2. Linked issues and cross-references
3. Labels and project fields at merge time
4. Review outcomes and approvals
5. CI status matrix
6. Merge commit SHA and strategy

### Recommended gate

A PR should not merge without minimum metadata: type label, linked issue, user impact statement.

## Stage 7: QA design, execution, and defect loop

### Actors

1. QA Engineers
2. Developers
3. Product Manager
4. Release Manager

### What happens

1. Test cases are created or updated
2. Test runs execute across target environments
3. Defects are logged and triaged
4. Re-test cycles continue until exit criteria are met

### Nuances

1. Test case systems may be external and poorly linked to GitHub
2. Teams may track pass status but not feature-level coverage
3. Blocked tests can be ignored near deadlines

### Capture points

1. Test case ids linked to issue or PR
2. Test scope (functional, regression, integration, security)
3. Execution result and run timestamp
4. Environment details
5. Defect links and severity
6. Waivers and approved exceptions

### Recommended gate

For each committed release item, require explicit QA evidence or approved waiver.

## Stage 8: Release hardening and freeze

### Actors

1. Release Manager
2. Engineering Lead
3. QA Lead
4. Product Manager

### What happens

1. Feature freeze or code freeze starts
2. Only approved fixes allowed
3. Final risk review and go or no-go meetings happen

### Nuances

1. Freeze policy exceptions are common and often undocumented
2. Last-minute fixes can skip full test evidence

### Capture points

1. Freeze start time
2. Exception approvals
3. Remaining known issues list
4. Risk acceptance notes

### Recommended gate

No undocumented freeze exceptions should be allowed into final release evidence.

## Stage 9: Tagging, release creation, and publication

### Actors

1. Release Manager
2. DevOps
3. Maintainers

### What happens

1. Release tag is created
2. Release artifact or package is produced
3. Release notes and metadata are published

### Nuances

1. What is in milestone is not always what shipped
2. Final tag boundary is the strongest source for shipped truth
3. Manual tagging mistakes can break downstream extraction

### Capture points

1. Base tag and target tag
2. Exact commit range included
3. Released repositories or modules
4. Artifact checksums and links
5. Published release URL

### Recommended gate

Documentation generation should use the final tag boundary as shipped truth, then reconcile with planning data.

## Stage 10: Post-release monitoring and corrections

### Actors

1. Support
2. QA
3. Engineering
4. Product Manager
5. Technical Writer

### What happens

1. Incidents, regressions, and user feedback appear
2. Known issues list is updated
3. Patch or hotfix scope is created

### Nuances

1. Teams often forget to feed post-release findings back to docs
2. Hotfixes can bypass standard metadata hygiene

### Capture points

1. Post-release defects linked to release version
2. Known issue status changes
3. Workaround quality and validation status
4. Patch release linkage

### Recommended gate

Every hotfix should include delta documentation evidence, not only code changes.

## Cross-stage evidence model you should build

Create a canonical release item model so each content type uses one truth source.

Suggested fields:

1. item_id
2. release_version_target
3. issue_refs
4. pr_refs
5. commit_refs
6. change_type
7. user_impact
8. technical_scope
9. qa_evidence
10. shipped_status
11. known_issue_status
12. doc_areas_affected
13. confidence_score
14. unresolved_warnings

## Precedence model by field

Use field-level precedence, not source-level precedence.

1. Business intent fields: issues and project fields first
2. Implementation fields: PR metadata and changed files first
3. Shipped inclusion fields: tag boundary first
4. Validation fields: QA evidence first
5. Narrative fallback: generated summary only when upstream evidence is missing

## Capture taxonomy for richer automation

Define standard labels and project fields before scaling automation.

Suggested labels:

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

Suggested required fields:

1. release_target
2. change_type
3. user_impact
4. migration_required
5. related_test_cases
6. doc_areas_affected

## Minimum automation readiness checklist

1. Issue and PR templates enforce required metadata
2. Label taxonomy is documented and used consistently
3. Branch naming and release tagging conventions are stable
4. QA evidence can be linked per release item
5. A canonical evidence file is generated before content generation
6. Validation rules block publishing on missing critical evidence

## Practical adoption path

1. Start with one release pilot and one or two content types
2. Measure precision and missing evidence rate
3. Fix metadata gaps in templates and workflow
4. Expand to overview, user guide, installation guide, and API changes
5. Keep human approval gate until confidence stabilizes

## What this enables for technical writing

With this approach, you move from writing after engineering to governing the evidence quality that powers repeatable, high-trust documentation automation.
