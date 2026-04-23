# Registry-Manifest Architecture: How Standards and Content Types Fit In

## The Full Chain

A consumer repo agent starts at the registry and resolves everything from there — no hardcoded file paths.

```
standards/content-types.yaml                        ← master registry: "what content types exist?"
        ↓
standards/content-types/release-notes.yaml          ← manifest: "where are all the files for this type?"
        ↓ resolves entrypoints:
  schema:          templates/release-notes/release-notes-schema.yaml
  rendering:       templates/release-notes/release-notes-rendering.yaml
  quick_reference: templates/release-notes/release-notes-quick-reference.md
  skill:           skills/release-notes/SKILL.md
  human_rules:     rules/release-notes/release-notes-rule.yaml
```

The manifest is the **single fetch point** — a consumer repo only needs to know one URL. Everything else is resolved from it.

---

## Central vs. Consumer: What Lives Where

| File | Stays central | Consumer localizes | Reason |
|---|---|---|---|
| `standards/content-types.yaml` | ✅ | never | Registry index — consumer fetches to discover types |
| `standards/content-types/release-notes.yaml` | ✅ | never | Manifest with entrypoint URLs — fetched, not copied |
| `templates/release-notes/release-notes-schema.yaml` | ✅ | never | Structural authority — must stay canonical |
| `templates/release-notes/release-notes-rendering.yaml` | ✅ | never | Deterministic rendering — must stay canonical |
| `rules/release-notes/release-notes-rule.yaml` | ✅ | never | Validation rules — must stay canonical |
| `skills/release-notes/SKILL.md` | ✅ | never | Process contract — fetched by reference |
| `templates/release-notes/release-notes-quick-reference.md` | ✅ as base | ✅ localize | Product-specific examples and issue tracker keys |

---

## What a Consumer Repo Actually Does

```
1. Fetch content-types.yaml            → discover release-notes type exists
2. Fetch release-notes.yaml manifest   → get all entrypoint URLs
3. Agent loads SKILL.md                → knows the process (gather → validate → render)
4. Agent loads schema + rendering      → knows structure and output rules
5. Agent uses quick-reference          → local copy with product-specific examples
6. Agent validates against rule.yaml   → fetched from central
```

---

## Why Registry-First Matters

A consumer repo **never hardcodes** a path like `.github/templates/release-notes/release-notes-schema.yaml` directly. It discovers the path through the manifest.

This means:
- Files can be renamed or moved — update the manifest once, all consumers stay intact
- New entrypoints (e.g., `vale_styles`, `changelog_template`) can be added to the manifest without breaking existing consumers
- Consumer repos always get the current canonical paths without manual sync

---

## Scaling to New Content Types

As new content types are added (features page, API reference, overview), the same pattern applies:

```
standards/content-types.yaml                    ← add new entry here
standards/content-types/features.yaml           ← new manifest
standards/content-types/api-reference.yaml      ← new manifest

templates/features/
rules/features/
skills/features/
```

Each content type is self-contained and independently consumable. A consumer repo can adopt release-notes without adopting features-page.
