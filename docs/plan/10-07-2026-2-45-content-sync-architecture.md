# Content Sync Architecture — From Release Signal to Downstream Docs

## What this document is

The [release-notes skill](../../.claude/skills/release-notes/skill.md) already solves the hard
problem: turning raw git/GitHub history into a trustworthy, confidence-scored signal —
`docs/releases/{version}-release-evidence.yaml`. This document is the plan for the *next* stage:
reusing that same signal to keep other content types current — **Deployment Guide, User Guide,
Features/Capabilities page, Overview page** — instead of re-deriving evidence from scratch for
each one.

This is a plan, not a build. Nothing here is implemented yet. Treat it as the thing to review,
argue with, and cut down before any of it becomes a skill file or a registry YAML.

**Plain-language note on two terms used throughout:** an *anchor* is a small marker left inside a
Markdown file so a program can find "this exact spot" again later, the same way a bookmark finds a
page. *Idempotent* means "safe to run twice" — running the sync again on unchanged evidence should
produce zero edits, not a duplicate paragraph.

---

## 1. Two problems, solved in order

Release notes are a single append-only document, regenerated fresh every release — there's nothing
to "find," you just write the whole file. The four new content types are different: they are
**living pages**, edited by writers between releases, and an update means finding the *one right
spot* inside an existing page without disturbing anything else on it.

That splits into two problems, and they have to be solved in this order:

1. **Targeting** — given one evidence item (e.g. "SD-JWT VC support added"), which content
   type(s) does it belong in, and which section of each?
2. **Placement** — given a target page, where *exactly* inside that Markdown file does the
   update go, on this run and every future run — not a fresh guess each time?

Placement is the harder problem and the one you asked about directly ("a mechanism to identify
places... before it will update them"). It's addressed first, in Section 3, because targeting
(Section 5) is meaningless until there's a reliable place to put the answer.

A third concern cuts across both: **these are pages a technical writer already owns.** An
automated write that clobbers hand-polished prose, or silently duplicates content on a rerun, is
worse than not automating at all. Section 7 makes the review gate explicit for this reason.

---

## 2. Where these content types actually live (checked, not assumed)

| Content type | Found in this repo? | Current form |
|---|---|---|
| Overview page | ✅ `docs/technical_docs/Inji_Verify_API_Overview.md` | Headings + an `## Index` section linking to `#anchor` slugs — already anchor-*shaped*, just not sync-aware |
| Deployment Guide | ⚠️ Partial — `deploy/README.md` is a thin script-runner doc; the substantive deployment prose lives externally at `docs.inji.io/readme/setup/deploy` | In-repo file is a poor sync target on its own |
| User Guide | ❌ Not found in this repo | Presumed to live on `docs.inji.io` or a separate docs/marketing repo |
| Features / Capabilities page | ❌ Not found in this repo | Same as above |

**This is an open question, not a detail to paper over:** half of the target surface is outside
`inji-verify`. A mechanism that only knows how to edit files in *this* repo can fully automate the
Overview page, can partially help with the Deployment Guide, and can only produce a *handoff
artifact* for the other two until it's confirmed where they live and who/what has write access
there. Section 8 designs for that split explicitly rather than assuming one repo.

---

## 3. Placement mechanism: how the agent finds "the right spot"

Two candidate mechanisms, and a recommendation.

### Mechanism A — Explicit sync anchors (HTML comments)

A marker pair wraps every agent-managed block:

```html
<!-- content-sync:begin id="feat.sd-jwt-vc-support" source-version="1.0.0-alpha.1" hash="sha256:9f2a..." -->
Inji Verify can now parse and verify SD-JWT format verifiable credentials...
<!-- content-sync:end id="feat.sd-jwt-vc-support" -->
```

- `id` — a **stable slug carried on the evidence item itself**, not tied to a release version, so
  the *same* feature getting a follow-up enhancement two releases later updates the *same* block
  instead of creating a duplicate one.
- `hash` — a content hash of the evidence fields that produced this block. On the next run, if the
  hash of the current evidence for `id` matches the stored hash, skip — nothing changed, don't
  touch the page. This is what makes reruns idempotent (see the "plain-language note" above) and
  what makes it safe to detect drift (see below).
- `source-version` — audit trail: which release last touched this block.
- Content **outside** any `content-sync:begin/end` pair is presumed human-authored and is
  permanently off-limits — the mechanism must never edit, reflow, or delete anything outside its
  own markers, full stop.

Pro: deterministic, exact, cheap to check. Con: requires a one-time "instrumentation" pass to add
markers to pages that don't have them yet — which is every target page today.

### Mechanism B — Structural / heading matching

No markers needed: find `## SD-JWT Support` (or something worded closely enough) by heading text
and insert near it. Zero retrofit cost, but fragile — a writer rewording a heading for clarity
silently breaks the match, and "closely enough" is exactly the kind of fuzzy judgment call that
produces wrong-spot edits on a doc someone else owns.

### Recommendation — self-instrumenting hybrid

Use B only to *bootstrap* A, then never rely on B again for that block:

1. First time a given `id` needs a home: try an exact anchor match (A). Not found → fall back to
   heading/structural match (B) to locate the right section.
2. Once placed via either path, **write the anchor pair around it**. Every subsequent run for that
   `id` uses exact-match A only.
3. If a run later expects an anchor and it's gone (a writer edited around it, or removed it during
   a rewrite), treat that as a signal the page changed out from under the sync, not a bug to
   silently paper over — fall back to B once, and flag the result for human confirmation rather
   than auto-inserting. A page a writer is actively reworking is exactly the case where a silent
   auto-insert does the most damage.

