# Overview Content Type

Schema-driven product/component overview pages, maintained by a single incremental pipeline
rather than a separate "generate fresh every time" model like release-notes.

## Why This Exists

This content type started from a hand-authored Markdown template with curly-brace placeholders
(`{PRODUCT_NAME}`, `{FEATURE_1}`, ...), now retired to `_old/` (see File Map below). It worked
for a one-off manual write-up, but two problems show up as soon as more than one product needs one:

1. There's no machine-checkable definition of what a "complete" overview page contains —
   validation is "does it look right to a human," not a deterministic rule set.
2. Every time a feature ships, keeping the feature-coverage table and "Upcoming Features" list
   current is a manual chore someone has to remember to do, and it tends to drift.

This content type formalizes the same section structure into `overview-schema.yaml` +
`overview-rendering.yaml` (the same pattern already established for `release-notes`), and adds
one pipeline capability release-notes doesn't need: **patching a long-lived file** instead of
generating a fresh one every run.

**Schema note**: the first bootstrap run against a real product surfaced gaps between the
retired template's flat feature list and how a well-developed overview page actually organizes
standards compliance — grouped into named categories (data models, protocols, formats, encoding
standards, crypto algorithms), with a `partial` status in addition to supported/unsupported, plus
an architecture "Key Components" breakdown, deployment prerequisites, and an optional SDK
Integration section for products that ship embeddable components. `overview-schema.yaml`,
`overview-rendering.yaml`, and `evidence-to-overview-mapping.yaml` were revised to support all of
these — see `examples/comprehensive-overview.yaml` for a worked example exercising every one of
them, and `examples/minimal-overview.yaml` for the simpler flat-list case that's still fully valid.

## The Pipeline (One Mode, Two Evidence Sources)

Unlike release-notes, which produces a brand-new `release-notes.yaml` every release, `overview`
maintains a single `overview.yaml` per product across its whole lifetime. Every run is the same
five-phase pipeline — extract → reconcile → patch → validate → render — but the extraction
phase draws from one of two sources depending on when it's run:

| | Bootstrap ingestion | Release signal |
|---|---|---|
| **When** | First run for a product (or a deliberate full refresh) | Every release after that |
| **Reads from** | Existing overview page + features page (code-repo ingestion deferred) | The release-notes pipeline's own output for one release |
| **Writes to** | Every schema section | Only `featureCoverage` and `upcomingFeatures` |
| **Touches prose?** | Yes — this is the only time narrative fields are populated/refreshed | Never |

This asymmetry is deliberate and enforced by `standards/overview-source-precedence.yaml`'s
`excluded_sources` field: a release-signal run structurally cannot rewrite hand-authored prose,
because the mapping file (`evidence-to-overview-mapping.yaml`) simply has no entries for those
fields under `release_signal_mapping`.

## Relationship to Release Notes

This content type is a **downstream consumer** of the release-notes pipeline, not a fork of it.
Every ongoing run reads `release-notes.yaml` / `release-evidence.yaml` — resolved from
`standards/content-types/release-notes.yaml`, never hardcoded — and asks one question per
shipped feature: "does this match something already on the overview page (as upcoming or
already-supported-but-wrong), or is it net-new?" See
`standards/overview-source-precedence.yaml`'s Scenario 1/2/3 for the concrete resolution logic.

## File Map

| File | Role |
|---|---|
| `overview-schema.yaml` | Schema authority — required/optional sections, field types, validation hooks |
| `overview-rendering.yaml` | Deterministic YAML → Markdown conversion, matching the retired template's section order |
| `evidence-to-overview-mapping.yaml` | Source-scoped mapping: what `bootstrap_ingestion` may touch vs. what `release_signal` may touch |
| `overview-quick-reference.md` | Condensed, up-to-date cheat sheet for agent prompting — the current human-readable reference (use this, not the retired template) |
| `_old/overview-page.template.md` | Retired hand-written template this schema formalized. Historical reference only — not maintained in sync with schema changes, same convention as `templates/release-notes/_old/` |
| `_old/overview-page-guideline.md` | Retired guideline for the above. Historical reference only |
| `examples/minimal-overview.yaml` | Minimal valid `overview.yaml`, flat featureCoverage only, including a `release_signal`-provenance feature row |
| `examples/comprehensive-overview.yaml` | Full example — categorized featureCoverage (with and without a Status column), `partial` status, `architecture.components`, `deployment.prerequisites`, flexible `configurations.content`, `sdkIntegration` |
| `examples/overview-evidence.sample.yaml` | Sample normalized evidence file for a `release_signal` run |

Standards-level files this content type depends on (outside this directory):

| File | Role |
|---|---|
| `standards/overview-source-precedence.yaml` | Field-level precedence and the hard `excluded_sources` boundary |
| `standards/overview-evidence-fields.yaml` | Field glossary for `overview-evidence.yaml` |
| `rules/overview-evidence-confidence-rule.yaml` | Confidence scoring for feature-status patches (auto-apply / warn / manual-review) |
| `rules/overview/overview-rule.yaml` | Deterministic validation rules (STR-*, CON-*, SEC-*, PRO-*) |

## Out of Scope (For Now)

- **Code-repo ingestion** as a third bootstrap source — deferred to a later iteration; the
  pipeline is designed to accept it later without a schema change (see `metadata.sourceRepository`).
- **Category-index pages** (e.g. "Supporting Components" module listings, see
  `templates/overview/id-lifecycle-categorization-overview-page-template.md`) — structurally
  different from a single product's overview page; not part of this content type.

## Consumer Repo Pattern

Same as release-notes: fetch `standards/content-types/overview.yaml` first, resolve every other
file from its `entrypoints`, and copy `skills/overview/consumer-overview.skill.md` into the
consumer repo, editing its Step 0a configuration block for that product.
