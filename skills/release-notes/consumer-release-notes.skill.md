---
name: release-notes-consumer
description: >
  Generate production-ready release notes for this repository. Fetches standards
  from the central .github registry, collects git data, classifies changes,
  validates against rules, and renders final YAML + Markdown outputs.
  Supports hybrid input: reads release-inputs.yaml if present, otherwise
  auto-detects from git and prompts only for what cannot be resolved.
argument-hint: 'Optionally provide path to release-inputs.yaml. If absent, skill auto-detects from git or prompts for missing values.'
---

# Release Notes — Consumer Skill

## When To Use

- At release time: generate release notes from git history, PRs, and issues.
- In CI/CD: trigger automatically on tag push or release branch creation.
- Manually: run in VS Code Copilot chat to draft release notes before a release.
- To validate an existing release YAML payload before rendering.

---

## Step 0 — Resolve Inputs (Hybrid)

### 0a. Check for input file

Look for `docs/releases/release-inputs.yaml` in the repository root.

**If the file exists:**
- Read all fields from it
- Validate that required fields are present (see required fields below)
- If `evidence_source` is present, use it to drive issue/PR/milestone discovery
- Log which fields were loaded: `"Input file found: docs/releases/release-inputs.yaml"`
- Skip Step 0b entirely

**If the file does not exist:**
- Log: `"No release-inputs.yaml found. Falling back to auto-detection and prompts."`
- Proceed to Step 0b

### 0b. Auto-detect and prompt for missing values (fallback only)

Attempt to auto-detect in this order:

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

After auto-detection, resolve remaining fields:

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

> If running in CI/CD mode (detected via `CI=true` env var), never prompt — use auto-detected values and defaults only. Log any field that could not be resolved as a WARNING in the output.

### Required fields summary

```
version, base_tag, release_ref, release_type, product_name
```

If any required field cannot be resolved after auto-detection and prompting, stop and report:
```
ERROR: Cannot proceed. Unresolved required fields: [field_name]
Fix: create docs/releases/release-inputs.yaml or provide values when prompted.
```

---

## Step 1 — Load Standards from Central Registry

Fetch the manifest first. Resolve all other URLs from it — never hardcode file paths.

```
Manifest URL:
https://raw.githubusercontent.com/{org}/{repo}/main/standards/content-types/release-notes.yaml
```

> Replace `{org}/{repo}` with the central .github repository (e.g. `svahsek/.github`).
> For stability in CI/CD, pin to a commit SHA instead of `main`.

From the manifest, resolve and load:

| Entrypoint key | What to do with it |
|---|---|
| `schema` | Load — defines required fields and validation patterns |
| `rendering` | Load — defines how to convert YAML to Markdown |
| `human_rules` | Load — defines STR-*, CON-*, SEC-* validation rule IDs |
| `skill` | Reference only — this file IS the consumer version of that skill |
| `quick_reference` | Load — use as fill-in template during Step 5 |

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
   https://raw.githubusercontent.com/{org}/{repo}/main/standards/classification-rules.yaml

2. Consumer overrides (local, product-specific):
   .github/standards/classification-overrides.yaml
   → If file does not exist, use central baseline only
```

Merge behaviour:
- `additional_prefixes` in overrides → appended to central prefixes list
- `additional_keywords` in overrides → appended to central keywords list
- `omit_from_output` in overrides → overrides central value
- `jira_project_key` in overrides → used in issue reference pattern for CON-002

Log: `"Classification rules loaded. Consumer overrides: [found/not found]"`

---

## Step 3 — Collect Git Data

Resolve target ref in this order:
1. `release_ref`
2. `release_branch` (legacy fallback)

Run all commands against `base_tag..target_ref`.

If `evidence_source` exists in inputs:
- Use `evidence_source.release_ref` as the target ref when provided (highest priority override)
- Use `evidence_source.milestone` for issue/PR filtering
- Apply `evidence_source.issue_query` and `evidence_source.pr_query` if provided
- Restrict results by `include_issue_labels` and `include_pr_labels` when present
- Include QA/security evidence based on `include_qa_evidence` and `include_security_scans`

```bash
# 1. Commit log — primary classification input
git log {base_tag}..{target_ref} \
  --pretty=format:"COMMIT|%H|%s|%an|%ad" \
  --date=short \
  --no-merges

# 2. File change summary
git diff --stat {base_tag}..{target_ref}

# 3. Full diff — for deep analysis
git diff {base_tag}..{target_ref}
```

> ⚠️ If the full diff exceeds ~4000 lines, switch to file-by-file analysis:
> ```bash
> # Get changed files
> git diff --name-only {base_tag}..{target_ref}
>
> # Diff each file in priority order:
> # 1. Source files (src/, lib/, app/)
> # 2. Config files (*.yaml, *.json, *.toml)
> # 3. API specs (openapi.yaml, swagger.yaml)
> # Skip: test/, docs/, *.md, generated files
> git diff {base_tag}..{target_ref} -- {file}
> ```

Additionally, if GitHub/JIRA access is available:

```bash
# Linked issues from commit messages — extract issue references
git log {base_tag}..{target_ref} --no-merges \
  --pretty=format:"%s %b" | grep -oE "(#[0-9]+|[A-Z]+-[0-9]+)"