This mirrors a decision already made elsewhere in this repo's standards: the release-notes skill
treats a missing PO draft or missing `.github/release.yml` as "expected steady state, not
degraded" and falls through cleanly (see `skill.md` Steps 3b/3c). The same posture applies here —
missing anchors on a first run are normal, not an error condition.

---

## 4. The content-type registry (the manifest)

A single file, `.github/standards/content-types.yaml`, analogous to the central manifest the
release-notes skill fetches in its Step 1 — one entry per content type, everything else resolved
from it rather than hardcoded per-skill:

```yaml
content_types:
  overview_page:
    target:
      kind: local_file
      path: docs/technical_docs/Inji_Verify_API_Overview.md
    section_map:
      apiChanges.newApis: { placement: table_row_append, anchor_prefix: "api." }
      apiChanges.deprecatedApis: { placement: table_row_strike, anchor_prefix: "api." }
      newFeatures: { placement: prose_section, anchor_prefix: "feat.", condition: "affects supported formats/endpoints" }
    review_policy: suggest_only     # see Section 7 for the trust ramp this belongs to
    confidence_floor: 70

  deployment_guide:
    target:
      kind: external          # see Section 8 — not writable directly yet
      note: "Substantive content lives at docs.inji.io; deploy/README.md in this repo is a thin script pointer only"
    section_map:
      technicalImprovements.infrastructure: { placement: prose_section, anchor_prefix: "infra." }
      enhancedFeatures: { placement: prose_section, anchor_prefix: "infra.", condition: "changes deployment steps or prerequisites" }
    review_policy: handoff_report_only
    confidence_floor: 80

  user_guide:
    target:
      kind: unresolved         # blocked on Section 2's open question
    review_policy: handoff_report_only
    confidence_floor: 80

  features_page:
    target:
      kind: unresolved         # blocked on Section 2's open question
    review_policy: handoff_report_only
    confidence_floor: 75
```

`section_map` keys are the same `change_type` categories the release-evidence contract already
produces (`newFeatures`, `enhancedFeatures`, `apiChanges`, `technicalImprovements`, `deprecations`,
etc.) — no new vocabulary, just a routing table on top of the existing one. `placement` names a
small fixed set of strategies (`table_row_append`, `table_row_strike`, `prose_section`,
`list_item_append`) so the mechanism stays deterministic instead of improvising per item.

---

## 5. Routing: which evidence goes to which content type

| Evidence category | Overview page | Deployment Guide | User Guide | Features page |
|---|---|---|---|---|
| `newFeatures` | ✅ if it changes supported formats/endpoints | — | ✅ if it changes a user-facing workflow | ✅ always |
| `enhancedFeatures` | ✅ section update | ✅ if it changes deploy steps/prerequisites | ✅ behavior change | ✅ update existing entry |
| `apiChanges` (new/modified/deprecated) | ✅ table row add/update/strike | — | — | — |
| `technicalImprovements.infrastructure` | — | ✅ | — | — |
| `deprecations` | ✅ flag, never silent removal | ✅ if deploy-relevant | ✅ flag | ✅ flag |
| `bugFixes` / `knownIssues` | — | — | **manual-only**, see note | — |
| `securityUpdates` | — | ✅ if it changes a deploy/config step | **manual-only** | — |

**Why bug fixes are manual-only, not auto-propagated:** a bug fix can mean a User Guide's
documented workaround is now obsolete — but deciding *how* to reword prose that describes a
workaround away is a judgment call, not a mechanical field mapping. This mechanism should surface
"this fix may make section X stale" as a flag for the writer, and never auto-edit prose on that
basis. Auto-writing is reserved for additive, structurally mappable changes (new table row, new
list entry) — not for rewriting existing sentences.

---

## 6. Update algorithm (runs once per release, per content type)

