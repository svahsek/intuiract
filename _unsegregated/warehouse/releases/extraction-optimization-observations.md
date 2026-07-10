# Extraction Optimization — Observations

> **Revision note (superseded in part):** the first pass of this document over-weighted PO
> release notes and `.github/release.yml` as primary sources, and mischaracterized
> `release-inputs.yaml` using a draft (`release-inputs.v2.yaml`, this folder) instead of the
> template actually in use
> ([templates/release-notes/examples/release-inputs.yaml](../../../templates/release-notes/examples/release-inputs.yaml),
> which already has a working `evidence_source` block with `milestone`, `include_issue_labels`,
> `include_pr_labels`, `issue_query`, `pr_query` — that part was never broken). §3-§6 below are
> corrected accordingly: PO notes and `.github/release.yml` are now framed strictly as optional,
> conditional accelerants that the extraction path must work identically without. The corrected
> design is implemented in
> [`skills/release-notes/consumer-release-notes.skill.md`](../../../skills/release-notes/consumer-release-notes.skill.md) —
> treat that file as the concrete answer; this document is the reasoning behind it.

Deep-dive requested after the Inji Verify `/crn-iv` run (`0.17.0-release-notes.*` in this
directory): 134 commits / 133 PRs, ~20 minutes, ~30K tokens. This is an observations document,
not an implementation — three things were asked for: (1) understand why that run cost what it
cost and how to cut it, (2) fold extraction logic into the skill itself so it stops depending on
`release-inputs.yaml`, (3) evaluate two new high-confidence sources — PO-written release notes,
and a `release.yml`-style GitHub-native mechanism — as the *first* things read, before any git
mining starts.

This builds directly on three documents already sitting in this folder
(`release-notes-process-retrospective.md`, `release-inputs-improvement-proposal.md`,
`how-mature-organizations-release.md`) plus `docs/reports/release-input-improvent-scope.md`.
Rather than repeat their findings, this synthesizes them into one architecture change and
extends it with the two new sources. Read those first if you want the raw evidence; this is the
"so what do we actually build" layer on top.

---

## 1. Why the run cost what it cost — the real root cause

Every prior document diagnosed a symptom. Underneath all of them is one structural problem:

