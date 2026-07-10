---
name: release-notes-consumer
description: >
  Generate production-ready release notes for this repository. Fetches standards
  from the central .github registry, collects git/GitHub data through a
  confidence-gated extraction funnel, classifies changes, validates against
  rules, and renders final YAML + Markdown outputs. Configuration lives inline
  in this file (Step 0a) — no separate release-inputs.yaml to find or maintain.
  The extraction funnel is self-sufficient on git + GitHub evidence alone; a
  PO-written release notes draft and a `.github/release.yml` taxonomy are
  optional accelerants used automatically when present and skipped silently
  when not — neither is a dependency.
argument-hint: 'No external input file. Edit the Release Configuration block in Step 0a of this file directly; override per-run values via CI env vars; anything left blank is auto-detected or prompted for.'
---

# Release Notes — Consumer Skill

## When To Use

- At release time: generate release notes from git history, PRs, and issues.
- In CI/CD: trigger automatically on tag push or release branch creation.
- Manually: run in VS Code Copilot chat to draft release notes before a release.
- To validate an existing release YAML payload before rendering.

## Extraction Philosophy (read this before Step 3)

Naively mining every commit/PR, classifying it, and only *then* discarding most of it at
publication filtering is expensive and slow on repos with 100+ commits. This skill inverts that:
cheap, batched, structural signals go first; expensive per-item reading (PR bodies, diffs)
happens only for the items that survive filtering.

Two extra signals are folded into the funnel — a PO-authored release notes draft, and a
`.github/release.yml` taxonomy — but both are **conditional accelerants, not dependencies**:

- Check for them once, cheaply, at the top of Step 3.
- If present: use them to skip work that would otherwise require per-item mining.
- If absent: say so once in the log, and continue on the git + GitHub evidence path exactly as
  if they didn't exist. **Their absence is normal, not degraded, and must never lower confidence
  scores, trigger warnings, or block anything.**

This matters because both will exist for some releases and not others — PO drafts get written
"sometimes and sometimes not at all," and `.github/release.yml` requires label discipline this
org doesn't have yet. The core git/GitHub extraction path is the one thing that must always
work; the two extras are bonus speed when the org's process happens to have produced them.

---

## Step 0 — Resolve Inputs (Inline Configuration)

No external input file. Configuration for this repo/release lives in the block below, in this
file. Wherever this document says `{inputs.field_name}` later on (Steps 3, 6), it means "the
value this step resolved," regardless of whether it came from an env var, the block, or a
prompt — the three sources are ranked, not alternated.

### 0a. Release Configuration (edit this block directly)

Resolution order per field, highest wins:

1. **CI env vars** — checked only when `CI=true`. `RELEASE_VERSION`, `RELEASE_BASE_TAG`,
   `RELEASE_REF`, `RELEASE_TYPE`, `RELEASE_DATE` override the block below for a single
   automated run without editing this file. See the CI/CD trigger mode for the full mapping.
2. **This block**, for any field with a non-empty value.
3. **Step 0b** (auto-detect from git, or prompt), for anything still unresolved.

```yaml
# ─── PER-RELEASE (edit before every run) ──────────────────────────────────
version: ""                                # e.g. "0.14.0"
base_tag: ""                               # e.g. "v0.13.1" — previous release tag
release_ref: ""                            # branch or tag, e.g. "release/0.14.0"
release_branch: ""                         # legacy fallback if release_ref left blank
release_type: ""                           # Major | Minor | Patch | Hotfix | LTS
release_date: ""                           # YYYY-MM-DD — blank = today

# ─── PER-REPO (set once, when adapting this file for a new consumer repo) ─
product_name: ""                           # e.g. "Inji Certify" — used in header + SEC-003
product_name_abbr: ""                      # e.g. "INJICERT" — max 10 chars
jira_project_key: ""                       # used in CON-002 issue-ref pattern; blank if unused
output_dir: "docs/releases/"
target_audience:
  - "Developers"
  - "Implementers"

# ─── EVIDENCE SOURCE (per-release scope selectors — feeds Step 3a) ────────
evidence_source:
  milestone: ""                            # GitHub milestone title, if used
  include_issue_labels: []                 # e.g. ["type:feature", "type:bug"]
  include_pr_labels: []                    # e.g. ["ready-for-release"]
  issue_query: ""                          # e.g. "milestone:v0.14.0 is:closed"
  pr_query: ""                             # e.g. "is:pr is:merged base:main merged:>=2026-04-01"
  include_discussions: false
  include_projects: false
  include_qa_evidence: true
  include_security_scans: true

# ─── DOCUMENTATION LINKS — CON-008 required, CON-009 required for Major/Minor ─
documentation:
  feature_docs: ""
  api_docs: ""
  test_report: ""
  collab_guide: ""

# ─── SUPPORT CONTEXT (recommended — improves prose quality and validation) ─
release_highlights: ""                     # free text, e.g. "Focus on OpenID4VCI compliance"
known_issues_tracker: ""                   # CON-010: URL to full known-issues filter
rollout_timeline: ""                       # e.g. "Staggered rollout over 2 weeks"
compatible_modules: []                     # CON-006: [{module: "", version: ""}]
repositories_released: []                  # CON-007: [{repo: "", tag: ""}]

# ─── OPTIONAL ACCELERANT (see Extraction Philosophy — probe-and-skip) ─────
po_notes_path: ""                          # blank = default docs/releases/po-notes/{version}.md
```

