---
name: overview-consumer
description: >
  Generate and maintain this product's overview page. Fetches standards from the central
  .github registry, bootstrap-ingests the existing overview page and features page the first
  time a product runs this, then patches only the feature-coverage table on every release
  after that using the release-notes pipeline's own output. Configuration lives inline in this
  file (Step 0a) — no separate overview-inputs.yaml to find or maintain.
argument-hint: 'No external input file. Edit the Product Configuration block in Step 0a of this file directly; the run mode (bootstrap vs release-signal) is auto-detected from whether overview.yaml already exists.'
---

# Overview — Consumer Skill

## When To Use

- First time setting this up for a product: bootstrap `overview.yaml` from the existing
  overview page and features page.
- After every release ships: patch the feature-coverage table using that release's output
  from the release-notes pipeline.
- To force a full refresh of a product's overview page (rare — see Step 0a `force_full_refresh`).
- To validate an existing `overview.yaml` before rendering.

## Mode Selection Philosophy (read this before Step 3)

This is **one pipeline, not two skills**. The branch taken at Step 3 is auto-detected, not
something you choose per invocation:

- If `{output_dir}/overview.yaml` does not exist yet, or `force_full_refresh: true` is set in
  Step 0a → **bootstrap mode**. Reads the existing overview page + features page.
- Otherwise → **release-signal mode**. Reads one release's output from the release-notes
  pipeline and patches only `featureCoverage`/`upcomingFeatures`.

Never let release-signal mode touch prose sections, and never let bootstrap mode run silently
on every release (it would repeatedly overwrite hand-refined narrative fields) — the existence
check above is what prevents both failure modes. If you're ever unsure which mode a run took,
check `metadata.lastUpdateSource` in the resulting `overview.yaml`.

---

## Step 0 — Resolve Inputs (Inline Configuration)

No external input file. Configuration for this repo/product lives in the block below.

### 0a. Product Configuration (edit this block directly)

```yaml
# ─── PER-PRODUCT (set once, when adapting this file for a new product) ────
product_name: ""                           # e.g. "Inji Certify"
product_name_abbr: ""                      # e.g. "INJICERT"
target_users: []                           # e.g. ["issuers", "verifiers"]
core_functionality: ""                     # one sentence — used only on bootstrap runs
output_dir: "docs/overview/"
overview_yaml_filename: ""                 # blank = "{product_name_slug}-overview.yaml"

# ─── BOOTSTRAP SOURCES (used only when overview.yaml doesn't exist yet, or force_full_refresh) ─
existing_overview_page_path: ""            # e.g. "docs/product-overview.md" — blank if none exists
features_page_path: ""                     # e.g. "docs/features.md"
code_repo_path: ""                         # deferred capability — leave blank for now, not an error if empty
force_full_refresh: false                  # true = re-run bootstrap mode even if overview.yaml exists

# ─── RELEASE-SIGNAL SOURCE (used on every non-bootstrap run) ───────────────
release_notes_manifest_url: ""             # blank = resolve from standards/content-types.yaml at runtime
release_evidence_path: ""                  # blank = auto-detect the most recent {version}-release-evidence.yaml
                                            # in the release-notes pipeline's output_dir for this product
```

**Before trusting any value from this block:**
- **Duplicate-key check.** Same failure mode as release-notes' Step 0a — a repeated key
  silently keeps the last value under a permissive parser.
- Log which mode this run resolved to (bootstrap vs release-signal) and why, so a run stays
  auditable.

### 0b. Auto-detect and prompt for missing values (fallback only)

```bash
# 1. Does overview.yaml already exist for this product?
test -f "{output_dir}/{overview_yaml_filename}"
# → exists and force_full_refresh=false: release-signal mode
# → does not exist, or force_full_refresh=true: bootstrap mode

# 2. Release-signal mode: find the most recent release-evidence file
ls -t {release_notes_output_dir}/*-release-evidence.yaml 2>/dev/null | head -1
```