**The skill classifies and describes every PR in range, then discovers most of them get
filtered out at publication.** The 0.17.0 run mined all 133 PRs — fetched bodies, ran keyword
classification, cross-checked labels — and the validation report shows only **29 items actually
got published** (3 newFeatures + 2 enhancedFeatures + 17 bugFixes + 3 securityUpdates +
4 technicalImprovements). ~95 commits were internal (chore/ci/test/docs) and discarded. So
roughly **78% of the fetch-and-classify work was thrown away** — not because the fetching was
slow (though it was, see the retrospective's GraphQL-vs-REST saga), but because the funnel was
inverted: mine everything, then filter, instead of filter first, then mine only what survives.

The retrospective's fixes (GraphQL batching, avoid `WebFetch` for YAML, cache standards locally,
don't retry blindly) are all real and worth keeping — they make the 133-item fetch faster. But
they don't address the 78% waste. That requires shrinking the 133 down to something close to 29
*before* any per-PR reading happens. That's what both new sources below are actually for.

---

## 2. Reframing extraction as a funnel, not a flat pass

Proposed shape — three tiers, each one narrowing what the next tier has to touch. Tier 0 is
**conditional, checked cheaply and skipped silently when absent** — Tier 1 is the one tier that
must be assumed present on every run, since it's the only one built entirely from things git and
GitHub guarantee to have (commits, PRs, labels-if-any):

```
Tier 0 — Pre-classified, IF present (optional accelerant)  (seconds, ~0 tokens/item)
  ├─ PO-written release notes file — used if found, silently skipped if not
  └─ GitHub native categorization (.github/release.yml + generate-notes API) — same

Tier 1 — Structured GitHub metadata, batched, ALWAYS runs   (one call, low tokens)
  └─ GraphQL: PR numbers from commit log, batch-fetched title/body/labels/linked-issues —
     this is the resilient core; works identically whether or not Tier 0 found anything

Tier 2 — Targeted mining, gap-fill only                      (proportional to what's left)
  └─ PR body / diff reads — ONLY for items that will actually appear in a published section
     and don't already have a usable description from Tier 0/1
```

Today's skill has no Tier 0 and effectively does Tier 2 for everything, because nothing upstream
told it what could be skipped. Tier 0, when it happens to be available, shrinks Tier 2's input
set further; Tier 1 alone (structural classification before any body is read) already shrinks it
substantially on its own, and is the part that has to carry every run where Tier 0 is empty —
which, per the org's current state, is the common case, not the edge case.

---

## 3. New source #1 — PO-written release notes (conditional accelerant)

### What it is, and why it must stay optional

A PO (or whoever owns release communication) sometimes knows, in advance, what shipped and why
it matters. When that draft exists, it's a shortcut for content the skill otherwise has to
*invent* in Phase 3 ("Agent writes: benefit-driven... no marketing language"). But per direct
feedback: this will exist for some releases and not others — "sometimes and sometimes not atal"
— and the intent is to reduce reliance on it over time, not build toward it. So the extraction
path **must produce full-quality output with zero change in behavior when no PO draft exists.**
That's the default case to design for, not the fallback case.

Correction from the first pass of this document: this is not a "primary source" in the sense of
something the pipeline expects or degrades without. It's a probe-and-skip accelerant — checked
once, used if found, silently ignored if not, with no warning and no confidence penalty either
way. It's a replacement for the agent's own *inference step* for narrative fields when available,
never a replacement for git/GitHub evidence, which remains the only thing the system can count on
being there every time.

### What it should and shouldn't be trusted for

| Field | Trust PO notes? | Why |
|---|---|---|
| `summary.headline`, `summary.paragraphs`, `keyBenefits` | Yes — primary source | This is exactly the voice/content these fields need; today the agent fabricates it |
| Candidate `newFeatures` / `enhancedFeatures` titles + descriptions | Yes — primary source, but still requires an evidence link | PO knows what shipped and why it matters; still needs a PR/issue reference for CON-002 |
| `shipped` status | **No** | Tag boundary is the only source of truth per `source-precedence.yaml` — a PO draft written before the tag is cut may describe things that slip. Never let it override `shipped_included_in_release`. |
| `migrationRequired`, `migrationNotes`, severity, CVE details | **No** — verify from git/PR evidence | POs write for impact, not technical precision; a missing migration note here is a compliance risk (CON-005), not just a style issue |
| Issue/PR references (CON-002) | **No** — must still resolve to a real `#1234` | PO prose rarely cites ticket numbers precisely |

### Format and location

Recommend a lightweight structured file, not free prose the agent has to parse loosely —
mirrors the `operator_input` block that already exists in `SKILL.md` Phase 3, so a PO can
directly author close-to-final content instead of the agent reverse-engineering structure from
paragraphs:

```
docs/releases/po-notes/{version}.md
```

```yaml
---
headline: ""             # PO writes directly — becomes summary.headline candidate
target_audience: []
key_benefits: []
---

## Feature notes (PO draft — agent verifies against evidence, adds refs/migration notes)

- Feature: <what PO calls it>
  Why it matters: <PO's framing>
  Related: <PR/issue link if PO knows it, else agent resolves>
```

### Risk to flag explicitly

A PO draft can absolutely fail `PRO-001` (no superlatives) or `SEC-002` (no comparative
marketing language) — this is expected and fine, because **Phase 4 validation already gates
this regardless of source**. PO notes shortcut the *drafting* step, not the *validation* step.
Nothing about trusting this source requires relaxing any existing rule.

---

## 4. New source #2 — GitHub-native `release.yml` categorization (conditional accelerant)

### Clarifying the actual mechanism

"release.yml on master, the way GitBook automates it" maps to a real, already-shipped GitHub
feature (documented in `github-content-automation-landscape.md` and
`how-mature-organizations-release.md`, but not yet connected to a concrete API call in either):

1. A `.github/release.yml` config file on the default branch maps **PR labels** to changelog
   categories:
   ```yaml
   changelog:
     categories:
       - title: "New Features"
         labels: ["type:feature"]
       - title: "Bug Fixes"
         labels: ["type:bug"]
       - title: "Security"
         labels: ["type:security"]
   ```
2. One REST call generates the categorized body for a range, using that config:
   ```
   POST /repos/{owner}/{repo}/releases/generate-notes
   { "tag_name": "v0.18.0", "previous_tag_name": "v0.17.0", "target_commitish": "master" }
   ```
   Response: already-categorized Markdown (grouped by the labels above), full contributor list,
   and a compare link — **one HTTP call**, not 133.

Where this is genuinely useful, even before label discipline exists: the fetch-mechanics win is
real on its own — one call returns every PR in range, which is strictly better than any
per-PR-number loop regardless of whether the result is categorized. Treat it as an optional
alternative *input* into 3a's batching step, not as a system to build a dependency on.

### The honest caveat — do not build toward this; our own system is the reliable one

The retrospective confirmed all 133 PRs in the 0.17.0 range had **empty label arrays**. Without
labels, `generate-notes` returns every PR in one call but everything lands in a single
uncategorized "Other Changes" bucket — no classification value at all. And per direct feedback:
`.github/release.yml` on master is something to *push for* over time, will exist for some
releases and not others, and **GitHub label/taxonomy discipline is not there yet and isn't
guaranteed to arrive.** The existing `source-precedence.yaml` → `classification-rules.yaml`
chain (issue labels → PR labels → PR title prefix → commit prefix → keyword match) is the system
that has to carry classification quality on its own, indefinitely if necessary — it is more
tensile than depending on a config file and label discipline this org doesn't reliably have.
`release.yml`/`generate-notes` is worth calling when it's there, purely as a batching shortcut;
it is not the classification strategy, and the skill must never assume it, warn on its absence,
or read as degraded when it's missing.

### Precedence framing — this is a mechanics win, not a new truth source

Important distinction for `source-precedence.yaml`: `generate-notes` output is *computed from*
`github_pr_labels`, which is already `change_type`'s fallback #2. It isn't new evidence — it's
the same evidence, pre-aggregated by GitHub in one call instead of N. So this doesn't need a new
precedence tier; it needs the skill's fetch step to *call the aggregate endpoint first* and only
fall back to per-PR label reads for anything `generate-notes` couldn't categorize (no label
applied).

