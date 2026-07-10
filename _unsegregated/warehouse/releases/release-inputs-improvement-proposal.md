# Improving `release-inputs.yaml` for Faster `/crn-iv` Runs

Synthesis of [`0.17.0-validation-report.txt`](0.17.0-validation-report.txt) and
[`release-notes-process-retrospective.md`](release-notes-process-retrospective.md). Both
documents describe symptoms; this one traces each symptom back to a missing or ambiguous field
in `release-inputs.yaml` and proposes a fix. The goal is a version of that file where a future
agent run for this repo needs **zero clarifying questions and zero trial-and-error** before it
starts producing output.

> **Revision note:** the organization is moving off JIRA onto GitHub Issues for tracking. Every
> field and recommendation below that originally assumed JIRA (`jira_project_key`,
> `jira_fix_version`, JIRA-sourced `userStories`) has been replaced with a GitHub-native
> equivalent (`github_milestone`, label-driven classification, Issues queried by milestone). See
> the "Industry practice" section near the end for what mature GitHub-based orgs do instead of
> JIRA, and [`release-inputs.v2.yaml`](release-inputs.v2.yaml) for the concrete revised file.

## Method

Every warning in the validation report and every time sink in the retrospective was one of two
things: (a) information the input file *could* have supplied but didn't, or (b) a decision the
agent had to make by trial-and-error that the input file could have made explicit. The table
below maps each observed problem to a proposed field.

| Observed problem | Source | Root cause | Proposed input field |
|---|---|---|---|
| `release_ref` pointed at a branch/tag that didn't exist (`release-v0.17.0`) | This run's Step 0 | No pre-flight check that `release_ref` resolves in git before the skill commits to it | Skill-side fix (see "Pre-flight checks" below), not a new field — but see `release_ref` guidance change |
| Duplicate `release_ref:` key silently overrode the real value | This run's Step 0 | Free-form commented-out block left an uncommented line with the same key as an earlier active field | Restructure file so every key appears exactly once, always active, with empty string for "unset" (see proposed template) |
| File found at `docs/release-inputs.yaml` instead of `docs/releases/release-inputs.yaml` | This run's Step 0 | No canonical-location check/warning when a similarly-named file exists elsewhere | Skill-side fix: if the expected path is missing, glob for `**/release-inputs.yaml` and warn if a near-miss is found |
| CON-009: no `test_report` link | Validation report | No field exists for it | `test_report_url` (optional, but skill should ask directly instead of falling back to a warning) |
| CON-012: no `userStories` | Validation report | No GitHub milestone was given, so the agent had no scoped issue list to query (originally this gap was framed as "JIRA wasn't queried" — see revision note above) | `github_milestone` (required when `release_type` is Major/Minor); userStories sourced via `gh issue list --milestone <github_milestone> --state closed` |
| Deprecation with unknown `removalVersion` | Validation report | No field captures planned removal versions for known deprecations | `known_deprecations:` list (see template) |
| ~15-20 min spent on PR-fetch trial-and-error | Retrospective §1 | Input file gives no instruction on *how* to fetch PR/issue evidence efficiently, so the agent improvised (badly) | `evidence_source.fetch_strategy: graphql_batch`, `evidence_source.repo` (explicit `owner/name`), `evidence_source.label_usage: none` |
| Repo identity ambiguity (`origin` = `svahsek/inji-verify`, `upstream` = `inji/inji-verify`) — agent had to infer which to use for generated links | Retrospective (implicit — see doc links in `0.17.0-release-notes.md`) | No explicit canonical repo field | `evidence_source.repo: "inji/inji-verify"` |
| Standards re-fetched from network (WebFetch misfire, then curl) | Retrospective §3 | No local cache, no pinned ref | `standards.cache_dir`, `standards.pinned_ref` |
| Noisy keyword classification false-positives (`securityContext` matched `security`) | Retrospective §2 | Not really an input-file problem — this is a `classification-rules.yaml` precision issue upstream | Out of scope for `release-inputs.yaml`; note only |
| No way to say "just give me a fast draft" vs "full quality with PR enrichment" | Implicit across both docs | No speed/quality knob exists | `generation_mode: fast \| full` |

