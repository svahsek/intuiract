# Overview Content Type

Schema-driven product/component overview pages, maintained by a single incremental pipeline
rather than a separate "generate fresh every time" model like release-notes.

## Why This Exists

The existing `overview-page.template.md` in this directory is a hand-authored Markdown
template with curly-brace placeholders (`{PRODUCT_NAME}`, `{FEATURE_1}`, ...). It works for a
one-off manual write-up, but two problems show up as soon as more than one product needs one:

1. There's no machine-checkable definition of what a "complete" overview page contains —
   validation is "does it look right to a human," not a deterministic rule set.
2. Every time a feature ships, keeping the feature-coverage table and "Upcoming Features" list
   current is a manual chore someone has to remember to do, and it tends to drift.

This content type formalizes the same section structure into `overview-schema.yaml` +
`overview-rendering.yaml` (the same pattern already established for `release-notes`), and adds
one pipeline capability release-notes doesn't need: **patching a long-lived file** instead of
generating a fresh one every run.

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
| `overview-rendering.yaml` | Deterministic YAML → Markdown conversion, matching the original template's section order |
| `evidence-to-overview-mapping.yaml` | Source-scoped mapping: what `bootstrap_ingestion` may touch vs. what `release_signal` may touch |
| `overview-quick-reference.md` | Condensed cheat sheet for agent prompting |
| `overview-page.template.md` | Legacy hand-written template. Kept as a reference for manual authoring; not read by the automated pipeline |
| `overview-page-guideline.md` | Legacy guideline for the above; the section list under "REQUIRED SECTIONS" is what `overview-schema.yaml`'s `required` array formalizes |
| `examples/minimal-overview.yaml` | Minimal valid `overview.yaml`, including a `release_signal`-provenance feature row |
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
