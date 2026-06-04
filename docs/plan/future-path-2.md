# Future Path 2: Multi-Output Documentation Pipeline

## The core principle

Don't chain from `release-notes.md`. All content types — features page, user guide, API reference — should be **siblings**, all reading from `release-evidence.yaml`. The release notes are just one rendering of the evidence, not the source of truth for others.

```
release-evidence.yaml  (canonical, normalized)
  ├── → release-notes generator     → release-notes.md      ✅ done
  ├── → features page generator     → features-page.md      ← next
  ├── → user guide delta generator  → user-guide-delta.md   ← next
  └── → api changes generator       → api-changelog.md      ← later
```

---

## What's missing (and in what order to build it)

### Step 1: Add a routing layer

Right now the pipeline stops at release notes. A step is needed after reconciliation that answers: **given this evidence batch, which content types need updating?**

Add a new file — `standards/content-type-routing.yaml` — that maps evidence field combinations to affected content types:

```yaml
routing_rules:
  features_page:
    condition: change_type == "feature" AND shipped == true
  user_guide:
    condition: (change_type in ["feature", "enhancement"]) AND ui_change == true
  api_reference:
    condition: api_change == true AND shipped == true
```

This keeps routing deterministic and standards-driven, consistent with the source precedence model.

### Step 2: Extend the evidence model

`standards/release-evidence-fields.yaml` needs two new boolean fields per item:

- `ui_change` — does this change affect a user-visible flow or screen?
- `api_change` — does this change add/modify/deprecate an API endpoint?

These should be populated during reconciliation — from PR labels, file paths touched (e.g. `src/ui/*`), or manual input. Without these flags, routing to user guide vs. features page is guesswork.

### Step 3: Build the features page as a full content type

`templates/features/` already has a template and guidelines. What's missing is the rest of the chain — same pattern as release-notes:

1. `standards/content-types/features.yaml` — manifest
2. `templates/features/features-schema.yaml` — schema
3. `templates/features/evidence-to-features-mapping.yaml` — which evidence fields map to which schema fields
4. `rules/features/` — validation rules
5. `skills/features/SKILL.md` — orchestration skill
6. Register in `standards/content-types.yaml`

The features page is the easiest to build first because the evidence signal is clean: `change_type == "feature" AND shipped == true`. No ambiguity.

### Step 4: Build user-guide-delta as a content type

This is harder because a user guide isn't generated from scratch — it's a **delta**: go find the existing section about X and update it. The skill needs to:

1. Identify affected user guide sections from evidence (via `ui_change == true` items)
2. Locate the existing section in the docs repo by topic or feature name
3. Compose only the changed portion
4. Output a diff or replacement block, not a full doc rewrite

This needs a "locate existing content" step that the features page doesn't need. Build features page first, then tackle this.

### Step 5: Dispatch skill

Once 2+ content type skills exist, add an orchestration skill — `skills/dispatch/SKILL.md` — that:

1. Loads `release-evidence.yaml`
2. Runs `standards/content-type-routing.yaml` to get the affected types list
3. Calls each content type's skill in sequence
4. Reports what was generated and what needs manual review

---

## Practical next step

Before writing any new skills: **open a real `release-evidence.yaml` from the pilot run and manually tag each item with `ui_change` and `api_change`**. That exercise will immediately surface whether the current evidence model captures enough signal to route correctly, or whether the reconciliation layer needs additional extraction logic for those fields first.

If the signal is there, the rest is replication of the release-notes pattern. If it's not, the gaps will point exactly to what needs adding in `standards/source-precedence.yaml` before building anything else.
