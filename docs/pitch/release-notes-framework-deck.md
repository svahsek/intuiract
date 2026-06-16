---
title: "Release Notes Automation Framework"
subtitle: "Standards-first documentation, built for reliable release publishing"
author: "intuiract"
date: "2026"
---

## Problem Statement

- Release notes are often manual, inconsistent, and error-prone
- Teams lose time reconciling issues, PRs, commits, and QA evidence
- Output quality drops when docs are generated without validation
- We needed a governed automation path — not ad hoc markdown generation

## Architecture Diagram
![Release Notes Architecture](docs/pitch/diagram-architecture.png)

## Pipeline

![Release Notes Pipeline](docs/pitch/diagram-pipeline.png)

## Reconciliation Logic

![Release Notes Reconciliation](docs/pitch/diagram-reconciliation.png)

## Our Solution

A standards-driven framework for release documentation.

The `release-notes` skill:

- Extracts evidence from GitHub
- Reconciles multiple conflicting sources
- Normalizes and scores each item
- Validates structured output against a schema
- Renders publication-ready Markdown only after passing all checks

## What Makes It Different

Not just "git diff → doc."

Built on:

- `standards/content-types.yaml` — central registry
- Content type manifests — stable pointer files
- Canonical schema and rendering rules
- Validation rules — schema, policy, confidence
- Human-readable skill and process documentation

Outcome: **deterministic, auditable, reusable**

## Key Architecture

Registry-first discovery:

- Registry: `standards/content-types.yaml`
- Manifest: `standards/content-types/release-notes.yaml`
- Schema: `templates/release-notes/release-notes-schema.yaml`
- Rendering: `templates/release-notes/release-notes-rendering.yaml`
- Rules: `rules/release-notes/release-notes-rule.yaml`
- Skill: `skills/release-notes/SKILL.md`

## Evidence Workflow

Five phases — no skipping, no merging:

1. **Extraction** — issues, PRs, commits, QA results, scans
2. **Reconciliation** — source precedence, conflict resolution, confidence scoring
3. **Content Generation** — map normalized evidence into release-notes schema
4. **Validation** — schema + policy + confidence gate
5. **Rendering** — Markdown output only after all checks pass

## Governance and Safety

- Source precedence removes guesswork — every field has a deterministic authority chain
- Validation prevents bad releases from being published
- Confidence thresholds enable safe automation with minimal manual review
- Path conventions protect schema and rule references from breaking across repos

## Benefits Delivered

- Faster, more reliable release note assembly
- Consistent structure across every release
- Better audit trail — provenance recorded for each release item
- Single authoritative standard for all consumer repositories
- Easier onboarding for new products and content types

## Why This Matters for Management

- Reduces release documentation risk
- Scales from one skill to many content types without rework
- Supports compliance and traceability requirements
- Lowers cost of documentation creation and review
- Positions us to offer a reusable, upgrade-safe documentation standard

## Next Steps

- Demo the `release-notes` skill with a real release cycle
- Onboard one product team to the registry/manifest pattern
- Extend the framework to additional content types
- Track quality improvements and time saved
- Adopt as the standard release documentation process across all products

## Course Correction — Closing the Execution Gap

The design is Tier 4. The execution is still Tier 2 — reconciliation logic lives in prose instructions where deterministic code should run. Three focused moves close that gap:

- **Prompt caching** — standards files are large and static; cache them once, cut token cost on every future run permanently
- **Structured outputs** — replace free-form YAML generation with schema-enforced structured output; catch violations at generation time, not after
- **`reconcile.py`** — move source-precedence application from LLM instruction to actual Python code; this is the single most important gap between design and reliable production behaviour

None of these change the architecture. All three make what is already designed actually trustworthy at scale.

## Immediate Roadmap

| Stage | Action |
|---|---|
| **Now** | Harden execution — prompt caching, structured outputs, `reconcile.py` |
| **Next** | Test reconciliation on a real repo with messy data — inconsistent labels, PRs without linked issues, missing milestones |
| **Then** | Package as GitHub Action — triggered on tag push, runs pipeline end-to-end with zero manual work |
| **Later** | MCP Server + multi-output pipeline — expose pipeline phases as callable tools; extend evidence model to generate features pages and API changelogs alongside release notes |



## Roadmap Diagram

![Roadmap](docs/pitch/diagram-roadmap.svg)
