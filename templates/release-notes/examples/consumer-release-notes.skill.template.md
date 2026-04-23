---
name: release-notes-consumer
description: Thin consumer skill that orchestrates evidence-first release-notes generation using centralized standards and local release inputs.
argument-hint: Provide version/base_tag/release_ref or path to docs/releases/release-inputs.yaml
---

# Release Notes Consumer Skill Template

## Goal

Generate validated release notes for this repository by consuming centralized standards. This skill orchestrates only. Do not embed extraction or precedence logic here.

## Inputs

- Prefer local input file: docs/releases/release-inputs.yaml
- If missing, prompt for required fields:
  - version
  - base_tag
   - release_ref (branch or tag)
  - release_type
  - product_name

## Step 1: Resolve Central Standards

1. Load central manifest first:
   https://raw.githubusercontent.com/{org}/{standards-repo}/{ref}/standards/content-types/release-notes.yaml

2. Resolve from manifest:
   - schema
   - rendering
   - rules
   - mapping
   - references

Use ref as commit SHA in CI.

## Step 2: Extract Release Signals

Resolve target ref in this order:
1. release_ref
2. release_branch (legacy fallback)

Collect signals between base_tag..target_ref:
1. issues
2. PRs
3. commits
4. QA/build/security evidence

Output raw snapshots.

## Step 3: Reconcile to Canonical Evidence

Using centralized standards:
1. Apply field-level precedence
2. Resolve conflicts
3. Score confidence
4. Normalize for release-notes mapping

Output:
- docs/releases/{version}-release-evidence.yaml

## Step 4: Validate Canonical Evidence (Gate)

Validate canonical evidence before generating release notes:
1. release-evidence contract shape
2. required lifecycle sections present
3. confidence tier assignment complete
4. unresolved items captured

If evidence validation fails, stop and return evidence + report only.

## Step 5: Generate Structured Release Notes

1. Apply evidence-to-release-notes mapping
2. Filter by change type + confidence + shipped status
3. Merge operator inputs (headline, summary, audience, rollout timeline)

Output:
- docs/releases/{version}-release-notes.yaml

## Step 6: Validate Structured Content

Run:
1. schema validation
2. policy rule validation
3. confidence gating
4. style checks (if configured)

Output:
- docs/releases/{version}-validation-report.txt

If blocking errors exist, stop and do not render markdown.

## Step 7: Render

Render release-notes.yaml to markdown using centralized rendering rules.

Output:
- docs/releases/{version}-release-notes.md

## Required Outputs

1. release-evidence.yaml
2. release-notes.yaml
3. validation-report.txt
4. release-notes.md (only when validation passes)

## Failure Policy

1. Evidence validation fails: do not generate release-notes.yaml.
2. Structured validation fails: do not render release-notes.md.

## Consumer Guardrails

1. Keep this skill thin and orchestration-only.
2. Do not hardcode schema/rule/mapping file paths.
3. Resolve paths from manifest.
4. Do not change centralized standards in product repo.
5. Use local classification overrides only when product-specific behavior is required.
