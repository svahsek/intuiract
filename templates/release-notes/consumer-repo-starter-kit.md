# Consumer Repo Starter Kit — Release Notes

## Purpose

This guide shows the minimum setup required in any product (consumer) repository to use the release-notes standards from this standards repository.

## What You Need In Consumer Repo

### Required

1. Local consumer skill file (orchestrator only)
2. Per-release input file
3. Output folder for generated artifacts

### Generated Each Release Run

1. release-evidence.yaml (normalized, reconciled, scored)
2. release-notes.yaml (structured content)
3. release-notes.md (publishable markdown)
4. validation-report.txt (rule outcomes)

### Optional

1. classification-overrides.yaml (product-specific categorization rules)
2. local cache of standards files for offline/air-gapped usage

---

## Minimal Folder Layout (Consumer Repo)

```text
consumer-repo/.github/
  skills/
    release-notes.skill.md
  standards/
    classification-overrides.yaml           # optional

docs/
  releases/
    release-inputs.yaml                    # per-release operator inputs
    v1.2.3-release-evidence.yaml           # generated
    v1.2.3-release-notes.yaml              # generated
    v1.2.3-release-notes.md                # generated
    v1.2.3-validation-report.txt           # generated
```

---

## Copy-Ready Files

### 1. Consumer Skill

Use this template:
- [examples/consumer-release-notes.skill.template.md](examples/consumer-release-notes.skill.template.md)

Or use the fuller version in this repo:
- [../../.github/skills/release-notes/consumer-release-notes.skill.md](../../.github/skills/release-notes/consumer-release-notes.skill.md)

### 2. Release Inputs

Copy and fill this input file in consumer repo:
- [examples/release-inputs.yaml](examples/release-inputs.yaml)

---

## What To Edit Per Release

Edit only:
1. docs/releases/release-inputs.yaml
2. unresolved items in release-evidence.yaml (only if confidence < 60 or validation fails)

Do not edit per release:
1. source-precedence rules
2. confidence scoring model
3. rendering rules
4. schema definitions

Those are standards-level assets and should remain centralized.

---

## Central Standards You Fetch (Read-Only in Consumer Repo)

1. standards/content-types/release-notes.yaml (manifest)
2. standards/source-precedence.yaml
3. standards/release-evidence-fields.yaml
4. rules/release-evidence-confidence-rule.yaml
5. templates/release-notes/release-notes-schema.yaml
6. templates/release-notes/evidence-to-release-notes-mapping.yaml
7. templates/release-notes/release-notes-rendering.yaml
8. rules/release-notes/release-notes-rule.yaml

Pin to commit SHA in CI for stability.

---

## Run Modes

### Manual (Agent in VS Code)

1. Fill docs/releases/release-inputs.yaml
2. Invoke local consumer skill
3. Review validation report
4. Publish markdown if no blocking errors

### CI Mode

1. Trigger on tag push (vX.Y.Z)
2. Populate release-inputs.yaml from pipeline context
3. Run skill/toolchain
4. Upload generated artifacts
5. Fail job on validation errors

---

## Decision Rule

If validation fails, stop publication and fix data first.

No partial release-notes publication.

---

## Quick Adoption Checklist

- [ ] Added consumer-repo/.github/skills/release-notes.skill.md to consumer repo
- [ ] Added docs/releases/release-inputs.yaml template
- [ ] Confirmed output folder docs/releases/
- [ ] Confirmed standards manifest URL and pinned SHA
- [ ] Dry-run completed on one past release
- [ ] Validation report reviewed and accepted