## Proposed additions to the schema

None of these break the existing required-fields contract (`version`, `base_tag`,
`release_ref`, `release_type`, `product_name`). They're additive.

```yaml
# ─── REPO IDENTITY (new) ───────────────────────────────────────────────────
# Removes the need for the agent to infer this from `git remote -v`, which is
# ambiguous whenever origin (fork) and upstream (canonical) differ, as they do
# in this repo (origin=svahsek/inji-verify, upstream=inji/inji-verify).
github_repo: "inji/inji-verify"            # owner/name used for all generated links, issue refs, PR refs

# ─── EVIDENCE FETCH STRATEGY (new) ─────────────────────────────────────────
# Tells the agent HOW to pull PR/issue evidence instead of letting it improvise.
# This is the single highest-leverage field for run speed on repos with 50+ commits.
evidence_source:
  fetch_strategy: "graphql_batch"          # graphql_batch (fast, default) | rest_sequential | commits_only (fastest, lowest quality)
  label_usage: "none"                      # none | full — this repo doesn't use PR/issue labels yet; skip label-based filtering entirely instead of discovering that at runtime. Flip to "full" once the label taxonomy below is actually applied to issues/PRs.
  github_milestone: "0.17.0"               # GitHub milestone scoping this release — replaces JIRA fixVersion. Required if release_type in [Major, Minor] and CON-012 (userStories) matters.
  label_taxonomy:                          # what label on an Issue/PR maps to which release-notes section — replaces free-text keyword classification
    "type:feature": newFeatures
    "type:enhancement": enhancedFeatures
    "type:bug": bugFixes
    "type:security": securityUpdates
    "type:breaking": enhancedFeatures       # + migrationRequired=true
    "type:deprecation": deprecations
    "type:chore": internal                  # omitted from output

# ─── KNOWN DEPRECATIONS (new) ──────────────────────────────────────────────
# Prevents the "unknown removalVersion" gap. If a deprecation is known at input
# time, state it here; the agent should never guess a removal version.
known_deprecations:
  - item: "@mosip/react-inji-verify-sdk"
    replacement: "@injistack/react-inji-verify-sdk"
    tracking_issue: ""                     # GitHub issue number tracking the removal, e.g. "#1300" — replaces JIRA ticket reference
    removal_version: ""                    # leave empty string (not omitted) if genuinely undecided — agent must NOT fabricate this

# ─── SUPPORT RESOURCES (new) ───────────────────────────────────────────────
test_report_url: ""                        # CON-009 — leave empty if unavailable; skill should ask once rather than silently warn

# ─── STANDARDS CACHE (new) ─────────────────────────────────────────────────
standards:
  cache_dir: ".github/standards/"          # if populated, skip network fetch entirely
  pinned_ref: ""                           # commit SHA in the standards repo, for CI reproducibility — empty means "use main, log a warning"

# ─── SPEED/QUALITY KNOB (new) ──────────────────────────────────────────────
generation_mode: "full"                    # full (fetch + read PR bodies, best descriptions) | fast (commit subjects only, ~10x faster, rougher copy)
```

## Structural fix: stop using comments to mean "unset"

The duplicate-key bug in this run's original `release-inputs.yaml` happened because the
"optional / commented out" convention relies on humans (or agents editing the file) correctly
leaving a `#` in front of every line in a block — and correctly *not* leaving a stray active
line with the same key as something already set above it. That's exactly what happened:
`release_ref:` was set once near the top, then a second `release_ref:` was left active inside
what was meant to be a fully-commented `evidence_source:` block.