---

## 5. What moves into the skill vs. what `release-inputs.yaml` keeps

**Correction from the first pass:** this section originally compared against
`release-inputs.v2.yaml` (an unadopted draft sitting in this folder) rather than the template
actually in use —
[`templates/release-notes/examples/release-inputs.yaml`](../../../templates/release-notes/examples/release-inputs.yaml).
That template already has a working `evidence_source` block (`milestone`,
`include_issue_labels`, `include_pr_labels`, `issue_query`, `pr_query`) — those scope selectors
are legitimate, already correct, and **should not be removed or shrunk.** The recommendation to
gut the file was wrong; withdrawn.

`SKILL.md`'s existing rule — *"Do not embed extraction or precedence logic in the skill. Fetch
those from standards files"* — is about **field-level precedence** (which source wins for a
given field), and that boundary stays exactly where it is: `source-precedence.yaml` remains the
single authority for that. What actually needs to move is narrower than originally proposed:
**fetch mechanics** (batch vs. sequential, one call vs. many, what to skip before reading it) is
not precedence logic and was never something `release-inputs.yaml` declared in the first
place — the retrospective's problems (sequential REST loop, `xargs` quoting bug, racing parallel
writes) were the skill improvising fetch mechanics live, with nothing in the input file that
could have prevented it either way. So this isn't "move fields out of the input file" — it's
"stop leaving fetch mechanics undefined and let the skill improvise them under time pressure."
That's now written directly into the procedure (`consumer-release-notes.skill.md`, Step 3a)
as non-negotiable behavior, independent of anything in `release-inputs.yaml`.

The two conditional accelerants (§3, §4) plug in alongside the existing input file, not instead
of it: `release-inputs.yaml` gains one new optional field (`po_notes_path`, defaulting to a
conventional path) and nothing else changes about its shape.

---

## 6. Proposed `source-precedence.yaml` additions

