## Approach: Use release evidence to drive a separate documentation-impact artifact

Release notes should remain release notes. For documentation coordination, create a separate artifact that records which document types should be updated based on the same release evidence.

The workflow is:
- extract and reconcile release evidence
- generate release notes separately
- generate a companion documentation-impact file separately
- keep the companion file as the coordination artifact for downstream docs

---

## 1. Define the target document types clearly

Start by listing the downstream docs you want to recommend:
- `api-docs`
- `deployment-guides`
- `migration-guides`
- `upgrade-notes`
- `architecture-overview`
- `release-process` / rollout notes

Make these part of your taxonomy, ideally in a central standard file such as:
- `standards/release-evidence-fields.yaml`
- or a new `standards/documentation-impact-types.yaml`

This gives the framework a stable vocabulary for recommendations.

---

## 2. Capture signals in release evidence

Your existing evidence model already has great hooks:
- `migration_guide`
- `deployment_timestamp`
- `change_type`
- `shipped`
- `normalized_for_release_notes`
- source precedence for `migration_required`

Extend the reconciled evidence model with explicit impact fields for a separate artifact:
- `documentation_impacts`
- `recommended_documentation_types`
- `documentation_references`

Example fields:
```yaml
item:
  documentation_impacts:
    - type: api-docs
      reason: "New public endpoint added"
      evidence_source: "github_pr_label"
    - type: migration-guides
      reason: "Breaking change in configuration schema"
      evidence_source: "commit_footer"
    - type: deployment-guides
      reason: "New rollout step for cluster upgrade"
      evidence_source: "pr_template_section"
```

Populate these during reconciliation so the downstream recommendation is authoritative and separated from the release note payload.

---

## 3. Map evidence to content-type recommendations

Use your classification layer to derive recommendations from the same signals you already use for release notes.

Examples:
- `BREAKING CHANGE` / `migration_required` → `migration-guides`, `upgrade-notes`
- `api_change`, `public-api`, `endpoint_added` → `api-docs`
- `deployment`, `rolling_update`, `infrastructure` → `deployment-guides`
- `database migration`, `schema change` → `migration-guides`
- `config change`, `feature toggle` → `upgrade-notes`

This belongs in:
- `standards/classification-rules.yaml`
- or a new dedicated `documentation-impact-rules.yaml`

If the repo already uses labels / PR template sections, derive it from there:
- issue/PR label `api-docs`
- issue/PR label `deployment`
- PR body section `## Migration`
- commit footer `BREAKING CHANGE`

---

## 4. Create a separate documentation-impact file

Generate a companion artifact such as `release-documentation-impact.yaml` or `documentation-impact.yaml`.

That artifact should include:
- overall summary
- recommended documentation types
- reason for each recommendation
- source of each recommendation
- confidence level
- optional links or doc owners

Example:
```yaml
documentationImpact:
  summary: "Next documentation work should focus on API docs, migration guides, and deployment guides."
  recommendations:
    - type: api-docs
      description: "New /users endpoint and expanded webhook schema."
      source: "github_pr_label"
      confidence: 90
    - type: migration-guides
      description: "Database schema change requires a migration script."
      source: "commit_footer"
      confidence: 95
    - type: deployment-guides
      description: "New cluster rollout order for zero-downtime upgrade."
      source: "release_evidence"
      confidence: 80
```

Keep this artifact separate from `release-notes.yaml` and `release-notes.md`.

---

## 5. Keep it manifest-driven and reusable

Because your framework already uses:
- `standards/content-types.yaml`
- `standards/content-types/release-notes.yaml`
- `templates/release-notes/*`

you can add a companion content type or a companion manifest entry for documentation impact.

Options:
- add a new content type `documentation-impact`
- or extend `release-notes` to reference a companion `documentation-impact` artifact

For a separate artifact, define:
- `templates/documentation-impact/documentation-impact-schema.yaml`
- `templates/documentation-impact/documentation-impact-rendering.yaml`
- `rules/documentation-impact/documentation-impact-rule.yaml`
- `.github/skills/documentation-impact/SKILL.md`

This keeps the new behavior aligned with your architecture.

---

## 6. Add a content-type dependency graph

Maintain a small map of documentation dependencies separately from release notes:
```yaml
release-notes:
  related_documentation:
    api-docs: "New API behavior introduced"
    migration-guides: "Breaking schema or upgrade behavior"
    deployment-guides: "New deployment or rollout steps"
    overview-pages: "Major platform changes that affect architecture"
```

Store that in a central place:
- `standards/content-type-dependencies.yaml`
- or a new `standards/documentation-impact-dependencies.yaml`

The companion artifact can reference those dependencies without changing the release note itself.

---

## 7. Use the release evidence confidence model

Your current confidence score is a powerful mechanism. Apply it to the companion recommendations too:
- high confidence → automatically recommend
- medium confidence → recommend with review
- low confidence → flag as “needs validation before doc work”

That avoids noisy or speculative doc suggestions.

---

## 8. Make the recommendation actionable

A companion documentation-impact artifact should include:
- `type`
- `reason`
- `source`
- `confidence`
- optional `link` to the actual doc or template
- optional `owner` / `team`

That makes the output a launch point for actual documentation work rather than just a note inside release notes.

---

## Example implementation path

1. Add `documentation_impacts` to `release-evidence-fields.yaml`
2. Add precedence rules for those fields in `standards/source-precedence.yaml`
3. Update `standards/classification-rules.yaml` with doc-impact triggers
4. Create a separate `documentation-impact` schema and artifact
5. Add rendering rules for the companion artifact in `templates/documentation-impact/*`
6. Add validation rules to `rules/documentation-impact/*`
7. Document the new feature in a companion `.github/skills/documentation-impact/SKILL.md`

---

## Why this fits the framework

This approach preserves the framework’s core principles:
- evidence-first
- source precedence
- schema-driven output
- manifest-based discovery
- validation before publishing

It also keeps release notes focused while making downstream documentation coordination explicit and actionable.

If you want, I can also sketch the exact YAML schema extension and sample `documentation-impact` artifact for this model.