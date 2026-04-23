# Release Evidence Coverage Checklist

## Purpose

Define QA/coverage requirements for each lifecycle stage to ensure release evidence is complete before rendering documentation.

This checklist is used during the **Validation Phase** to confirm the release meets minimum quality gates.

## Lifecycle Stages & Requirements

### Stage 1: Intent (Planning)

**Timeline**: When issue is created through feature planning

**Checklist** (all required for confidence > 60):

- [ ] GitHub issue created with title and description
  - Description must explain "what" and "why"
  - No placeholder text (TBD, TODO, FIXME)
- [ ] Issue type set (feature, enhancement, bug, security, deprecation)
  - Via GitHub issue type OR label (type:feature, type:bug, etc.)
- [ ] Issue assigned to milestone (e.g., v1.2.3)
- [ ] User impact statement provided
  - One sentence: what user gets or what problem is solved
  - No marketing hype
- [ ] For breaking changes: flagged with label or documented
  - Label: breaking-change or migration:required
  - Or explicitly marked in description: "BREAKING CHANGE:"

**Coverage Questions**:
- Do we know what this change is trying to accomplish?
- Do we have a clear link to who requested it or why?
- Can we articulate the user/business value?

**Acceptance Criteria**:
- All required fields populated
- No ambiguity about scope or intent
- Links to related issues/PRs present

### Stage 2: Change (Implementation in Progress)

**Timeline**: When PR is created through merge

**Checklist** (all required for confidence > 60):

