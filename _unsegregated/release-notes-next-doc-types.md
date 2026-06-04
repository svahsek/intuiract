## Approach: Recommend next document types from release notes

The right way to do this is to treat release notes as both:
- a publication artifact, and
- a decision hub for downstream documentation updates.

That means adding explicit “next content type” signals to the release evidence + generation pipeline, not trying to infer them only from prose after the fact.

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

Extend that model with explicit impact fields:
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
      evidence_source: "release_notes_section"
```

Populate these during reconciliation so the downstream recommendation is as authoritative as the release note content.

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

## 4. Extend release-notes output with an impact summary

Add a structured section to `release-notes.yaml` and `release-notes.md` such as:
- `documentationImpact`
- `recommendedDocumentationUpdates`
- `nextDocumentationTypes`

For example:
```yaml
documentationImpact:
  summary: "This release requires updates to API docs and migration guides."
  recommendations:
    - type: api-docs
      description: "New /users endpoint and expanded webhook schema."
    - type: migration-guides
      description: "Database schema change requires a migration script."
    - type: deployment-guides
      description: "New cluster rollout order for zero-downtime upgrade."
```

Then render that section into the generated release notes.

---

## 5. Keep it manifest-driven and reusable

Because your framework already uses:
- `standards/content-types.yaml`
- `standards/content-types/release-notes.yaml`
- `templates/release-notes/*`

you should extend it by adding:
- a new schema element in `templates/release-notes/release-notes-schema.yaml`
- a rendering rule in `templates/release-notes/release-notes-rendering.yaml`
- a validation rule in `rules/release-notes/release-notes-rule.yaml`

This keeps the new behavior consistent with your existing architecture.

---

## 6. Add a content-type dependency graph

For the next step beyond release-notes, maintain a small map of content-type dependencies:
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
- or inside `standards/content-types/release-notes.yaml`

This makes your release notes aware of the broader documentation landscape.

---

## 7. Use the release evidence confidence model

Your current confidence score is a powerful mechanism. Apply it to recommendations too:
- high confidence → automatically recommend
- medium confidence → recommend with review
- low confidence → flag as “needs validation before doc work”

That avoids noisy or speculative doc suggestions.

---

## 8. Make the recommendation actionable

A final recommended output structure should include:
- `type`
- `reason`
- `source`
- `confidence`
- optional `link` to the actual doc or template
- optional `owner` / `team`

That makes the release notes not just informative, but a launch point for actual documentation work.

---

## Example implementation path

1. Add `documentation_impacts` to `release-evidence-fields.yaml`
2. Add precedence rules for those fields in `standards/source-precedence.yaml`
3. Update `standards/classification-rules.yaml` with doc-impact triggers
4. Add a `documentationImpact` section to `templates/release-notes/release-notes-schema.yaml`
5. Add rendering rules for it in `templates/release-notes/release-notes-rendering.yaml`
6. Add validation rules to `rules/release-notes/release-notes-rule.yaml`
7. Document the new feature in `skills/release-notes/SKILL.md`

---

## Why this fits the framework

This approach preserves the framework’s core principles:
- evidence-first
- source precedence
- schema-driven output
- manifest-based discovery
- validation before publishing

It also turns release notes into a practical coordination artifact, not just a summary document.

If you want, I can also sketch the exact YAML schema extension and sample `documentationImpact` section for your current release-notes model.