**Before trusting any value from this block:**
- **Duplicate-key check.** Same failure mode as any YAML, file or not — a repeated key silently
  keeps the last value under a permissive parser. This exact bug (an active `release_ref:` left
  inside what was meant to be a fully-commented section, silently overriding the real value)
  caused a real incident. Parse-validate for duplicates before trusting the block.
- **Pre-flight resolve `release_ref`**: `git rev-parse --verify <release_ref>` (or the tag/branch
  equivalent). If it fails, stop and ask now — don't discover a bad ref 7 steps deep while
  running `git log`.
- Log which fields resolved from an env var, from this block, or from Step 0b, so a run stays
  auditable without needing a separate file as a paper trail.

### 0b. Auto-detect and prompt for missing values (fallback only)

```bash
# 1. Try to detect version from current branch name
git branch --show-current
# Pattern: release/X.Y.Z or release-X.Y.Z → extract X.Y.Z

# 2. Try to detect base tag (most recent tag reachable from current branch)
git describe --tags --abbrev=0

# 3. Release date = today
date +"%Y-%m-%d"

# 4. Release type: infer from version diff
# If MAJOR changed → Major; if MINOR changed → Minor; if PATCH only → Patch
```

| Field | Auto-detectable? | If not — prompt |
|---|---|---|
| `version` | ✅ from branch name | "What is the release version? (e.g. 0.14.0)" |
| `base_tag` | ✅ from `git describe` | "What is the base tag to compare from? (e.g. v0.13.1)" |
| `release_ref` | ✅ from `git branch` or tag ref | "What is the release ref? (branch or tag, e.g. release/0.14.0 or v0.14.0)" |
| `release_type` | ✅ infer from semver diff | "Is this a Major, Minor, Patch, or Hotfix?" |
| `release_date` | ✅ today's date | rarely needed |
| `product_name` | ❌ | "What is the product name? (e.g. Inji Certify)" |
| `product_name_abbr` | ❌ | "What is the short abbreviation? (e.g. INJICERT)" |
| `jira_project_key` | ❌ | "What is the JIRA project key? (e.g. INJICERT) — or leave blank if not using JIRA" |
| `output_dir` | default: `docs/releases/` | accept default or ask |
| `target_audience` | default: `[Developers, Implementers]` | accept default or ask |

> If running in CI/CD mode (detected via `CI=true` env var), never prompt — use auto-detected
> values and defaults only. Log any field that could not be resolved as a WARNING.

### Required fields summary

```
version, base_tag, release_ref, release_type, product_name
```

If unresolved after auto-detection and prompting:
```
ERROR: Cannot proceed. Unresolved required fields: [field_name]
Fix: fill the Release Configuration block in Step 0a, or provide values when prompted.
```

---

## Step 1 — Load Standards from Central Registry

Fetch the manifest first. Resolve all other URLs from it — never hardcode file paths.

```
Manifest URL:
https://raw.githubusercontent.com/svahsek/intuiract/main/standards/content-types/release-notes.yaml
```

> For stability in CI/CD, pin to a commit SHA instead of `main`.

