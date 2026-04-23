# 30-Day Course-Correction Execution Plan

## Purpose

This plan converts course-correction strategy into an executable 30-day sequence for building a robust agentic documentation pipeline.

It is grounded in:

- [architecture-assessment-and-course-correction-for-doc-automation.md](architecture-assessment-and-course-correction-for-doc-automation.md)
- [git-extraction-precedence-and-release-evidence-plan.md](git-extraction-precedence-and-release-evidence-plan.md)
- [documentation-intelligence-cli-v1-spec.md](documentation-intelligence-cli-v1-spec.md)
- [evidence-based-release-lifecycle-capture-guide.md](evidence-based-release-lifecycle-capture-guide.md)
- [product-vs-standards-strategy.md](product-vs-standards-strategy.md)

## 30-Day Outcomes

By Day 30, this repository should have:

1. A declared canonical intermediate evidence contract.
2. Machine-readable field-level source precedence rules.
3. A clear separation between extraction/reconciliation and content generation concerns.
4. A release-notes flow that runs on normalized evidence, then validates, then renders.
5. A pilot-ready workflow and quality gates for one release cycle.

## Operating Principles

1. Evidence first, narrative second.
2. Field-level precedence, not source-level opinion.
3. YAML as automation truth; Markdown as rendered output.
4. Validate before render.
5. Keep standards central and link-driven; avoid duplicated logic.

## Execution Timeline

## Week 1 (Days 1-7): Foundation Lock-In

### Milestone

Define the canonical model and standards boundaries.

### Deliverables

1. Add a canonical evidence contract draft:
	- [docs/course-correction/release-evidence-contract-v1.md](release-evidence-contract-v1.md)
2. Add machine-readable precedence rules:
	- [standards/source-precedence.yaml](../../standards/source-precedence.yaml)
3. Add field glossary to reduce ambiguity:
	- [standards/release-evidence-fields.yaml](../../standards/release-evidence-fields.yaml)
4. Add architecture snapshot for target flow:
	- [docs/architecture/agentic-doc-pipeline-target-state.md](../architecture/agentic-doc-pipeline-target-state.md)

### Acceptance Checks

1. Every required evidence field has one primary source and at least one fallback.
2. Precedence rules are machine-readable (not prose-only).
3. Extraction, reconciliation, generation, and validation boundaries are explicit.
4. No release-notes rendering rule contains source extraction logic.

## Week 2 (Days 8-14): Pipeline Refactor (Release Notes First)

### Milestone

Refactor release-notes path to consume normalized evidence.

### Deliverables

1. Add a sample normalized evidence file for one pilot release:
	- [templates/release-notes/examples/release-evidence.sample.yaml](../../templates/release-notes/examples/release-evidence.sample.yaml)
2. Add mapping specification from evidence to release-notes schema:
	- [templates/release-notes/evidence-to-release-notes-mapping.yaml](../../templates/release-notes/evidence-to-release-notes-mapping.yaml)
3. Update release-notes skill procedure to enforce sequence:
	- [skills/release-notes/SKILL.md](../../skills/release-notes/SKILL.md)

### Acceptance Checks

1. Workflow order is explicit: gather/extract -> reconcile -> validate structured content -> render.
2. Skill no longer embeds source-priority prose that conflicts with standards YAML.
3. Mapping spec covers required release-notes schema fields.
4. Validation failures block render.

## Week 3 (Days 15-21): Quality Gates and Evidence Coverage

### Milestone

Establish deterministic quality gates and lifecycle evidence coverage.

### Deliverables

1. Add coverage checklist for lifecycle stages:
	- [docs/course-correction/release-evidence-coverage-checklist.md](release-evidence-coverage-checklist.md)
2. Add confidence scoring rules:
	- [rules/release-evidence-confidence-rule.yaml](../../rules/release-evidence-confidence-rule.yaml)
3. Add integration guidance for Vale and Doc Detective signals as evidence:
	- [docs/course-correction/evidence-signals-vale-doc-detective.md](evidence-signals-vale-doc-detective.md)

### Acceptance Checks

1. Coverage checklist includes intent, change, evidence, and shipped stages.
2. Confidence scoring thresholds are documented and testable.
3. Failed critical evidence checks produce blocking outcomes.
4. Style and test signals are represented as evidence fields, not only final report text.

## Week 4 (Days 22-30): Pilot Run and Governance Closure

### Milestone

Run one pilot cycle and close governance gaps.

### Deliverables

1. Add pilot runbook:
	- [docs/course-correction/pilot-runbook-single-release.md](pilot-runbook-single-release.md)
2. Add pilot results template:
	- [templates/test-report.md](../../templates/test-report.md)
3. Add decision log for standards vs product boundary:
	- [docs/course-correction/decisions-log-standards-vs-product.md](decisions-log-standards-vs-product.md)
4. Add next-quarter roadmap from pilot evidence:
	- [docs/plan/agentic-documentation-pipeline-q-next.md](../plan/agentic-documentation-pipeline-q-next.md)

### Acceptance Checks

1. Pilot produces one complete release artifact chain (evidence -> structured content -> rendered output -> validation report).
2. At least five high-signal gaps are captured with owners and due dates.
3. Governance decisions are explicit: what stays in standards repo, what moves to product/CLI implementation.
4. A prioritized next-quarter backlog is published.

## Work Breakdown by Role

1. Documentation Architect (you): own evidence contract, precedence taxonomy, and final governance decisions.
2. Agent Workflow Designer: own skill flow updates and mapping documents.
3. Standards Maintainer: own schema/rule consistency checks across folders.
4. Pilot Operator: run one release simulation and publish findings.

## Suggested Weekly Cadence

1. Monday: lock sprint goals and file-level deliverables.
2. Wednesday: midpoint quality review (schema/rules/skill consistency).
3. Friday: publish progress note in [docs/INDEX.md](../INDEX.md) and decision updates in course-correction docs.

## Risks and Controls

1. Risk: path drift between root assets and .github-facing assets.
	Control: explicitly declare canonical location per asset class and add link-only mirrors where needed.
2. Risk: skill bloat with extraction logic reintroduced.
	Control: keep extraction and precedence logic in standards/rules files; skills orchestrate, not redefine.
3. Risk: weak input quality from source systems.
	Control: define minimum required fields and unresolved-item handling in evidence contract.
4. Risk: over-building product too early.
	Control: keep this 30-day plan standards-first and pilot-driven.

## Definition of Done (Day 30)

The course correction is complete for this phase when:

1. Canonical evidence contract and precedence rules exist and are referenced by skills.
2. Release-notes generation is evidence-driven and validation-gated.
3. One pilot run has been executed and documented.
4. Governance decisions and next-quarter plan are written and linked.

