# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A documentation-intelligence workspace: no application build, no test runner, no package manager. All work is Markdown and YAML edits. This repository is a **central standards source** that consumer repositories fetch from — treat it accordingly.

There are no `build`, `test`, or `lint` commands. Validation is conceptual (does the YAML match its schema?) not executable from the command line.

## Navigation

Start with [AGENTS.md](AGENTS.md) for workspace-wide agent instructions — it is the authoritative complement to this file.

Key entry points:
- [docs/INDEX.md](docs/INDEX.md) — planning and architecture documents in chronological order
- [standards/content-types.yaml](standards/content-types.yaml) — master registry of all supported content types
- [docs/architecture/registry-manifest-architecture.md](docs/architecture/registry-manifest-architecture.md) — how the registry/manifest chain works
- [docs/course-correction/git-extraction-precedence-and-release-evidence-plan.md](docs/course-correction/git-extraction-precedence-and-release-evidence-plan.md) — evidence precedence model

## Core Architecture

### Registry → Manifest → Asset Chain

Everything resolves from the registry. Never hardcode asset paths — always traverse the chain:

```
standards/content-types.yaml           ← "what content types exist?"
  ↓
standards/content-types/<type>.yaml    ← manifest: "where are all files for this type?"
  ↓
templates/<type>/                      ← schema, rendering rules, examples, quick reference
rules/<type>/                          ← validation rules
skills/<type>/SKILL.md                 ← agent orchestration contract
```

This means: if a file moves, update the manifest once — all consumers stay intact.

### Release Notes Pipeline (the primary active content type)

The 4-layer pipeline in [skills/release-notes/SKILL.md](skills/release-notes/SKILL.md):

1. **Extraction** — pull issues, PRs, commits, QA results from GitHub
2. **Reconciliation** — apply field-level precedence from [standards/source-precedence.yaml](standards/source-precedence.yaml); output `release-evidence.yaml`
3. **Content Generation** — map evidence to schema via [templates/release-notes/evidence-to-release-notes-mapping.yaml](templates/release-notes/evidence-to-release-notes-mapping.yaml); output `release-notes.yaml`
4. **Validation + Rendering** — validate against schema and [rules/release-notes/](rules/release-notes/); render to `release-notes.md`

**Key principle**: If validation fails, do not render. Never skip or combine pipeline layers.

### Source Precedence

[standards/source-precedence.yaml](standards/source-precedence.yaml) is the machine-readable authority for resolving field conflicts. Example: for `change_type`, the order is GitHub issue label → PR label → PR title prefix → commit prefix. The release tag boundary is always the source of truth for shipped status — issue milestones are advisory only.

### Consumer Repo Pattern

Consumer repositories fetch the manifest, resolve all entrypoints from it, and keep a local copy only of the quick reference (which they customize). They never copy the schema, rendering rules, or skill — they reference them. See [templates/release-notes/consumer-repo-starter-kit.md](templates/release-notes/consumer-repo-starter-kit.md).

## Directory Roles

| Directory | Role |
|---|---|
| `standards/` | Registries, manifests, classification references — YAML is authoritative |
| `templates/` | Schemas, rendering rules, examples, quick references per content type |
| `rules/` | Validation rule files (YAML and Markdown) |
| `skills/` | Reusable agent workflow contracts (SKILL.md files) |
| `.github/prompts/` | Reusable prompt inputs and examples |
| `docs/` | Planning, architecture, course-correction notes (not publishable output) |
| `_unsegregated/` | Staging area — files not yet assigned to their final location |

## Adding or Updating a Content Type

All five layers must be updated together (or the change documented as intentionally partial):

1. Registry entry in [standards/content-types.yaml](standards/content-types.yaml)
2. Manifest in [standards/content-types/](standards/content-types/)
3. Schema + rendering assets in [templates/](templates/)
4. Validation rules in [rules/](rules/)
5. Agent skill in [skills/](skills/)

## Path Conventions

- Use repo-local paths (`standards/`, `templates/`, `rules/`) in manifests and standards files.
- Use `.github/` paths only when the target file actually exists in this repo.
- In consumer-facing examples, use `consumer-repo/.github/...` to avoid implying files live here.
- Full conventions: [docs/course-correction/path-conventions.md](docs/course-correction/path-conventions.md)