| Entrypoint key | What to do with it |
|---|---|
| `schema` | Load — defines required fields and validation patterns |
| `rendering` | Load — defines how to convert YAML to Markdown |
| `human_rules` | Load — defines STR-*, CON-*, SEC-* validation rule IDs |
| `source_precedence` | Load — field-level source precedence, required by Step 4 |
| `release_evidence_fields` | Load — field semantics glossary, required by Step 4 |
| `confidence_rule` | Load — confidence scoring model, required by Step 4 |
| `mapping` | Load — evidence-to-release-notes field mapping, required by Step 6 |
| `release_evidence_contract` | Load — canonical evidence shape, required by Step 5's gate |
| `skill` | Not fetched at runtime — `skills/release-notes/SKILL.md` documents the full pipeline this file implements a thinner version of. Every path it names is also an entrypoint above, so there's no path-resolution reason to read it; read it only for the conceptual model behind these steps. |
| `quick_reference` | Load — use as fill-in template during Step 6 |

**All nine entrypoints above are resolved from this one manifest fetch — none of Steps 2-7 should
ever reference a standards file by bare name.** If a later step names a file that isn't in this
table, that's a manifest gap, not something to guess a path for — stop and flag it rather than
inventing a URL.

**Always pull YAML/structured standards files with `curl`/raw content, never `WebFetch`** —
`WebFetch` runs content through a summarizing model and returns paraphrase instead of exact YAML.
Reserve `WebFetch` for genuinely prose content where a summary is acceptable.

**Cache on first fetch.** Write all resolved entrypoints — `schema.yaml`, `rendering.yaml`,
`human-rules.yaml`, `classification-rules.yaml`, `source-precedence.yaml`,
`release-evidence-fields.yaml`, `release-evidence-confidence-rule.yaml`,
`evidence-to-release-notes-mapping.yaml` — into `.github/standards/` in this repo. On every
subsequent run, check that cache before any network call — don't re-fetch from network every time.

**Fallback if manifest URL is unreachable:**
```
WARNING: Central manifest URL unreachable.
Falling back to locally cached versions if present in .github/standards/.
If no local cache exists, validation will be skipped and a WARNING will be added
to the output noting that rules were not applied.
```

---

## Step 2 — Load Classification Rules

Load in this order and deep-merge:

```
1. Central baseline (from manifest or direct URL):
   https://raw.githubusercontent.com/svahsek/intuiract/main/standards/classification-rules.yaml

2. Consumer overrides (local, product-specific):
   .github/standards/classification-overrides.yaml
   → If file does not exist, use central baseline only
```

Merge behaviour:
- `additional_prefixes` in overrides → appended to central prefixes list
- `additional_keywords` in overrides → appended to central keywords list
- `omit_from_output` in overrides → overrides central value
- `jira_project_key` in overrides → used in issue reference pattern for CON-002

This is the fallback classifier used whenever neither `.github/release.yml` categorization nor
issue/PR labels resolve a change's type (see Step 3c). It must stand on its own — treat it as
the primary classifier for repos with zero label discipline, not a last resort.

Log: `"Classification rules loaded. Consumer overrides: [found/not found]"`

---

## Step 3 — Collect Evidence (extraction funnel)

Four passes. Each pass narrows what the next one has to touch. Passes 3a and 3d always run.
Passes 3b and 3c are conditional probes — cheap to check, free to skip.

### 3a. Always — batch-fetch structural evidence in one pass (the resilient core)

Resolve target ref: `release_ref`, else `release_branch` (legacy fallback). All commands run
against `base_tag..target_ref`.

```bash
# 1. Commit log — cheap, local, always available
git log {base_tag}..{target_ref} \
  --pretty=format:"COMMIT|%H|%s|%an|%ad" --date=short --no-merges

# 2. Extract PR numbers referenced in commit subjects (GitHub squash-merge appends "(#1234)";
#    merge commits reference PR number directly) — this scopes exactly which PRs matter,
#    instead of paginating every merged PR in the repo's history.
git log {base_tag}..{target_ref} --no-merges --pretty=format:"%s %b" | grep -oE "#[0-9]+"

# 3. Batch-fetch those PR numbers via a single GraphQL query using aliases — dozens of PRs
#    (title, body, labels, linked issues) in one request instead of N sequential REST calls.
#    This is the single highest-leverage fix for run time on any repo with 50+ commits.
gh api graphql -f query='
  query {
    repository(owner:"{owner}", name:"{repo}") {
      pr1234: pullRequest(number:1234) {
        title body mergedAt
        labels(first:10) { nodes { name } }
        closingIssuesReferences(first:5) { nodes { number title } }
      }
      # ...one aliased field per PR number, chunked ~50 per request
    }
  }'
```

If `evidence_source` in the Step 0a block has any of `milestone`, `include_issue_labels`,
`include_pr_labels`, `issue_query`, `pr_query` set, apply them to scope/filter this pull — this
remains the primary way an operator narrows scope, just read from this file instead of a
separate one.

