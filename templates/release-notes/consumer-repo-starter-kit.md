# Consumer Repo Starter Kit — Release Notes

## Purpose

This guide shows the minimum setup required in any product (consumer) repository to use the release-notes standards from this standards repository.

Configuration is inline: the consumer skill embeds per-repo and per-release settings directly in its own Step 0a ("Release Configuration") block. There is no separate input file to create, find, or keep in sync with the skill file.

---

## What You Need In Consumer Repo

### Required

1. Local consumer skill file, adapted for this repo — copy of
   [`consumer-release-notes.skill.md`](../../skills/release-notes/consumer-release-notes.skill.md)
2. Output folder for generated artifacts

### Generated Each Release Run

1. `release-evidence.yaml` (normalized, reconciled, scored)
2. `release-notes.yaml` (structured content)
3. `release-notes.md` (publishable markdown)
4. `validation-report.txt` (rule outcomes)

### Optional

1. `classification-overrides.yaml` (product-specific categorization rules)
2. Local cache of standards files for offline/air-gapped usage (`.github/standards/`)
3. A PO-written release notes draft and/or a `.github/release.yml` taxonomy — both are
   conditional accelerants the skill probes for automatically; neither needs to exist for the
   skill to produce full-quality output. See the skill's "Extraction Philosophy" section.

---

## Minimal Folder Layout (Consumer Repo)

```text
consumer-repo/.github/
  skills/
    consumer-release-notes.skill.md         # copied and adapted — configuration lives inside it
  standards/
    classification-overrides.yaml           # optional

docs/
  releases/
    v1.2.3-release-evidence.yaml            # generated
    v1.2.3-release-notes.yaml               # generated
    v1.2.3-release-notes.md                 # generated
    v1.2.3-validation-report.txt            # generated
    po-notes/
      v1.2.3.md                              # optional accelerant, only if a PO draft exists
```

Notice there is no `release-inputs.yaml` anywhere in this layout — configuration lives in the
skill file itself.

---

## Setup Steps

1. Copy [`consumer-release-notes.skill.md`](../../skills/release-notes/consumer-release-notes.skill.md)
   into `consumer-repo/.github/skills/`.
2. Open the copy and fill in the **PER-REPO** section of the Step 0a Release Configuration
   block once (`product_name`, `product_name_abbr`, `jira_project_key`, `output_dir`,
   `target_audience`, `documentation` links, `compatible_modules`, `repositories_released`).
   This is a one-time step per repo, not per release.
3. Resolve manifest/schema/rendering URLs from Step 1 of the skill (points at this central
   repo) — pin to a commit SHA in CI for stability.

---

## What To Edit Per Release

Edit only:
1. The **PER-RELEASE** fields in the Step 0a block of your copied skill file
   (`version`, `base_tag`, `release_ref`, `release_type`, `release_date`) — or, in CI, set the
   equivalent env vars (`RELEASE_VERSION`, `RELEASE_BASE_TAG`, `RELEASE_REF`, `RELEASE_TYPE`,
   `RELEASE_DATE`) instead of editing the file.
2. `unresolved` items in `release-evidence.yaml` (only if confidence < 60 or validation fails).

Do not edit per release:
1. Source-precedence rules
2. Confidence scoring model
3. Rendering rules
4. Schema definitions
5. The **PER-REPO** section of Step 0a — that's set once, not per run

Those are standards-level assets and should remain centralized.

---

## Central Standards You Fetch (Read-Only in Consumer Repo)

1. `standards/content-types/release-notes.yaml` (manifest)
2. `standards/source-precedence.yaml`
3. `standards/release-evidence-fields.yaml`
4. `rules/release-evidence-confidence-rule.yaml`
5. `templates/release-notes/release-notes-schema.yaml`
6. `templates/release-notes/evidence-to-release-notes-mapping.yaml`
7. `templates/release-notes/release-notes-rendering.yaml`
8. `rules/release-notes/release-notes-rule.yaml`

Pin to commit SHA in CI for stability.

---

## Run Modes

### Manual (Agent in VS Code)

1. Edit the PER-RELEASE fields in Step 0a of your copied skill file.
2. Invoke the skill.
3. Review the validation report.
4. Publish markdown if no blocking errors.
5. Commit the generated outputs and the edited skill file together.

### CI Mode

1. Trigger on tag push (`vX.Y.Z`).
2. Export `RELEASE_VERSION`, `RELEASE_BASE_TAG`, `RELEASE_REF`, `RELEASE_TYPE`, `RELEASE_DATE`
   as job env vars — these override the skill file's PER-RELEASE block for that run only,
   without touching the file.
3. Run the skill.
4. Upload generated artifacts.
5. Fail the job on validation errors.

---

## Decision Rule

If validation fails, stop publication and fix data first.

No partial release-notes publication.

---

## Quick Adoption Checklist

- [ ] Added `consumer-repo/.github/skills/consumer-release-notes.skill.md` to consumer repo
- [ ] Filled the PER-REPO section of its Step 0a block once
- [ ] Confirmed output folder `docs/releases/`
- [ ] Confirmed standards manifest URL and pinned SHA
- [ ] Dry-run completed on one past release
- [ ] Validation report reviewed and accepted
- [ ] CI env var mapping confirmed, if using CI mode
