# Evidence Layer Syntax for Non-Developers

This guide explains how to define and capture evidence in a non-developer-friendly way, focusing on what events to capture, at which stage, and how to represent them in simple syntax.

## Core Concept

Think of one release as a timeline of events. Each event has:
1. A type (what happened)
2. A source (where it came from)
3. A timestamp (when it happened)
4. Links (connections to issues, PRs, commits)

Automate content from events, not from raw git text directly.

## 7 Event Types to Start With

Do not start with 30 fields. Start with these seven:

1. Issue planned
2. Issue scoped to release
3. PR merged
4. Commit included in release branch
5. QA test executed
6. Release tag created
7. Known issue recorded

If you capture these well, you can generate strong release content.

## Capture by Stage (What and When)

### Planning Stage
Capture: issue id, type, labels, release target, user impact
Source: GitHub Issues and Projects

### Build Stage
Capture: PR id, linked issue, changed files, merge date
Source: Pull Requests

### Integration Stage
Capture: commit SHA, branch, module touched
Source: git commits and branch history

### QA Stage
Capture: test case id, pass or fail, environment, defect links
Source: QA tracker or issue comments

### Release Stage
Capture: base tag, target tag, included PRs, release date
Source: git tags and GitHub Releases

### Post-Release Stage
Capture: regressions, workarounds, hotfix link
Source: new issues after release

## Simple Evidence Syntax

Create one file like this for each release cycle.

Example structure:

```yaml
release:
  version: 0.6.0
  base_tag: v0.5.0
  target_tag: v0.6.0
  release_date: 2026-04-22

events:
  - id: EVT-001
    stage: planning
    event_type: issue_scoped
    source: github_issue
    timestamp: 2026-04-10T09:30:00Z
    refs:
      issue: 142
    data:
      change_type: feature
      labels: [type:feature, area:ui, release:target-0.6.0]
      user_impact: Users can filter by category.

  - id: EVT-018
    stage: build
    event_type: pr_merged
    source: github_pr
    timestamp: 2026-04-14T16:22:00Z
    refs:
      pr: 233
      issue: 142
    data:
      files_changed:
        - src/filter.js
        - src/home.html

  - id: EVT-031
    stage: qa
    event_type: qa_test_executed
    source: qa_report
    timestamp: 2026-04-18T11:10:00Z
    refs:
      issue: 142
      pr: 233
    data:
      test_case_id: QA-88
      result: pass
      environment: staging

  - id: EVT-044
    stage: release
    event_type: release_tag_created
    source: git_tag
    timestamp: 2026-04-21T08:00:00Z
    refs:
      tag: v0.6.0
    data:
      included_prs: [233, 229, 227]
```

This is enough to power release notes, features page, and known issues.

## Field Ownership

Define one owner source per field to avoid confusion later.

1. Change type: issue labels first, PR labels second
2. User impact: issue description first, PR summary second
3. Technical scope: PR files changed first, commits second
4. Shipped status: release tag boundary only
5. QA status: QA evidence only

This prevents conflicting sources and makes debugging easier.

## Minimum Rules to Enforce

Keep it simple:

1. Every merged PR must link to at least one issue
2. Every release item must have change type
3. Major or minor release items must have QA evidence
4. Only items inside base tag to target tag are considered shipped
5. Any item without enough evidence is marked review-required

## Confidence Scoring (Easy Version)

Give each release item a score out of 100:

1. Linked issue: +25
2. Linked merged PR: +25
3. Present in release tag boundary: +30
4. QA pass evidence: +20

Policy:

1. 80 and above: auto-include
2. 60 to 79: include with warning
3. Below 60: manual review

## What to Learn First (No Coding Required)

In order:

1. Issue and label discipline
2. PR linking discipline
3. Tag and release boundary correctness
4. QA evidence attachment
5. Then schema refinement

You can operate most of this with process design and YAML editing, not deep coding.

## Your 2-Week Learning Plan

### Week 1

1. Create one test repo
2. Define label taxonomy
3. Run one mini release with 5 to 8 issues
4. Fill events manually in one evidence file

### Week 2

1. Generate release notes from that evidence file
2. Check what was missing
3. Improve fields and rules
4. Repeat once

After two cycles, your clarity will jump dramatically.

## Next: Starter Templates

Ready-to-use templates to copy directly:

1. docs-events.yaml (empty template for your first release)
2. source-precedence.yaml (field ownership rules)
3. qa-evidence.yaml (test tracking structure)

These templates require no coding to start using.