> ⚠️ Only fall back to a full `git diff` when a specific item's classification is still
> ambiguous after 3a-3d (see 3d) — never run a full-range diff up front. If a full diff is
> needed for one file, diff that file only.

**Mechanical rules — non-negotiable regardless of repo size or which conditional sources exist:**
- Never loop sequential `gh api` calls over more than ~20 PR numbers. Batch via GraphQL aliases.
- If parallelizing fetches, write each result to its own file (`pr-$n.json`) and merge after.
  Never have concurrent processes append to one shared file — this has silently corrupted and
  dropped records before.
- If a background fetch produces zero output after ~15s, inspect `ps`/partial file state before
  launching a second "just in case" attempt. Don't retry blindly with a new approach while the
  first attempt's failure is undiagnosed.

### 3b. Conditional — GitHub-native categorization, if `.github/release.yml` exists

Probe once, cheaply:

```bash
gh api repos/{owner}/{repo}/contents/.github/release.yml --jq '.name' 2>/dev/null
```

**If found:**
```bash
gh api repos/{owner}/{repo}/releases/generate-notes \
  -f tag_name="{version}" -f previous_tag_name="{base_tag}" -f target_commitish="{release_ref}"
```
One call returns a PR list already grouped by the categories defined in `release.yml` (which
themselves map PR labels to sections). Use this grouping as a **pre-classification hint** for
`change_type` — still subject to the same confidence/precedence rules in Step 4, not blindly
trusted. Skips the keyword-matching pass in Step 2 for anything it already grouped.

**If not found, or found but the repo has no label discipline yet (every PR lands in one
ungrouped "Other Changes" bucket):** log once —
`"No .github/release.yml categorization available/effective. Using classification-rules.yaml
baseline (Step 2) for all items."` — and continue. This is not a warning surfaced to the
validation report; it's expected steady state for repos that haven't adopted PR labels yet.
**Never let this probe's absence slow down or degrade the rest of the run.**

### 3c. Conditional — PO-authored release notes draft, if present

Probe once, cheaply:

```bash
test -f "{inputs.po_notes_path:-docs/releases/po-notes/{version}.md}"
```

**If found:** read it. Treat it as a strong source of information — the PO produced it with
context an agent doesn't have — for narrative-only fields (summary.headline,
summary.paragraphs, keyBenefits) and as a hint list of feature names to look for corroborating
evidence on. It is not an authority: PO drafts get scrutinized the same way any other source
does, because POs make mistakes too. Score every PO-sourced claim through the confidence_rule
(Step 1) same as git/GitHub-sourced claims; only treat it as usable without further corroboration
at confidence ≥80 — below that band it needs the same corroboration-or-exclude treatment as any
other sub-84 item in Step 3d. Every claim still requires a resolvable PR/issue reference before
publication (CON-002) — a PO draft never substitutes for evidence, it substitutes for the agent's
own guesswork when drafting prose.

**If not found:** log once — `"No PO release notes draft found at {path}. Drafting narrative
fields from evidence."` — and continue: the agent writes
`summary.headline`/`summary.paragraphs`/`keyBenefits` from reconciled evidence in Step 6. **This
is the default path, not a fallback path** — it must produce full-quality output on its own,
because it will be the common case until PO drafts become routine.

### 3d. Confidence-gated deep mining (the actual cost control)

After 3a-3c, every item has a provisional `change_type` (from label/`release.yml`/prefix/keyword,
in that precedence order) and a provisional description (from PR body/CodeRabbit summary/commit
subject/PO draft, whichever resolved).

- **Skip deep reads entirely** for anything provisionally classified `internal`
  (chore/ci/test/docs/build/infra-without-metric/reverts/version-bumps per
  `classification-rules.yaml`). These will never publish — don't spend tokens reading them.
- For everything else, only go deeper (full PR body if not already fetched, targeted file diff)
  when:
  - the provisional description is too thin to write a compliant entry (CON-003: 10+ words), or
  - confidence is in the 60-84 band and needs corroboration before publication, or
  - the item touches a migration-relevant path (schema files, API specs, DB migration scripts —
    these are worth a targeted diff even when the commit message doesn't flag them explicitly;
    this is the one category where "nice to have" reading has historically caught things no
    commit message mentioned, e.g. a mandatory DB migration script).
- Never run a full-range `git diff` as a first move. File-by-file, and only for surviving items.

**Output of Step 3**: raw evidence bundle — commits, PR metadata, any `release.yml` grouping,
any PO draft content, each item tagged with its source(s) and a provisional classification.

