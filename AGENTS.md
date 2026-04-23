# AGENTS.md

## Scope

These instructions apply to the whole workspace.

## What This Repository Is

This repository is a documentation-intelligence workspace for standards, templates, prompts, rules, and agent skills. It is documentation-first, not an application repo with a normal build or test loop.

- Start with [README.md](README.md) for the repo stub.
- Use [docs/INDEX.md](docs/INDEX.md) to navigate the planning and architecture documents.
- Use [docs/architecture/registry-manifest-architecture.md](docs/architecture/registry-manifest-architecture.md) for the core discovery model.
- Use [docs/course-correction/git-extraction-precedence-and-release-evidence-plan.md](docs/course-correction/git-extraction-precedence-and-release-evidence-plan.md) for the evidence precedence model.

## Working Model

- Do not assume `package.json`, `Makefile`, `pyproject.toml`, or a conventional build pipeline exists. This workspace currently has no standard app build surface.
- Prefer Markdown and YAML edits over code-first assumptions.
- Link to existing docs instead of copying their content into new instruction or skill files.
- Keep changes minimal and structural. Preserve the repository's role as a central standards source.

## Core Conventions

- Treat [standards/content-types.yaml](standards/content-types.yaml) as the registry entrypoint for supported content types.
- For a content type, resolve its manifest first, then its schema, rendering rules, quick reference, rules, and skill.
- Do not hardcode content-type file paths when the manifest can resolve them.
- YAML is the source of truth for automation; rendered Markdown is the publishable output.
- Validate structured content before proposing rendered output.

## Directory Guide

- [docs/](docs/) holds planning, architecture, and course-correction material.
- [standards/](standards/) holds registries, manifests, and classification references.
- [templates/](templates/) holds schemas, rendering guidance, examples, and quick references.
- [rules/](rules/) holds validation-oriented rule files.
- [.github/skills/](.github/skills/) holds reusable agent workflows. The strongest current example is [.github/skills/release-notes/SKILL.md](.github/skills/release-notes/SKILL.md).
- [.github/prompts/](.github/prompts/) holds reusable prompt inputs and examples.

## When Adding Or Updating Content Types

For a new content type or a major extension, update the full set of related assets together:

1. Registry entry in [standards/content-types.yaml](standards/content-types.yaml)
2. Content-type manifest in [standards/content-types/](standards/content-types/)
3. Schema and rendering assets in [templates/](templates/)
4. Validation rules in [rules/](rules/)
5. Agent workflow in [.github/skills/](.github/skills/)

Do not add only one layer unless the change is intentionally partial and documented.

## Release Notes Pattern

When working on release notes, follow [.github/skills/release-notes/SKILL.md](.github/skills/release-notes/SKILL.md) and the release-notes manifest chain. The expected sequence is gather, structure as YAML, validate, then render.

## Path Policy

- Use repository-local canonical paths in standards manifests and references: `standards/`, `templates/`, `rules/`.
- Use `.github/` paths only when the target actually exists in this repo (for example, `.github/skills/` and `.github/prompts/`).
- In conceptual docs and consumer examples, prefer `consumer-repo/.github/...` to avoid implying those files exist in this repository.
- See [docs/course-correction/path-conventions.md](docs/course-correction/path-conventions.md) for full conventions, examples, and PR checklist.

## Editing Guidance For Agents

- Prefer concise, standards-oriented language.
- Preserve existing terminology and file organization unless the task requires restructuring.
- If a file already documents a concept well, reference it from instructions instead of restating it.
- If validation or examples are missing for a standards change, add the smallest adjacent supporting file update needed.