| Field | Auto-detectable? | If not — prompt |
|---|---|---|
| `product_name` | ❌ | "What is the product name? (e.g. Inji Certify)" |
| `target_users` | ❌ | "Who is this product for? (comma-separated list)" |
| `existing_overview_page_path` | Partial — probe common paths (`docs/overview.md`, `README.md`'s Overview section) | "Path to the existing overview page, if any? (blank if none)" |
| `features_page_path` | Partial — probe `docs/features.md`, `docs/*/features.md` | "Path to the features page, if any?" |
| `output_dir` | default: `docs/overview/` | accept default or ask |
| release/evidence source | ✅ from `standards/content-types/release-notes.yaml` + newest evidence file | rarely needed |

> If running in CI/CD mode (`CI=true`), never prompt — use auto-detected values and defaults
> only. Log any field that could not be resolved as a WARNING.

### Required fields summary

```
product_name, target_users, output_dir
```

Bootstrap mode additionally requires at least one of `existing_overview_page_path` or
`features_page_path` to be non-empty (both blank means there's nothing to ingest).

---

## Step 1 — Load Standards from Central Registry

Fetch the manifest first. Resolve all other URLs from it — never hardcode file paths.

```
Manifest URL:
https://raw.githubusercontent.com/svahsek/intuiract/main/standards/content-types/overview.yaml
```

> For stability in CI/CD, pin to a commit SHA instead of `main`.

| Entrypoint key | What to do with it |
|---|---|
| `schema` | Load — defines required sections and validation patterns |
| `rendering` | Load — defines how to convert YAML to Markdown |
| `human_rules` | Load — defines STR-*, CON-*, SEC-* validation rule IDs |
| `source_precedence` | Load — field-level precedence and `excluded_sources`, required by Step 4 |
| `overview_evidence_fields` | Load — field semantics glossary, required by Step 4 |
| `confidence_rule` | Load — confidence scoring model, required by Step 4 (release-signal mode only) |
| `mapping` | Load — evidence-to-overview field mapping, required by Step 6 |
| `skill` | Not fetched at runtime — `skills/overview/SKILL.md` documents the full pipeline this file implements a thinner version of. Read it for the conceptual model, not for file paths. |
| `quick_reference` | Load — use as fill-in template during Step 6 |

**Also resolve, only in release-signal mode**: `standards/content-types/release-notes.yaml`
(this is the `upstream_dependency` entry in the overview manifest) — resolve its `entrypoints`
the same way, don't hardcode `release-notes.yaml`'s file paths here either.

**Always pull YAML/structured standards files with `curl`/raw content, never `WebFetch`** —
`WebFetch` runs content through a summarizing model and returns paraphrase instead of exact YAML.

**Cache on first fetch.** Write all resolved entrypoints into `.github/standards/` in this repo.
On every subsequent run, check that cache before any network call.

**Fallback if manifest URL is unreachable:**
```
WARNING: Central manifest URL unreachable.
Falling back to locally cached versions if present in .github/standards/.
If no local cache exists, validation will be skipped and a WARNING will be added
to the output noting that rules were not applied.
```

---

## Step 2 — Determine Run Mode

Per Step 0b's auto-detection: does `{output_dir}/{overview_yaml_filename}` exist, and is
`force_full_refresh` set?

```
overview.yaml exists AND force_full_refresh == false  →  MODE = release_signal
overview.yaml missing OR force_full_refresh == true   →  MODE = bootstrap_ingestion
```

Log: `"Run mode: {MODE}"`. Everything from Step 3 onward branches on this value.

---

## Step 3 — Collect Evidence

### Branch A — MODE == bootstrap_ingestion

1. Read `existing_overview_page_path`, if set. Extract:
   - Opening overview narrative
   - Any existing feature-coverage table (name + ✅/❌ status)
   - Architecture, deployment, configuration, database, upgrade, documentation, contribution
     sections
2. Read `features_page_path`, if set. Extract per-feature name, description, and current status
   as documented there.
3. Cross-reference: a feature present in both sources uses `features_page` for status/description
   (per `overview-source-precedence.yaml`), but retains the existing page's row if
   `features_page` is silent on it — flag that case to `unresolved[]` rather than dropping it.
4. `code_repo_path`: skip entirely if blank. This is a deferred capability, not a missing input —
   do not warn about its absence.

**If neither `existing_overview_page_path` nor `features_page_path` resolved to a readable
file**: stop with an ERROR — there's nothing to bootstrap from. (An operator can still hand-author
`overview.yaml` directly against `overview-schema.yaml` in that case; this skill isn't required.)

### Branch B — MODE == release_signal

1. Resolve the release-notes evidence file: use `release_evidence_path` from Step 0a if set,
   else auto-detect the most recent `{version}-release-evidence.yaml` (or `release-notes.yaml`
   if evidence isn't retained) in the release-notes pipeline's output directory for this product.
2. Filter items: `change_type == "feature"` AND `shipped == true` AND
   `confidence.score >= 60` (per `rules/release-evidence-confidence-rule.yaml`, the
   release-notes-side gate — this skill does not re-derive that threshold, it trusts it).
3. For each surviving item, extract `normalized_for_release_notes.feature_name` and
   `feature_description`.

**If no release-evidence/release-notes file is found**: stop with an ERROR — release-signal
mode has nothing to patch against. Do not silently fall back to bootstrap mode; that would
overwrite prose the operator didn't ask to touch.

**Output of Step 3**: raw evidence bundle tagged with `evidence_source: {MODE}`.

---

## Step 4 — Reconcile Evidence

Use these authorities — both resolved in Step 1 from the manifest:
1. `source_precedence` for field-level source selection and `excluded_sources` enforcement
2. `overview_evidence_fields` for field semantics

Reconciliation tasks:

**Bootstrap mode:**
1. Merge existing-page and features-page signals per field, applying `source_precedence`.
2. Collect `unresolved[]` for any contradiction (see Scenario 3 in `overview-source-precedence.yaml`).
3. Tag every item's `provenance: bootstrap_ingestion`.

**Release-signal mode:**
1. Fuzzy-match each surviving feature against `upcomingFeatures.items[*].name`, then
   `featureCoverage.features[*].name`, then treat as net-new (see `overview-source-precedence.yaml`
   Scenario 1/2).
2. Score confidence using `rules/overview-evidence-confidence-rule.yaml` (loaded in Step 1).
3. Tier: >=80 auto-apply, 50-79 apply-with-warning, <50 → `unresolved[]`, hold.
4. Any item flagged `contradicts_existing_overview` → also added to `unresolved[]` regardless
   of score.
5. Tag every applied item's `provenance: release_signal`, with `source_release_version` and
   `shipped_status` copied from the release-notes evidence.

**Output of this step**: `overview-evidence.yaml` (ephemeral — not committed long-term).

---

## Step 5 — Validate Evidence (Gate)

1. Every item has a `provenance` tag.
2. Release-signal items all carry `source_release_version` and `shipped_status == true`.
3. Confidence tiers assigned; `unresolved[]` populated for held/contradicting items.

Blocking rule: if evidence validation fails structurally (missing required evidence fields per
`overview-evidence-fields.yaml`), stop here. Do not proceed to Step 6.

---

## Step 6 — Patch / Generate `overview.yaml`

Using `schema` and `mapping` (resolved in Step 1), and reconciled evidence from Step 4:

**Bootstrap mode** — populate every section using `mapping.bootstrap_mapping`:

```yaml
metadata:
  productName: "{inputs.product_name}"
  productNameAbbr: "{inputs.product_name_abbr}"
  targetUsers: {inputs.target_users}
  coreFunctionality: "{inputs.core_functionality}"
  lastUpdated: "{today}"
  lastUpdateSource: "bootstrap_ingestion"
overview:
  narrative: ""          # from existing_overview_page_path, or drafted from features_page + operator input
featureCoverage:
  features: []            # populated from reconciled evidence
architecture: {}          # from existing_overview_page_path
deployment: {}             # from existing_overview_page_path
documentation: {}           # from existing_overview_page_path
contribution: {}              # from existing_overview_page_path
```

**Release-signal mode** — load the existing `overview.yaml` and patch only two fields using
`mapping.release_signal_mapping`, leaving every other key byte-for-byte as read:

```yaml
# Only these two top-level keys are touched:
featureCoverage:
  features: [...]   # upserted rows for auto-applied/warned items
upcomingFeatures:
  items: [...]       # matching entries removed atomically with the featureCoverage write
metadata:
  lastUpdated: "{today}"
  lastUpdateSource: "release_signal"
```

**Agent writing rules:**
- Never touch `overview.narrative`, `architecture`, `deployment`, `configurations`,
  `documentation`, or `contribution` in release-signal mode, even if a change seems like an
  improvement — that's out of scope for this run; raise it to the operator as a suggestion for
  a future `force_full_refresh` bootstrap pass instead.
- Feature descriptions: only set on a net-new row, or if the existing description is empty —
  never overwrite operator-refined prose on an existing row.

---

## Step 7 — Validate Structured Content

Run all deterministic checks from `human_rules` (loaded in Step 1), in order:

**1. Structure rules (STR-*) — stop if ERROR**
STR-001 required sections present · STR-002 product name consistent · STR-003 date format

**2. Content rules (CON-*)**
CON-001 overview narrative present · CON-002 feature status enum valid ·
**CON-003 release-signal rows carry sourceRelease + lastConfirmedDate** ·
**CON-004 no feature listed in both upcomingFeatures and featureCoverage-as-supported** ·
CON-005 documentation present · CON-007 deployment modes present · CON-009 no placeholder text

**3. Security rules (SEC-*)**
SEC-001 no internal PII/team names · SEC-002 no comparative marketing language

**On ERROR:** Stop. Do not render Markdown. Require human fix before proceeding. On a
release-signal run, scope the failure to the offending item where possible: move just that
item to `unresolved[]` and proceed with the rest of the patch rather than blocking the whole run.

**On WARNING only:** Proceed to rendering; include warnings in the validation report.

---

## Step 8 — Render Markdown

Using `overview-rendering.yaml` (loaded in Step 1), convert the validated `overview.yaml` to
Markdown, applying rules in section order: HDR-*, FEA-*, TRY-*, ARC-*, PLG-*, DEP-*, SDK-*,
CFG-*, DB-*, UPG-*, UPC-*, DOC-*, CTR-*.

---

## Outputs

Save to `{inputs.output_dir}` (default: `docs/overview/`).

| File | Name pattern | Purpose |
|---|---|---|
| Overview evidence | `{product}-overview-evidence.yaml` | Ephemeral — normalized, reconciled, scored (not required to be committed) |
| YAML source | `{product}-overview.yaml` | The long-lived source of truth — commit this |
| Markdown output | `overview.md` | Publishable overview page |
| Validation report | `{product}-overview-validation-report.txt` | Warnings and rule check results |

If validation produced ERRORs, output only the evidence file, the annotated YAML, and the
validation report — do not create the `.md` file until all ERRORs are resolved.

---

## Trigger Modes

### Manual (VS Code Copilot chat)
1. First time: fill in the Step 0a block's bootstrap source paths, invoke this skill.
2. Every release after: leave the block as-is (mode auto-detects to release-signal); invoke
   this skill after the release-notes pipeline has produced its evidence for that release.
3. Review outputs, fix any ERRORs, commit `overview.yaml` and `overview.md`.

### CI/CD (GitHub Actions)

```yaml
on:
  workflow_run:
    workflows: ["release-notes"]
    types: [completed]

jobs:
  overview-patch:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - uses: actions/checkout@v4
      - name: Patch overview page
        uses: github/copilot-action@v1
        with:
          skill: .github/skills/consumer-overview.skill.md
```

Triggering off the release-notes workflow's completion, rather than the same tag-push event
independently, guarantees the release-evidence file this skill needs already exists when it runs.

---

## Fallback Handling

| Failure | Behaviour |
|---|---|
| Manifest URL unreachable | Use local `.github/standards/` cache if present; log WARNING; skip validation if no cache |
| Bootstrap mode, no source files resolvable | Stop with ERROR: nothing to ingest |
| Release-signal mode, no evidence file found | Stop with ERROR: nothing to patch against — do not fall back to bootstrap |
| Duplicate key detected in Step 0a block | Stop at Step 0 and report the duplicated key |
| Evidence validation fails structurally | Stop before patching. Output evidence file + validation report only |
| A specific feature item fails CON-003/CON-004 | Move that item to `unresolved[]`; proceed with the rest of the patch |
| Validation ERRORs present after patch | Stop before rendering. Output YAML + validation report only |
| Required input field unresolvable | Stop with ERROR listing unresolved fields |