**Proposal:** every key in `release-inputs.yaml` should always be present and always active,
using an empty string / empty list to mean "not provided," rather than toggling entire lines
between commented and active. This makes the file impossible to accidentally duplicate a key
in, and makes "is this field set?" a `!= ""` check instead of "is this line commented?" —
trivial for both a human skimming the file and an agent parsing it.

## Proposed pre-flight checks (skill-side, not input-file, but triggered by input-file content)

These belong in the `/crn-iv` skill's Step 0, run immediately after loading
`release-inputs.yaml` and before Step 1 (standards fetch) begins:

1. **Resolve `release_ref` before doing anything else.** Run
   `git rev-parse --verify <release_ref>` (or `--verify refs/tags/<release_ref>` /
   branch equivalent). If it fails, stop immediately and ask — don't proceed 7 steps deep and
   discover the ref problem while running `git log`. This alone would have caught this run's
   scope ambiguity in the first 10 seconds instead of after a full read of the input file plus
   git history.
2. **Warn on near-miss file locations.** If `docs/releases/release-inputs.yaml` doesn't exist,
   glob for `**/release-inputs.yaml` before falling back to auto-detect-and-prompt. A file at
   `docs/release-inputs.yaml` (one directory up) is very likely the intended input, misplaced.
3. **Parse-validate for duplicate keys**, not just presence of required fields. Python's
   `yaml.safe_load` silently accepts duplicate keys and keeps the last one — exactly the bug
   that happened here. A stricter loader (or a `yamllint` pass with duplicate-key detection
   enabled) run against `release-inputs.yaml` before trusting any field would have caught this
   immediately.
4. **Check `evidence_source.fetch_strategy` before writing any fetch code.** If unset, default
   to `graphql_batch` for any range with more than ~20 commits, rather than starting with a
   naive sequential/parallel REST loop and discovering the slowness empirically.

## Industry practice: how mature GitHub-native orgs extract release-note data

Moved to its own document — see
[`how-mature-organizations-release.md`](how-mature-organizations-release.md) for the full
comparison (GitHub native auto-generated notes, release-drafter, Conventional Commits,
Kubernetes-style release-note blocks, Changesets, milestone-based scoping) and a suggested
adoption order for this repo. Short version: every mature pattern pushes classification to the
moment the change is authored (labels, enforced commit grammar, author-written snippets)
instead of reconstructing it after the fact the way this run had to — that's the actual
long-term fix; the input-file changes in this document are the best available mitigation given
the current, unlabeled state of this repo's history.

## Priority if implementing

1. `evidence_source.fetch_strategy: graphql_batch` + `github_repo` — directly eliminates the
   retrospective's #1 time sink (~15-20 min → seconds).
2. Pre-flight `release_ref` resolution check — directly eliminates this run's scope-ambiguity
   detour and the wasted mkdir/interrupt/re-invoke cycle.
3. `standards.cache_dir` + populate `.github/standards/` once for this repo — removes the
   WebFetch/curl fumbling and all future network dependency for Steps 1-2.
4. Structural fix (no more comment-toggling for optional fields) — cheap to do, removes an
   entire class of "silently wrong config" bugs like the one that started this whole run.
5. `known_deprecations`, `test_report_url`, `github_milestone` — closes the 3 validation
   warnings from this run specifically; lower urgency since they're warnings, not errors.
6. `generation_mode: fast` — nice to have for quick drafts, but shouldn't be the default; the
   PR-body enrichment materially improved description quality in this run (compare the git
   evidence file's bare commit subjects to the final release notes prose).
7. **(Longer-term, highest ultimate payoff)** Adopt a PR/issue label taxonomy + milestones and,
   ideally, a commit/PR-title convention enforced by CI. This is the only change that removes
   classification uncertainty entirely rather than mitigating it — see
   [`how-mature-organizations-release.md`](how-mature-organizations-release.md). Everything
   else in this document is a mitigation for the *current* unlabeled state of this repo's
   history; this item is the actual fix.
