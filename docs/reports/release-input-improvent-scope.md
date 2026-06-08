# Release Input Improvement Scope

## Context

This report captures honest feedback on how `release-inputs.yaml` performed during release notes generation, where token spend actually happened, and three options for improving the skill.

---

## What `release-inputs.yaml` Actually Did

It helped for every metadata field — nothing was re-derived or prompted:

| Field | Came from inputs | Without it |
|---|---|---|
| `version` | ✅ `0.18.0` | Would have parsed branch name |
| `base_tag` | ✅ `v0.17.0` | Would have run `git describe` |
| `release_ref` | ✅ `release-0.18.x` | Would have used current branch |
| `release_type` | ✅ Minor | Would have inferred from semver diff |
| `product_name` / `abbr` | ✅ directly used | ❌ Cannot be derived from git — would have prompted |
| `compatible_modules` | ✅ directly used | ❌ Cannot be derived from git — would have prompted |
| `repositories_released` | ✅ directly used | ❌ Same |
| `documentation URLs` | ✅ directly used | ❌ Same |
| `output_dir` | ✅ `docs/releases/` | Default would have been used |

The inputs file fully eliminated prompting and auto-detection overhead for all structural fields.

---

## What It Could NOT Help With

The inputs file is a metadata file, not a content file. It doesn't and can't contain what actually changed. For content, git commands are unavoidable:

```bash
git log v0.17.0..release-0.18.x   # no way around this
```

That single command is the core token spend and it's necessary regardless of how good the inputs file is.

---

## Where Tokens Were Burned Unnecessarily

| Command | Necessary? | Why it was run |
|---|---|---|
| `git log` (140 commits) | ✅ Core | Required for all content |
| `git diff --stat` | ⚠️ Optional | Useful but commit messages were sufficient |
| `git diff -- VPRequestController.java` | ⚠️ Optional | Discovered cookie session — but commit messages hinted at it |
| `git diff -- VPResultController.java` | ⚠️ Optional | Similar — not strictly needed |
| `git diff -- openapi.yaml \| grep` | ⚠️ Optional | Quick scan but still a full diff |
| `cat 0.17.0_to_0.18.0_upgrade.sql` | ✅ High value | Found DB migration requirement — commit messages alone would have missed this |

The DB migration script read was the only deep dive that genuinely earned its token cost — it revealed a mandatory operator action that no commit message mentioned. The controller diffs were a nice-to-have.

---

## Options to Fix the Skill

### Option A — Add a `known_changes` section to `release-inputs.yaml`

Let contributors pre-populate key features before running the skill. The skill uses these as the primary content source and only runs `git log` to fill gaps:

```yaml
known_changes:
  new_features:
    - "Web Wallet same-device VP flow (INJIVER-1438)"
    - "VP Session API endpoints (INJIVER-1580)"
  bug_fixes:
    - "Invalid VP token acceptance fixed (INJIVER-1614)"
  migration_required: true
  migration_script: "db_upgrade_script/inji_verify/sql/0.17.0_to_0.18.0_upgrade.sql"
```

When this section exists, the skill skips deep file diffs entirely and only runs `git log` for verification and gap-filling.

### Option B — Add a depth control to `release-inputs.yaml`

```yaml
analysis_depth: "commits_only"   # commits_only | commits_and_stats | full_diff
```

| Value | Behaviour | Token cost |
|---|---|---|
| `commits_only` | `git log` only, no diffs | Minimum |
| `commits_and_stats` | `git log` + `git diff --stat` | Balanced |
| `full_diff` | Current behaviour | Maximum quality |

### Option C — Reserve deep diffs for flagged commits only

Run `git log` first, identify commits with ambiguous messages (no ticket reference, no conventional prefix), and only run `git diff` on those specific files. Saves approximately 60% of diff tokens.

---

## Bottom Line

The inputs file is working correctly for its purpose. Token spend is in the git content extraction layer, not the metadata layer. The skill needs either a depth control (Option B) or a `known_changes` pre-population option (Option A) so analysis depth can be dialled down when the team already knows what shipped.