Two changes, both additive (no existing field's precedence order changes):

```yaml
field_precedence:
  # New fields — currently undefined in source-precedence.yaml, handled ad hoc in SKILL.md
  # Phase 3 as "agent writes." Formalizing these closes that gap.
  summary_headline:
    primary: "po_release_notes"
    fallback:
      - "generated_from_evidence"   # today's behavior — agent drafts from evidence, as last resort

  summary_key_benefits:
    primary: "po_release_notes"
    fallback:
      - "generated_from_evidence"

  # Existing field — no order change, just documents that generate-notes is a faster
  # computation of the same fallback #2 (github_pr_labels), not a new source.
  change_type:
    primary: "github_issue_type_or_labels"
    fallback:
      - "github_pr_labels"              # computed in bulk via generate-notes when release.yml exists
      - "github_pr_title_prefix"
      - "commit_message_prefix"
```

And in `confidence_by_source`:

```yaml
confidence_by_source:
  po_release_notes: 80        # human-authored, high trust for narrative — below official
                               # GitHub fields (85-95) because it isn't independently verifiable
                               # against a ticket/PR the way labels are
  github_release_yml_bulk: 90 # same trust as github_pr_labels (90) — same underlying data,
                               # fetched in bulk instead of per-PR
```

---

## 7. Projected impact

Using the 0.17.0 run's own numbers as the baseline:

| | Before (0.17.0 run) | After (proposed) |
|---|---|---|
| PR-classification calls | 133 individual `gh api` calls (plus 2 failed background-job attempts) | 1 `generate-notes` call |
| PR bodies actually read for descriptions | ~133 (mined before filtering) | ~29 (only published items — PO notes may cover some of these already) |
| Time | ~20 min, ~15-20 min of which was the PR-fetch saga alone | Low single-digit minutes — dominated by Tier 2 reads for the ~29 surviving items, not fetch mechanics |
| Tokens | ~30K | Rough order-of-magnitude cut proportional to the 78% of Tier-2 reads eliminated, plus removal of the failed-attempt retries that inflated the original run |
| `release-inputs.yaml` failure modes | Duplicate-key bug, wrong path caused real detours this run | Same file, same fields — fixed by a pre-flight duplicate-key check and near-miss path glob at Step 0 (both now in `consumer-release-notes.skill.md`), not by removing fields |

These are directional, not measured — the only way to confirm them is a real run once this is
implemented. Also worth noting: this projection assumes Tier 0 is empty (no PO draft, no
effective `release.yml`) — i.e. it's the worst-case, no-accelerant number, since that's the
state most releases will actually run in. Any Tier 0 hit only improves on this baseline.

---

## 8. Open questions for you, before this becomes an implementation plan

1. **PO notes format** — is the lightweight YAML-frontmatter-plus-prose template in §3 workable
   for whoever the PO is on Inji Verify, or does it need to be plainer (just headings, no
   frontmatter) because they won't hand-write YAML?
2. **Where do PO notes get written relative to the release cut?** If they're written *before* the
   tag exists, the skill needs to treat every PO-sourced feature claim as "candidate, pending
   shipped-status verification" — worth confirming that's an acceptable process (PO drafts early,
   agent reconciles against the tag at generation time) rather than PO notes only existing after
   the fact, which would defeat the point.
3. **`po_notes_path` field** — is `docs/releases/po-notes/{version}.md` the right convention, or
   should this live somewhere else in the consumer repo layout?
4. **Label/`release.yml` adoption timing** — is this something you're actively pushing for now,
   or is it far enough out that the skill shouldn't reference it as more than a dormant probe for
   the time being?

`skills/release-notes/consumer-release-notes.skill.md` already implements the corrected
design (§3-§6). Remaining open items if you want this to propagate further:
`skills/release-notes/SKILL.md` (central orchestration doc) and
`skills/release-notes/consumer-release-notes-inji-verify.skill.md` (the Inji-Verify-specific
copy actually used for `/crn-iv`) still reflect the old flat-extraction approach and would need
the same Step 3 rewrite if you want them to match. `standards/source-precedence.yaml` would need
the additive fields from §6 if you want `po_release_notes` formally recognized as a source rather
than handled ad hoc in the skill's own prose.