1. Load the already-produced `{version}-release-evidence.yaml` — no re-extraction, no new git
   mining. This whole mechanism is a consumer of that file, not a second producer. Read from its
   relocated path (`release-artifacts/{version}/`, not `docs/releases/` — see Section 10)
   once that relocation lands.
2. For each evidence item at or above the content type's `confidence_floor` (Section 4), resolve
   candidate `(content_type, section)` pairs via the routing table (Section 5).
3. For each candidate, locate the anchor (Section 3's hybrid):
   - **Anchor found, hash matches** → no-op. This is the idempotent path and should be the common
     case on reruns.
   - **Anchor found, hash differs** → the evidence for this `id` changed since last sync (e.g. a
     feature described in one release got a correction in the next) → render a diff, route through
     `review_policy`.
   - **No anchor, heading match found** → first-time sync → render the new block, insert it,
     write the anchor pair around it, route through `review_policy`.
   - **No anchor, no heading match** → cannot place automatically → do not guess. Emit to the
     manual-placement list (Section 6b) instead.
4. Never touch anything outside a `content-sync:begin/end` pair.
5. Produce one consolidated artifact per release, mirroring the existing
   `{version}-validation-report.txt` pattern:

   `release-artifacts/{version}/content-sync-report.md` (relocated out of `docs/` for the
   same reason as the release-notes pipeline's own internal artifacts — see Section 10), containing:
   - Auto-applicable changes (by content type, with rendered diff)
   - Items sent to manual placement (no anchor/heading match found)
   - Items flagged manual-only by category (bug fixes/known issues, Section 5)
   - Items destined for external/unresolved targets (Section 8)

This keeps a single reviewable file per release — the same shape the release-notes pipeline
already gives you for the release notes themselves, so nothing new to learn about *how* to review
it, just a new file to look at.

---

## 7. Review gate — a trust ramp, not a one-time switch

Because these are pages a writer actively maintains (unlike release notes, which are generated
fresh each time), auto-applying changes from day one is the wrong default.

- **Phase 1 — bootstrap (comment-only-suggest for everything).** The agent never commits or opens
  a PR. It only produces the content-sync report from Step 5 above. This is also the phase where
  the one-time anchor-retrofit on existing pages happens, under direct supervision, so a human is
  watching the very first placements land in the right spot.
- **Phase 2 — draft PRs, still human-merged.** Once anchors exist on a page and hash-based
  idempotency has held up cleanly across a couple of real releases (i.e., reruns produced zero
  spurious diffs), allow the mechanism to open a PR with the proposed changes — never to push
  directly, never to merge. *A PR is just a proposed change sitting on a branch; nothing becomes
  real until a human clicks merge.* This should be per content type, not global — the Overview
  page (in-repo, already anchor-shaped) is a reasonable candidate to reach Phase 2 well before
  anything targeting an external repo does.
- **No Phase 3 auto-merge is proposed here.** If that's ever wanted, it's a separate decision to
  make explicitly later, not a default this document should set.

---

## 8. Cross-repo handling (the open question from Section 2)

Two of the four target content types don't live in `inji-verify`. Until that's confirmed, the
mechanism should be designed so the *compute* half and the *apply* half are separable:

- **Compute** ("what should change, and where") is fully automatable today, entirely from files
  already in this repo (`docs/releases/*-release-evidence.yaml`). This is what Section 6 builds.
- **Apply** ("actually write it into the target page") requires either write access to wherever
  User Guide / Features page / the substantive Deployment Guide content live, or a human to carry
  the handoff artifact over manually.

Recommendation: build compute first, and for `unresolved`/`external` targets (Section 4's registry
entries), stop at the content-sync report — a structured, ready-to-paste set of proposed edits —
rather than reaching across a repo boundary automatically. Once the target repo is confirmed and
someone with access there wants automated PRs into it, that's an explicit extension of the
`target.kind` field in the registry (`local_file` → `external_repo`), not a redesign.

---

## 9. Rollout plan

1. **Confirm target locations** for User Guide and Features/Capabilities page (Section 2). The
   routing table (Section 5) and registry (Section 4) can't be finished for those two until this
   is answered.
2. **Retrofit anchors** into the two known in-repo/partial targets — `Inji_Verify_API_Overview.md`
   and `deploy/README.md` — as a supervised, one-time pass (Phase 1 of Section 7).
3. **Build `.github/standards/content-types.yaml`** using the schema drafted in Section 4.
4. **Relocate release-notes pipeline artifacts** out of `docs/releases/` into
   `release-artifacts/{version}/` (Section 10), before the sibling skill in the next step
   is built against them — fix the path convention once, now, rather than migrating it after a
   skill and a report format already depend on the old one.
5. **Add a sibling skill**, e.g. `.claude/skills/content-sync/skill.md`, that runs *after*
   release notes are generated, consumes `{version}-release-evidence.yaml` from its relocated path,
   and implements Section 6 — starting in `suggest_only` mode for every content type, no exceptions.
6. **Run it once per real release**, review every suggestion by hand, and use the false
   positives/negatives to correct the routing table in Section 5 — it's a first draft, not a
   spec to build the skill against blindly.
7. **Only then** consider moving any individual content type from Phase 1 to Phase 2 in Section 7,
   one at a time, starting with the Overview page.
8. **Only after step 7 has held cleanly across real releases**, evaluate building the feature-level
   manual trigger described in Section 11. Deliberately last, not concurrent with the rest of this
   rollout — see Section 11 for why.

---

## 10. Pipeline Artifact Hygiene — Keep Machine Output Out of `docs/`

Not part of content-sync's own placement logic, but a prerequisite this document depends on
getting right, since Section 6 both *reads* and *writes* machine-internal artifacts alongside it.

**The problem, as it exists today:** the release-notes pipeline's `output_dir` default
(`docs/releases/`) puts four files in the same folder — `{version}-release-notes.md` (the
rendered, reader-facing document) alongside three machine-internal artifacts:
`{version}-release-evidence.yaml`, `{version}-release-notes.yaml` (structured pre-render source),
and `{version}-validation-report.txt`. Section 6, as originally drafted, compounded this by
defaulting content-sync's own `{version}-content-sync-report.md` to the same folder too.

**Only the rendered `.md` belongs under `docs/`.** It's the one file in that list a reader should
ever land on. The other three, plus content-sync's report, are pipeline bookkeeping — audit trail
for a machine, not content for a person — and belong somewhere that says so.

**Why this matters more now than it would have on day one:** `release-evidence.yaml` and
`release-notes.yaml` stopped being "release-notes' own output that happens to sit in `docs/`" the
moment this document made them content-sync's primary input (Section 6, step 1). A file read by
two separate pipelines is a shared signal, not a doc-adjacent scratch file.

**Revised layout:**

```text
docs/
  releases/
    {version}-release-notes.md              # the only reader-facing file — stays here

.github/
  release-artifacts/
    {version}/
      release-evidence.yaml                 # relocated
      release-notes.yaml                    # relocated
      validation-report.txt                 # relocated
      content-sync-report.md                # relocated (Section 6, step 5)
```

**Migration cost: effectively zero, if done now.** `1.0.0-alpha.1` is the first real output this
pipeline has ever produced — fixing the convention before a second or third release accumulates
under the old path is free; doing it after is not.

**Out of scope for this document, but required as a follow-up:** `output_dir` in
`skills/release-notes/consumer-release-notes.skill.md`'s Step 0a configuration block currently
defaults to `docs/releases/` for all four files, undifferentiated. That default needs to split
into two — one path for the rendered `.md`, one for the three internal artifacts — as a change to
that file, not this one.

---

## 11. Trigger Granularity — Release-Level Now, Feature-Level Later (Deliberately Gradual)

Every trigger specified so far in this document — Section 6's algorithm, Section 9's rollout —
runs **once per release**, driven by a finished `{version}-release-evidence.yaml` covering a full
`base_tag..target_ref` range. That is the only trigger being built right now.

**The gap this leaves:** feature-completion and release cadence aren't the same rhythm. A feature
can be done, merged, and stable well before a release is cut — but under the release-level trigger
alone, its placeholder in `overview.md` or `features.md` sits unfilled until the next release
pipeline runs, even though nothing about *placing* it actually required waiting that long.

**The future direction — explicitly not being built yet:** a second, manual entrypoint that takes
one completed issue/PR instead of a full commit range, runs it through the same precedence,
classification, and confidence logic the release-level path already uses, produces a single
evidence item, and hands it to the *exact same* Section 6 algorithm — same registry, same anchors,
same `review_policy` gate. This is a narrower on-ramp into the mechanism that already exists, not
a second, lighter-weight system running in parallel with different rules.

**Why this is deliberately sequenced after, not alongside, the rest of this document's rollout:**
the risk with "sync it the moment it's done" is that urgency becomes the reason to skip exactly
the steps that keep an automated write from clobbering a page a writer owns — confidence scoring,
precedence resolution, the suggest-only review gate. This should only be built once the
release-level path has held up cleanly across real releases, ideally with at least one content
type already promoted past Phase 1 in Section 7's trust ramp. Building the faster trigger before
the underlying placement engine is trusted just gives a shaky mechanism a shorter fuse.

**Still not full automation, even then.** The feature-level trigger is human-invoked — "this
feature is done, sync it now" — not a webhook firing automatically off a merge event. Whether it
ever becomes that is a separate, later decision this document isn't making.