---

## Step 4 — Reconcile Evidence (Canonical Model)

Transform raw extracted signals into canonical release evidence.

Use these authorities — all four resolved in Step 1 from the manifest, not fetched fresh here:
1. `source_precedence` for field-level source selection
2. `release_evidence_fields` for field semantics
3. `classification_rules` (loaded in Step 2) for change typing (baseline; `release.yml` grouping
   from 3b, when present, is treated as a computed shortcut for the same `github_pr_labels`
   fallback tier — not a new precedence tier)
4. `confidence_rule` for confidence scoring

Reconciliation tasks:
1. Merge issue, PR, commit, QA, and build/security signals by linkage keys.
2. Apply field precedence deterministically (primary then fallback sources).
3. Resolve conflicts and collect unresolved items.
4. Classify change type from labels/`release.yml` grouping/prefix/keywords, in that order.
5. Score confidence and assign publication tier.
6. Populate `normalized_for_release_notes` mapping fields, including narrative fields sourced
   from the PO draft when present (tag provenance as `po_release_notes`) or generated from
   evidence when absent (tag provenance as `generated_from_evidence`) — **absence of a PO draft
   must not lower a narrative field's confidence score; it's a normal, expected source state,
   not a gap.**

**Output of this step**:
- `docs/releases/{version}-release-evidence.yaml`
- `unresolved[]` entries for ambiguous/conflicting items

---

## Step 5 — Validate Canonical Evidence (Gate)

Validate `release-evidence.yaml` before generating any release-notes content, against
`release_evidence_contract` (resolved in Step 1).

1. Contract shape matches release evidence contract.
2. Required lifecycle fields exist (intent/change/evidence/shipped).
3. Precedence/provenance fields are present where required.
4. Confidence tiers are assigned and unresolved items are separated.

Blocking rule: if evidence validation fails, stop here. Do not generate `release-notes.yaml`.

## Step 6 — Generate Structured Release Notes YAML

Using the `schema` and `mapping` (evidence-to-release-notes field mapping) resolved in Step 1,
and reconciled evidence from Step 4, fill the release YAML.

```yaml
metadata:
  version: "{inputs.version}"
  releaseDate: "{inputs.release_date}"
  releaseType: "{inputs.release_type}"
  productName: "{inputs.product_name}"
  productNameAbbr: "{inputs.product_name_abbr}"
  targetAudience: {inputs.target_audience}

summary:
  headline: ""          # PO draft, verbatim or lightly edited, if 3c found one — else agent writes
  paragraphs:
    - text: ""           # Purpose — PO draft if present, else agent writes from evidence
      sectionLabel: "Purpose"
    - text: ""           # ThemesAndFocus
      sectionLabel: "ThemesAndFocus"
  keyBenefits: []        # PO draft candidates, verified against evidence, else agent-derived

content:
  newFeatures:           # Populated from release-evidence.yaml
  enhancedFeatures:
  bugFixes:
  securityUpdates:
  technicalImprovements:
  deprecations:
  knownIssues:
  compatibleModules: []  # CON-006: must be filled — agent prompts if not in inputs
  repositories:
    released: []          # CON-007: must be filled — agent extracts from git tags
  supportResources:
    documentation:
      feature_docs: ""    # CON-008: required
      api_docs: ""        # CON-008: required
      test_report: ""     # CON-009: required for Major/Minor
```

**Agent writing rules:**
- `summary.headline`: active voice, benefit-driven, no superlatives (PRO-001) — applies equally
  whether drafted by the agent or lightly edited from a PO draft; PO drafts are not exempt from
  style/marketing rules, they're just a faster starting point.
- Feature descriptions: lead with user value, not implementation detail.
- Bug fix descriptions: "Fixed [what] which caused [impact]" pattern.
- All JIRA refs: format as `{jira_project_key}-{number}` using key from inputs.

---

## Step 7 — Validate Structured Content

Run all deterministic checks from the human_rules file loaded in Step 1, in order:

**1. Structure rules (STR-*) — stop if ERROR**
STR-001 required sections present · STR-002 version pattern · STR-003 date format

**2. Content rules (CON-*)**
CON-001 overview present · CON-002 issue refs valid · CON-003 description length ·
CON-004 no empty sections · CON-005 migration notes if required · CON-006 compatibleModules ·
CON-007 repositories.released · CON-008 documentation links · CON-009 test_report for Major/Minor ·
CON-010 known issues tracker · CON-011 no newFeatures on Patch/Hotfix · CON-012 userStories for
Major/Minor · CON-013 no placeholder text

