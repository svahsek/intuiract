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

## Canonical Inputs (Remote)

- Manifest:
https://raw.githubusercontent.com/svahsek/.github/main/standards/content-types/release-notes.yaml

- Schema:
https://raw.githubusercontent.com/svahsek/.github/main/templates/release-notes/release-notes-schema.yaml

- Rendering rules:
https://raw.githubusercontent.com/svahsek/.github/main/templates/release-notes/release-notes-rendering.yaml

- Quick reference:
https://raw.githubusercontent.com/svahsek/.github/main/templates/release-notes/agent-quick-reference.md

## Repository Inputs and Extraction

### Step 1 — Gather git data

Run both of these. They are the raw material for AI analysis.

```bash
# 1. Full commit log with metadata
git log <base_tag>..<release_branch> \
  --pretty=format:"COMMIT|%H|%s|%an|%ad" \
  --date=short

# 2. Stat-level diff (files changed, no full content yet)
git diff --stat <base_tag>..<release_branch>

# 3. Full diff (for deep analysis — can be large)
git diff <base_tag>..<release_branch>
```

> ⚠️ If the full diff exceeds ~4000 lines, use file-by-file diffing instead:
>
> ```bash
> # Get list of changed files
> git diff --name-only <base_tag>..<release_branch>
>
> # Then diff each file individually, prioritizing non-test, non-generated files
> git diff <base_tag>..<release_branch> -- <file>
> ```

---

## Step 2 — Analyze and classify changes

With the commit messages and diff in hand, classify every meaningful change into one of:

| Category | What belongs here |
|---|---|
| **✨ New Features** | New capabilities, endpoints, UI, commands, config options |
| **🐛 Bug Fixes** | Defect corrections, crash fixes, incorrect behavior resolved |
| **💥 Breaking Changes** | Removed/renamed APIs, changed signatures, config format changes, dropped support |
| **🔄 Migration Guide** | Step-by-step instructions required due to breaking changes |
| **🔧 Internal / Chores** | Refactors, dependency bumps, CI changes (include briefly or omit) |

**Classification rules:**
- Use the diff to understand *what* changed, use the commit message to understand *why*
- Merge commits and version-bump commits should be skipped
- If a commit is ambiguous, lean on the diff to classify it
- Breaking changes **must** have a corresponding migration guide entry
- Be specific: "Added `--dry-run` flag to `deploy` command" not "Added a flag"


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
