# Release Notes: `supportResources.documentation` field mismatch + CON-008 severity

**Status**: Open — no fix applied yet, pending decision
**Found**: 2026-07-14, while reviewing manual clean-up needed after two real `/crn-iv`-style runs
(`1.0.0-alpha.1` in the `inji-verify` fork, and one additional run this week)
**Area**: `templates/release-notes/`, `rules/release-notes/`, `skills/release-notes/`

---

## Issue 1 — Schema/rendering vocabulary doesn't match what every real run actually produces

### Summary

`templates/release-notes/release-notes-schema.yaml`'s `supportResources.documentation` object
defines a vocabulary that no consumer file, and no real generated `release-notes.yaml`, actually
uses. Two independent vocabularies exist in this repo for the same field, with no mapping between
them:

| Vocabulary A (schema-side) | Vocabulary B (rule/skill/evidence-side) |
|---|---|
| `releasePage` | *(no equivalent)* |
| `upgradeGuide` | *(no equivalent)* |
| `troubleshootingGuide` | *(no equivalent)* |
| `apiReference` | `api_docs` |
| *(no equivalent)* | `feature_docs` |
| *(no equivalent)* | `test_report` |
| *(no equivalent)* | `collab_guide` |

These are not simple renames of the same four concepts — the two sides only overlap on one field
(`apiReference` ↔ `api_docs`). `test_report` and `collab_guide` have no field anywhere in the
schema, under any name.

### Evidence

- **Vocabulary A** — used consistently by:
  - `templates/release-notes/release-notes-schema.yaml:755-766` (the declared source of truth)
  - `templates/release-notes/release-notes-rendering.yaml:535-538`
  - `templates/release-notes/examples/comprehensive-release-notes.yaml:374-379`
- **Vocabulary B** — used consistently by:
  - `rules/release-notes/release-notes-rule.yaml:78-87` (CON-008, CON-009)
  - `skills/release-notes/consumer-release-notes.skill.md` (Step 0a input block, Step 6 fill-in
    template)
  - `standards/source-precedence.yaml`, `standards/release-evidence-fields.yaml`,
    `templates/release-notes/examples/release-evidence.sample.yaml`
- **No bridge**: `templates/release-notes/evidence-to-release-notes-mapping.yaml` has no entry
  translating either vocabulary into the other for this field.
- **Real production evidence, twice**: the actual generated `1.0.0-alpha.1-release-notes.yaml` (in
  the `inji-verify` fork) and a second real run this week both independently produced
  `supportResources.documentation.{feature_docs, api_docs, collab_guide, test_report}` —
  Vocabulary B — never Vocabulary A. Vocabulary A has, as far as this investigation found, never
  been produced by an actual run.

### Impact

- Strict validation against `release-notes-schema.yaml` would reject every real payload produced
  so far, since none of them use the schema's declared field names.
- CON-008/CON-009 validate fields (`feature_docs`, `api_docs`, `test_report`) that don't exist in
  the schema at all — a rule checking for a field the schema never defined.
- `rendering.yaml`'s template (`{documentation.releasePage}` etc.) would render empty links for
  every real payload, since no generator has ever populated those field names.

### Options (not decided)

- **A. Fix schema/rendering to match Vocabulary B** — adopt `feature_docs`/`api_docs`/
  `test_report`/`collab_guide` as canonical in `schema.yaml` and `rendering.yaml`, since that's
  what every real run has actually produced. Retire or explicitly deprecate
  `releasePage`/`upgradeGuide`/`troubleshootingGuide`/`apiReference`.
  Direction favored by this investigation.
- **B. Fix Vocabulary B to match schema** — would require adding `evidence-to-release-notes-mapping.yaml`
  entries to translate operator input into the schema's names, and would still leave
  `test_report`/`collab_guide` needing new schema fields since nothing in Vocabulary A covers them.

---

## Issue 2 — CON-008 is a hard blocker; CON-006 already set a precedent for downgrading the same class of problem

### Summary

CON-008 (`feature_docs`/`api_docs` required) is `severity: error` — a run cannot render Markdown
without these filled in. CON-006 (`compatibleModules` required) was previously downgraded from
`error` to `warning`, with this note already written into the rule file itself:

> "Downgraded from error to warning: compatibility pinning is frequently unknown at
> release-notes-generation time... blocking Markdown rendering on it forced operators through a
> manual gate on every run."

That is precisely what happened with `feature_docs`/`api_docs` in the real `1.0.0-alpha.1` run —
both were left blank in the generated `.md` (correctly, per CON-013's placeholder-text
prohibition) and required manual follow-up after the fact. CON-008 never received the same
downgrade CON-006 did, for what appears to be the same underlying reason.

### Evidence

- `rules/release-notes/release-notes-rule.yaml:65-70` — CON-006, `severity: warning`, with
  rationale note.
- `rules/release-notes/release-notes-rule.yaml:78-82` — CON-008, `severity: error`, no rationale
  note, no exception.
- `rules/release-notes/release-notes-rule.yaml:83-87` — CON-009, already `severity: warning` —
  no change needed here, already consistent with the CON-006 precedent.

### Related observation (not itself a bug)

The original `release-inputs.yaml` template (deleted in commit `fdcf08e`, "Course correction")
labeled `compatible_modules` as `# CON-006 REQUIRED` in its inline comment — even though CON-006's
actual rule severity was already `warning` (non-blocking) at the time. That comment was carried
forward verbatim into the current inline Step 0a config block in
`skills/release-notes/consumer-release-notes.skill.md`. This shows the "REQUIRED" labels in the
input config are decorative prose, not a live reflection of the rule file's actual severity, and
can drift out of sync with it — `rules/release-notes/release-notes-rule.yaml`'s `severity` field
is the only authoritative source for whether a field actually blocks a run.

### Options (not decided)

- Downgrade CON-008 to `severity: warning`, matching the CON-006 precedent and its exact rationale.
- Leave CON-008 as `error` if there's a reason feature/API docs should block publication in a way
  compatible-module pinning shouldn't — no such reason has been identified yet in this
  investigation.

---

## Decision

No fix applied. Both issues are logged for a future pass — revisit once there's appetite to touch
`schema.yaml`, `rendering.yaml`, the comprehensive example, and/or CON-008's severity together,
since Issue 1 and Issue 2 touch adjacent parts of the same files.