**3. Security rules (SEC-*)**
SEC-001 no internal PII/emails/team names · SEC-002 no comparative marketing language ·
SEC-003 release name contains productName + version

**On ERROR:** Stop. Do not render Markdown. Require human fix before proceeding.
**On WARNING only:** Proceed to rendering; include warnings in the validation report.

---

## Step 8 — Render Markdown

Using the rendering rules loaded in Step 1, convert validated YAML to Markdown, applying rules
in section order: HDR-*, ROL-*, SUM-*, FEA-*, ENH-*, BUG-*, RSEC-*, API-*, IMP-*, ISS-*.

---

## Outputs

Save to `{inputs.output_dir}` (default: `docs/releases/`).

| File | Name pattern | Purpose |
|---|---|---|
| Canonical evidence | `{version}-release-evidence.yaml` | Normalized, reconciled, scored |
| YAML source | `{version}-release-notes.yaml` | Source of truth for automation |
| Markdown output | `{version}-release-notes.md` | Publishable release notes |
| Validation report | `{version}-validation-report.txt` | Warnings and rule check results |

If validation produced ERRORs, output only the evidence file, the annotated YAML, and the
validation report — do not create the `.md` file until all ERRORs are resolved.

---

## Trigger Modes

### Manual (VS Code Copilot chat)
1. Edit the Release Configuration block in Step 0a of this file directly with release details
   (PO notes and `.github/release.yml` are picked up automatically if present — nothing to
   configure for either).
2. Open Copilot chat and invoke this skill.
3. Review outputs in `docs/releases/`.
4. Fix any ERRORs flagged in validation report.
5. Commit the Markdown/YAML outputs — and, since per-release fields (`version`, `base_tag`,
   `release_ref`, `release_date`) were edited in this file for the run, commit that edit too, the
   same way you'd have committed an updated `release-inputs.yaml` before.

### CI/CD (GitHub Actions)

No file-writing step needed — export the per-release fields as env vars, which Step 0a checks
first (highest precedence, above this file's block):

```yaml
on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release-notes:
    runs-on: ubuntu-latest
    env:
      CI: "true"
      RELEASE_VERSION: "${{ github.ref_name }}"
      RELEASE_REF: "${{ github.ref_name }}"
      # RELEASE_BASE_TAG resolved below since it needs full git history checked out first
      RELEASE_TYPE: "Minor"
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0          # required for full git history

      - name: Resolve base tag
        run: echo "RELEASE_BASE_TAG=$(git describe --tags --abbrev=0 HEAD^)" >> "$GITHUB_ENV"

      - name: Generate release notes
        uses: github/copilot-action@v1
        with:
          skill: .github/skills/consumer-release-notes.skill.md
```

Per-repo fields (`product_name`, `product_name_abbr`, `jira_project_key`, `output_dir`,
`documentation`, etc.) don't need env vars at all in CI — they're already set in the Step 0a
block as part of adapting this file for the repo, and don't change per run.

---

## Fallback Handling

| Failure | Behaviour |
|---|---|
| Manifest URL unreachable | Use local `.github/standards/` cache if present; log WARNING; skip validation if no cache |
| Classification rules URL unreachable | Use local `classification-overrides.yaml` only; log WARNING |
| `git log` returns zero commits | Stop with ERROR: "No commits found between {base_tag} and {target_ref}. Verify refs exist." |
| `.github/release.yml` absent, or present but no labeled PRs in range | Log once, no WARNING; fall through to `classification-rules.yaml` baseline (Step 2) — this is expected steady state, not a degraded run |
| PO notes draft absent at conventional/configured path | Log once, no WARNING; agent drafts narrative fields from evidence (Step 6) exactly as if this source never existed as a concept |
| `release_ref` does not resolve via `git rev-parse --verify` | Stop immediately at Step 0, before any fetch — ask for a corrected ref rather than discovering the problem mid-extraction |
| Duplicate key detected in the Step 0a Release Configuration block | Stop at Step 0 and report the duplicated key — do not silently take the last value |
| Evidence validation fails | Stop before generation. Output `release-evidence.yaml` + validation report only. |
| Required input field unresolvable | Stop with ERROR listing unresolved fields |
| All commits classify as internal | Warn: "All changes classified as internal. Verify base_tag is correct." Produce minimal release notes with only metadata and summary. |
| Validation ERRORs present | Stop before rendering. Output YAML + validation report only. |