- [ ] PR is linked to issue
  - Via "Linked Issues" field OR
  - Issue reference in PR title (#1234) OR
  - "Fixes #1234" in PR body
- [ ] PR title follows conventional format
  - Recommended: feat:, fix:, enhance:, security:
  - At minimum: readable and descriptive
- [ ] PR description explains:
  - What was changed
  - Why it was changed this way
  - Any breaking changes or migration notes
  - References to related test cases
- [ ] PR is labeled with type and area
  - type:feature, type:bug, etc.
  - area:api, area:ui, area:auth, etc.
- [ ] All commits in PR have messages
  - Not auto-generated or "WIP" messages
  - Reference issue if applicable (#1234)
- [ ] PR files changed are sensible
  - Code changes match PR description
  - No unrelated files in commit
- [ ] PR review is complete
  - At least one approval from code owner
  - Comments addressed or dismissed with rationale

**Coverage Questions**:
- Can we trace from issue to implementation?
- Do we understand what code changed?
- Was this change reviewed and approved?
- Are there any breaking changes documented?

**Acceptance Criteria**:
- Issue-PR linkage is clear and bidirectional
- PR description explains the "why" not just the "what"
- Code review is complete
- No unresolved comments

### Stage 3: Evidence (Validation & QA)

**Timeline**: During testing and before release

**Checklist** (per release type):

**For Major/Minor Releases** (all required for confidence >= 70):
- [ ] Functional tests pass
  - Test case IDs documented (QA-TC-2341, etc.)
  - All tests executed and passing
  - Environment: staging
  - Test date documented (within 5 days of release)
- [ ] Regression tests pass
  - At least one regression test per changed module
  - Covers existing workflows unaffected
- [ ] Integration tests pass (if applicable)
  - Multi-system interactions tested
  - API contracts verified
- [ ] Security tests pass
  - No critical/high SAST findings
  - Dependency scan shows no unpatched CVEs
  - No SQL injection/XSS vulnerabilities
- [ ] Performance baseline maintained
  - Response times within SLA
  - No memory leaks detected
- [ ] PR status checks all green
  - Build passes
  - Linter passes
  - Security scan passes

**For Patch Releases** (reduced scope):
- [ ] Functional tests pass for affected area
  - Minimum 1-2 test cases per fix
- [ ] Regression tests pass for touched modules
  - Verify no new issues in existing code
- [ ] Build and linter pass

**For Hotfix** (emergency mode):
- [ ] At least manual verification by author
  - Can be expedited but must be documented
  - Risk acknowledgment required

**Coverage Questions**:
- Has this been tested thoroughly?
- Are we confident it works?
- Did we verify no side effects?
- Are there documented test results?

**Acceptance Criteria**:
- All required tests executed and documented
- No failing tests blocking release
- QA sign-off or explicit risk acceptance

### Stage 4: Shipped (Release Tags & Deployment)

**Timeline**: At release commit and post-deployment

**Checklist** (all required for confidence >= 85):

- [ ] Release tag created
  - Format: v1.2.3 (semantic versioning)
  - Tag message includes release notes summary
- [ ] Release tag points to correct commit
  - All intended PRs merged
  - No unintended commits included
- [ ] Git tag diff is sensible
  - `git log base_tag..target_tag` shows expected commits only
  - No stray merges from other branches
- [ ] Release artifacts built
  - Docker image tagged
  - Package published to registry
  - Build metadata captured (SHA, timestamp)
- [ ] Documentation updated
  - Release notes published
  - API docs updated (if applicable)
  - Migration guide published (if breaking changes)
  - Changelog updated
- [ ] Deployment verified
  - Deployment to staging confirmed
  - Smoke tests pass
  - Logs show no errors
- [ ] Production deployment
  - Deployment to production completed
  - Health checks passing
  - No critical alerts
  - Rollback plan documented (if needed)

**Coverage Questions**:
- Is the tag accurate and complete?
- Has the code been built and published?
- Is it actually live in production?
- Can we prove what version is running?

**Acceptance Criteria**:
- Tag correctly represents release scope
- All artifacts built and published
- Deployment verified in all environments
- Release is live and stable

## Coverage Scoring

Map coverage to confidence tiers:

| Confidence | Intent | Change | Evidence | Shipped | Typical Items |
|---|---|---|---|---|---|
| **85-100** (Auto-Include) | ✓ All | ✓ All | ✓ All | ✓ All | Features with full QA, security fixes, high-impact enhancements |
| **60-84** (Include with Warning) | ✓ All | ✓ All | ⚠ Partial (60-70% tests) | ✓ Most | Bug fixes with basic regression test, enhancements with partial QA |
| **<60** (Manual Review) | ⚠ Gaps | ⚠ Gaps | ✗ Little/None | ⚠ Gaps | Unlinked PRs, untested changes, items without issue reference |

## Usage in Validation Pipeline

**During Release Evidence Reconciliation:**

```
For each item in evidence:
  1. Check against this checklist per lifecycle stage
  2. Count passes and gaps
  3. Calculate coverage %
  4. Assign confidence tier:
     - All stages >= 85% → confidence 85-100
     - At least 60% → confidence 60-84
     - Less than 60% → confidence <60 (manual review)
  5. If release_type == Major/Minor and confidence < 60 → unresolved
  6. If release_type == Patch and confidence < 50 → flag warning
```

## Release-Type-Specific Gates

### Major Release

**Minimum Evidence Requirements**:
- ✓ Every feature must have QA tests (functional + regression)
- ✓ Every security fix must have security scan pass
- ✓ Breaking changes must have migration guide
- ✓ Documentation must be updated
- **Gate**: If any feature has coverage < 60%, block release

### Minor Release

**Minimum Evidence Requirements**:
- ✓ Features: all have functional tests
- ✓ Enhancements: at least regression tests
- ✓ Bugs: at least basic verification
- **Gate**: If any feature has coverage < 70%, require manual approval

### Patch Release

**Minimum Evidence Requirements**:
- ✓ Bug fixes: developer tested + smoke verified
- ✓ Enhancements: basic verification
- **Gate**: Reduced scope; can proceed with 50% coverage if urgent

### Hotfix (Emergency)

**Minimum Evidence Requirements**:
- ✓ Author verification required
- ✓ Risk documented
- **Gate**: Expedited but with explicit risk acceptance

## Checklist Failure Scenarios & Recovery

**Scenario 1: Feature has no QA tests**
- [ ] Confidence < 60 (manual review required)
- [ ] Action: Either run tests before release OR explicitly accept risk
- [ ] If accepted: Document decision in release notes

**Scenario 2: Bug fix has no regression tests**
- [ ] Confidence 60-70 (include with warning)
- [ ] Action: Either add regression test OR accept 60% confidence
- [ ] If accepted: Monitor post-deployment closely

**Scenario 3: PR has no linked issue**
- [ ] Confidence drops significantly
- [ ] Action: Either find/create issue and link OR manually verify intent
- [ ] If verified: Re-score and update confidence

**Scenario 4: Deployment not verified**
- [ ] Confidence capped at 70% (not shipped yet)
- [ ] Action: Run smoke tests and verify production deployment
- [ ] Once verified: Update shipped stage and re-score

## Reporting Coverage

**Coverage Report Template** (per release):

```yaml
release: v1.2.3
coverage_summary:
  total_items: 10
  intent_stage:
    complete: 9
    partial: 1
    coverage: 90%
  change_stage:
    complete: 10
    coverage: 100%
  evidence_stage:
    complete: 8
    partial: 2
    coverage: 80%
  shipped_stage:
    complete: 10
    coverage: 100%

confidence_breakdown:
  auto_include: 8  (confidence >= 85%)
  include_with_warning: 2  (confidence 60-84%)
  manual_review: 0  (confidence < 60%)

gaps:
  - item_id: REL-005
    stage: evidence
    issue: "No regression tests"
    recommendation: "Add 1-2 regression tests before release"

gate_status: PASS
  reason: "All items >= 60% coverage; no blockers"
```

## Version History

- **v1.0.0** (2026-04-23): Initial coverage checklist aligned to 4-stage lifecycle.
