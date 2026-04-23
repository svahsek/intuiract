---
name: release-notes
description: 'Generate and validate release notes using the centralized release-notes manifest, YAML schema, and rendering rules. Use for release documentation, changelog generation, and consumer repos that fetch standards from this public .github repository.'
argument-hint: 'Provide the release version, source inputs, and output target'
---

# Release Notes

## When To Use

- Generate release notes from issues, PRs, discussions, test evidence, or design inputs.
- Validate a release-notes YAML payload before converting it to Markdown.
- Consume shared release-notes standards from another repository using direct file URLs.

## Canonical Inputs

- Content registry: [../../standards/content-types.yaml](../../standards/content-types.yaml)
- Release-notes manifest: [../../standards/content-types/release-notes.yaml](../../standards/content-types/release-notes.yaml)
- Schema: [../../templates/release-notes/release-notes-schema.yaml](../../templates/release-notes/release-notes-schema.yaml)
- Rendering rules: [../../templates/release-notes/release-notes-rendering.yaml](../../templates/release-notes/release-notes-rendering.yaml)
- Quick reference: [../../templates/release-notes/agent-quick-reference.md](../../templates/release-notes/agent-quick-reference.md)

## Procedure

1. Read the release-notes manifest to discover the current schema, rendering rules, and examples.
2. Gather source inputs for the release: merged PRs, linked issues, notable discussions, validated test evidence, and any approved design changes.
3. Structure the release data in YAML that matches the schema.
4. Validate required fields, formatting, and conditional rules against the schema.
5. Render Markdown using the rendering rules.
6. Run style checks before publishing.

## Consumer Repo Pattern

- In a consumer repository, create a local skill that fetches the manifest first.
- Resolve the schema and rendering files from the manifest instead of hardcoding every path independently.
- Until version tags exist, prefer pinning to a commit SHA if stability matters.

## Output Expectations

- YAML is the source of truth for automation.
- Markdown is the publishable output.
- If validation fails, report the failing fields and stop before rendering.