```

---

## Step 4 — Reconcile Evidence (Canonical Model)

Transform raw extracted signals into canonical release evidence.

Use these authorities:
1. `source-precedence.yaml` for field-level source selection
2. `release-evidence-fields.yaml` for field semantics
3. `classification-rules.yaml` for change typing
4. `release-evidence-confidence-rule.yaml` for confidence scoring

Reconciliation tasks:
1. Merge issue, PR, commit, QA, and build/security signals by linkage keys
2. Apply field precedence deterministically (primary then fallback sources)
3. Resolve conflicts and collect unresolved items
4. Classify change type from labels/prefix/keywords
5. Score confidence and assign publication tier
6. Populate `normalized_for_release_notes` mapping fields

**Output of this step**:
- `docs/releases/{version}-release-evidence.yaml`
- `unresolved[]` entries for ambiguous/conflicting items

---

## Step 5 — Validate Canonical Evidence (Gate)

Validate `release-evidence.yaml` before generating any release-notes content.

Evidence gate checks:
1. Contract shape matches release evidence contract
2. Required lifecycle fields exist (intent/change/evidence/shipped)
3. Precedence/provenance fields are present where required
4. Confidence tiers are assigned and unresolved items are separated

Blocking rule:
- If evidence validation fails, stop here.
- Do not generate `release-notes.yaml`.

## Step 6 — Generate Structured Release Notes YAML

Using the schema from Step 1 and reconciled evidence from Step 4, fill the release YAML.
Use the mapping file and quick-reference as the fill-in template.

```yaml
metadata:
  version: "{inputs.version}"
  releaseDate: "{inputs.release_date}"           # YYYY-MM-DD format for schema
  releaseType: "{inputs.release_type}"
  productName: "{inputs.product_name}"
  productNameAbbr: "{inputs.product_name_abbr}"
  targetAudience: {inputs.target_audience}

summary:
  headline: ""          # Agent writes: benefit-driven, 10-150 chars, no marketing language
  paragraphs:
    - text: ""          # Agent writes: Purpose paragraph — what this release addresses
      sectionLabel: "Purpose"
    - text: ""          # Agent writes: ThemesAndFocus — key themes across the changes
      sectionLabel: "ThemesAndFocus"
  keyBenefits: []       # Agent writes: 1-5 quantified benefits derived from evidence

content:
  newFeatures:          # Populated from release-evidence.yaml (filtered by mapping and confidence)
  enhancedFeatures:     # Populated from release-evidence.yaml
  bugFixes:             # Populated from release-evidence.yaml
  securityUpdates:      # Populated from release-evidence.yaml
  technicalImprovements: # Populated from release-evidence.yaml
  deprecations:         # Populated from release-evidence.yaml
  knownIssues:          # Populated from linked issues marked as known issues
  compatibleModules: [] # CON-006: must be filled — agent prompts if not in inputs
  repositories:         # CON-007: must be filled — agent extracts from git tags
    released: []
  supportResources:
    documentation:
      feature_docs: ""  # CON-008: required
      api_docs: ""      # CON-008: required
      test_report: ""   # CON-009: required for Major/Minor
