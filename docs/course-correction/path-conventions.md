# Path Conventions

## Purpose

Define a single, stable path policy for this repository so standards, manifests, docs, and consumer guidance stay consistent.

This document prevents path drift between:
- repository-local canonical paths (`standards/`, `templates/`, `rules/`)
- in-repo GitHub assets (`.github/skills/`, `.github/prompts/`)
- consumer examples (`consumer-repo/.github/...`)

## Scope

Applies to:
- standards manifests and registries
- schema, rendering, and rule references
- architecture and process documentation
- consumer onboarding examples

## Canonical Rule Set

1. Use repository-local canonical paths for standards assets:
   - `standards/`
   - `templates/`
   - `rules/`

2. Use `.github/` paths only when the referenced target exists in this repository:
   - `.github/skills/`
   - `.github/prompts/`

3. In conceptual docs and consumer examples, use:
   - `consumer-repo/.github/...`

4. Do not hardcode old mirror-style internal paths such as:
   - `.github/templates/...` (for this repository's templates)
   - `.github/standards/...` (for this repository's standards)
   - `.github/rules/...` (for this repository's rules)

## Manifest Authoring Rules

When writing or updating `standards/content-types/*.yaml` manifests:

1. `schema`, `rendering`, `quick_reference`, `overview`, `human_rules`, `classification_rules`, and `examples` must use repository-local canonical paths.
2. `skill` and `consumer_skill` may use `.github/skills/...` because those files physically live there in this repo.
3. Keep paths repository-relative (no absolute paths).
4. Prefer stable, real files over aspirational placeholders.

## Examples

### Correct (this repo)

```yaml
entrypoints:
  schema: "templates/release-notes/release-notes-schema.yaml"
  rendering: "templates/release-notes/release-notes-rendering.yaml"
  human_rules: "rules/release-notes/release-notes-rule.yaml"
  classification_rules: "standards/classification-rules.yaml"
  skill: ".github/skills/release-notes/SKILL.md"
```

### Correct (consumer example)

```text
consumer-repo/.github/workflows/release-notes.yml
consumer-repo/.github/skills/release-notes.skill.md
```

### Incorrect (path drift)

```yaml
schema: ".github/templates/release-notes/release-notes-schema.yaml"
classification_rules: ".github/standards/classification-rules.yaml"
human_rules: ".github/rules/release-notes/release-notes-rule.yaml"
```

## PR Checklist

Before merging docs or standards changes:

- [ ] Manifest paths resolve to existing files.
- [ ] No stale internal mirror paths (`.github/templates`, `.github/standards`, `.github/rules`) remain.
- [ ] Consumer examples use `consumer-repo/.github/...`.
- [ ] `.github/...` paths in docs point only to files that exist in this repo.
- [ ] Registry and manifest references are consistent.

## Quick Validation Command

```bash
rg -n "\.github/templates|\.github/standards|\.github/rules" docs templates standards rules
```

Expected result: no matches, except intentionally documented historical context.

## Related

- `AGENTS.md` (Path Policy section)
- `standards/content-types.yaml`
- `standards/content-types/release-notes.yaml`
- `docs/architecture/registry-manifest-architecture.md`