```

**Agent writing rules during this step:**
- `summary.headline`: active voice, benefit-driven, no superlatives (PRO-001)
- `summary.paragraphs`: factual, no marketing language, 50-500 chars each
- Feature descriptions: lead with user value, not implementation detail
- Bug fix descriptions: "Fixed [what] which caused [impact]" pattern
- All JIRA refs: format as `{jira_project_key}-{number}` using key from inputs

---

## Step 7 — Validate Structured Content

Run all deterministic checks from the human_rules file loaded in Step 1.
Process rules in this order:

**1. Structure rules (STR-*) — check first, stop if ERROR**
- STR-001: required sections present
- STR-002: version matches pattern `^v?\d+\.\d+\.\d+`
- STR-003: releaseDate in YYYY-MM-DD format

**2. Content rules (CON-*)**
- CON-001: overview paragraph present
- CON-002: issue refs in bug fixes and known issues match `(#\d+|GH-\d+|[A-Z]+-\d+)`
- CON-003: entry descriptions 10+ words, under 200 chars
- CON-004: no section headers with zero entries
- CON-005: if `migrationRequired=true`, `migrationNotes` must not be empty
- CON-006: `compatibleModules` present with at least one entry
- CON-007: `repositories.released` present with at least one entry
- CON-008: `supportResources.documentation` includes `feature_docs` and `api_docs`
- CON-009: if `releaseType` is Major or Minor, `test_report` link required
- CON-010: if `knownIssues` present, `external_tracker_url` required
- CON-011: if `releaseType` is Patch or Hotfix, `newFeatures` must be absent or empty
- CON-012: if `releaseType` is Major or Minor, `userStories` must be present
- CON-013: no placeholder text (TBD, TODO, FIXME, WIP, Coming Soon) in any field

**3. Security rules (SEC-*)**
- SEC-001: no internal email addresses, private IPs, team names, personal names
- SEC-002: no comparative marketing language
- SEC-003: release name contains productName and version values

**Report format for violations:**
```
❌ CON-005 [ERROR] enhancedFeatures[0]: migrationRequired=true but migrationNotes is empty
   Fix: add migration instructions to migrationNotes field

⚠️ CON-002 [WARNING] bugFixes[1]: no issue reference found in relatedIssues
   Fix: add GitHub (#123) or JIRA (INJICERT-456) reference

ℹ️ CON-012 [WARNING] userStories: section absent for Minor release
   Fix: add user stories table or change releaseType to Patch
```

**On ERROR:** Stop. Do not render Markdown. Output the YAML with errors marked. Require human fix before proceeding.

**On WARNING only:** Proceed to rendering but include warnings in a validation report appended to the output.

---

## Step 8 — Render Markdown

Using the rendering rules loaded in Step 1, convert the validated YAML to Markdown.

Apply rendering rules in section order:
- HDR-001, HDR-002: header date formatting and metadata visibility
- ROL-001: rollout quick reference block (if rolloutTimeline populated)
- SUM-001, SUM-002, SUM-003: headline, paragraphs, key benefits
- FEA-001 through FEA-004: new features (H4 headings, use case first, clickable links, issue links)
- ENH-001, ENH-002: enhancements (before/after tables, conditional migration rows)
- BUG-001, BUG-002: bug fixes (severity badges, workarounds)
- RSEC-001, RSEC-002, RSEC-003: security updates (warning header, CVE links, upgrade callout)
- API-001, API-002, API-003: API changes (code formatting, deprecation labels, param lists)
- IMP-001, IMP-002: technical improvements (quantified metrics required, component list)
- ISS-001: known issues (workarounds prominent)

---

## Outputs

Save both files to `{inputs.output_dir}` (default: `docs/releases/`).

| File | Name pattern | Purpose |
|---|---|---|
| Canonical evidence | `{version}-release-evidence.yaml` | Normalized, reconciled, scored evidence source |
| YAML source | `{version}-release-notes.yaml` | Source of truth for automation |
| Markdown output | `{version}-release-notes.md` | Publishable release notes |
| Validation report | `{version}-validation-report.txt` | Warnings and rule check results |

Example for version `0.14.0`:
```
docs/releases/
  0.14.0-release-evidence.yaml
  0.14.0-release-notes.yaml
  0.14.0-release-notes.md
  0.14.0-validation-report.txt
```

If validation produced ERRORs, output only:
```
docs/releases/
  0.14.0-release-evidence.yaml  ← canonical evidence (required)
  0.14.0-release-notes.yaml      ← with error annotations inline
  0.14.0-validation-report.txt   ← full error list
```
Do not create the `.md` file until all ERRORs are resolved.

---

## Trigger Modes

### Manual (VS Code Copilot chat)
1. Edit `docs/releases/release-inputs.yaml` with release details
2. Open Copilot chat and invoke this skill
3. Review outputs in `docs/releases/`
4. Fix any ERRORs flagged in validation report
5. Commit both YAML and Markdown files

### CI/CD (GitHub Actions)
```yaml
# .github/workflows/release-notes.yml
on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release-notes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0          # required for full git history

      - name: Write release inputs
        run: |
          cat > docs/releases/release-inputs.yaml << EOF
          version: "${{ github.ref_name }}"
          base_tag: "$(git describe --tags --abbrev=0 HEAD^)"
          release_ref: "${{ github.ref_name }}"
          release_type: "Minor"
          product_name: "Inji Certify"
          product_name_abbr: "INJICERT"
          jira_project_key: "INJICERT"
          output_dir: "docs/releases/"
          target_audience:
            - Developers
            - Implementers
          EOF

      - name: Generate release notes
        uses: github/copilot-action@v1     # or equivalent agent trigger
        with:
          skill: .github/skills/release-notes.skill.md
```

---

## Fallback Handling

| Failure | Behaviour |
|---|---|
| Manifest URL unreachable | Use local `.github/standards/` cache if present; log WARNING; skip validation if no cache |
| Classification rules URL unreachable | Use local `classification-overrides.yaml` only; log WARNING |
| `git log` returns zero commits | Stop with ERROR: "No commits found between {base_tag} and {target_ref}. Verify refs exist." |
| Evidence validation fails | Stop before generation. Output `release-evidence.yaml` + validation report only. |
| Required input field unresolvable | Stop with ERROR listing unresolved fields |
| All commits classify as internal | Warn: "All changes classified as internal. Verify base_tag is correct." Produce minimal release notes with only metadata and summary. |
| Validation ERRORs present | Stop before rendering. Output YAML + validation report only. |
