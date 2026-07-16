# Session Log — Release Notes Standards Review & Positioning

Compiled reference of decisions, findings, and open questions from a working session on the
`release-notes` content type's internal consistency, how bugs get tracked, and what this whole
project actually is. Organized by topic, not chronologically verbatim — each section carries
enough context to stand alone as a reference for future decisions.

---

## 1. Doc-links gap in a real run — the finding that started everything

A `/crn-iv`-style run in a consumer repo correctly left `[Feature Documentation]()` /
`[API Documentation]()` empty in the rendered `.md` rather than filling them with placeholder text
(CON-013 prohibits `TBD`/`TODO`/etc.) — required manual follow-up: fill the URLs in the `.md`, and
mirror them into `documentation.feature_docs`/`documentation.api_docs` in the `.yaml` so both stay
in sync.

While doing that, the run surfaced a claimed inconsistency: `rendering.yaml`'s template expects
`releasePage`/`upgradeGuide`/`troubleshootingGuide`/`apiReference`, while `CON-008`/`CON-009` and
the skill's config template use `feature_docs`/`api_docs`/`test_report`. The run went with the
latter (matched 2 files, was what was actually validated) and flagged it for upstream reporting.

**Initial recommendation given:** fix the field-name mismatch centrally in `intuiract` (it's a bug,
not a policy choice), and apply the same error→warning downgrade CON-006 already received to
CON-008, since the operational reason is identical ("forced operators through a manual gate on
every run").

---

## 2. The mismatch is deeper than "2 files vs 1" — verified against the actual repo

Checked directly rather than trusting the claim at face value. The real shape:

| Vocabulary A (schema-side) | Vocabulary B (rule/skill/evidence-side) |
|---|---|
| `releasePage` | *(no equivalent)* |
| `upgradeGuide` | *(no equivalent)* |
| `troubleshootingGuide` | *(no equivalent)* |
| `apiReference` | `api_docs` |
| *(no equivalent)* | `feature_docs` |
| *(no equivalent)* | `test_report` |
| *(no equivalent)* | `collab_guide` |

- **Vocabulary A** used by: `release-notes-schema.yaml` (declared source of truth), `rendering.yaml`,
  `examples/comprehensive-release-notes.yaml`.
- **Vocabulary B** used by: `rules/release-notes-rule.yaml` (CON-008/009), the consumer skill's
  Step 0a/Step 6, `source-precedence.yaml`, `release-evidence-fields.yaml`,
  `examples/release-evidence.sample.yaml`.
- No entry in `evidence-to-release-notes-mapping.yaml` bridges the two. `test_report` and
  `collab_guide` have **no field anywhere in the schema**, under any name — not a rename, a gap.
- **Real production evidence, twice**: the actual `1.0.0-alpha.1-release-notes.yaml` (in the
  `inji-verify` fork) and this week's run both independently produced Vocabulary B. Vocabulary A
  has never been produced by a real run.

**Conclusion:** Vocabulary A (`schema.yaml`/`rendering.yaml`) is the stale side, not Vocabulary B —
reverses the original "went with what's validated" framing. **Recommended fix direction:** adopt
Vocabulary B as canonical in `schema.yaml` + `rendering.yaml` + the comprehensive example.
**Status:** not applied — logged as GitHub Issue #1 (see §4).

---

## 3. Why CON-008 wasn't downgraded — and what "these were just addons" theory turned out to be

User's working theory: these fields were originally meant as optional add-ons in
`release-inputs.yaml`, and the inline-config refactor (moving inputs into the skill itself) may
have accidentally made them mandatory.

**Checked against git history** (`release-inputs.yaml` was deleted in commit `fdcf08e`, "Course
correction" — recovered the pre-deletion version): `feature_docs`/`api_docs` were **already**
labeled `# CON-008 REQUIRED` in the original file, before any inline-config refactor. So the
refactor didn't change requiredness — CON-008's `severity: error` always drove it.

**What the investigation actually found, more precisely:** the original file *also* labeled
`compatible_modules` as `# CON-006 REQUIRED`, even though CON-006 was already `severity: warning`
in the rule file at that point. That's proof the "REQUIRED" comments in input templates/config
blocks are decorative prose that can drift from the rule file's real severity — **the rule file's
`severity` field is the only authoritative source**, not the input template's comments.

**Refined conclusion:** not "these were never meant to be mandatory" — rather, **CON-006 already
answered this exact question for a sibling field (info often unavailable at generation time,
blocking rendering forces a manual gate every run) and CON-008 never got the same treatment.**
**Status:** not applied — logged as GitHub Issue #2 (see §4), decision explicitly deferred by user
("I will do nothing" for now).

---

## 4. GitHub Issues, not a `QA/` markdown folder — reasoning and outcome

Asked whether to track findings as GitHub Issues + Projects instead of markdown files.

**Recommendation given:** GitHub Issues, for a reason specific to this repo, not just general
practice — the entire release-notes/content-sync pipeline is architected around GitHub issues as
canonical evidence (`source-precedence.yaml`, `github_issue_number` linkage). Writing findings as
markdown works against that grain. Confirmed `intuiract` has Issues + Projects enabled and `gh` is
authenticated.

Initially created a `QA/release-notes-documentation-links-schema-mismatch.md` file with both
issues written up in full. User then asked: if using GitHub Issues, why keep the file at all?
**Agreed it's redundant** — two sources of truth for the same finding is exactly the kind of drift
this whole investigation was about. User is removing the `QA/` folder; GitHub Issues are now sole
source of truth for this class of finding.

**Filed:**
- **[Issue #1](https://github.com/svahsek/intuiract/issues/1)** — schema/rendering vocabulary
  mismatch (§2).
- **[Issue #2](https://github.com/svahsek/intuiract/issues/2)** — CON-008 severity precedent (§3).

---

## 5. `intuiract` itself has no version tags — adopting semver for the registry repo

Question evolved from "should `inji-verify`'s releases follow semver" (already effectively true —
`1.0.0-alpha.1` is a textbook-correct semver pre-release string, and the schema already cites
`semver.org` for version *format*) to the real question: **should `intuiract`, the manifest
registry itself, cut real semver-tagged releases.**

**Checked directly:** `git tag -l` returns nothing — zero tags exist. Yet three places in this
repo already assume tags exist or are coming:
- `standards/content-types/release-notes.yaml:44` — `"recommended_later: Pin to a release tag..."`
- `skills/release-notes/SKILL.md:197` — `"Until version tags exist, prefer pinning to a commit SHA"`
- `skills/release-notes/consumer-release-notes.skill.md:180` — `"pin to a commit SHA instead of main"`

Also found inconsistent, unsynchronized per-file `version:` fields across 6 files (`content-types.yaml`
0.1.0, `classification-rules.yaml` 1.0.0, `source-precedence.yaml` 1.0.0,
`evidence-to-release-notes-mapping.yaml` 1.0.0, `release-evidence-confidence-rule.yaml` **1.0** —
not even valid semver format, `references.yaml` 1.2.4) — none bumped together, none tied to any
repo-level tag.

**Recommendation:** repo-level git tags are the right mechanism (this repo is exactly the shape
semver is for — a registry other repos depend on, currently forcing consumers to choose between
tracking `main`, which breaks silently, or freezing on a commit SHA, which falls behind silently
with no signal). Draft a short versioning policy specific to a standards registry (e.g. removing a
required schema field = major, adding optional field = minor) before cutting the first tag.
Per-file `version:` fields should become internal/informational only, not the actual pinning
mechanism, once real tags exist.

**Filed:** **[Issue #3](https://github.com/svahsek/intuiract/issues/3)** — covers both the tagging
gap and the per-file version inconsistency, including the invalid `1.0` format on
`release-evidence-confidence-rule.yaml`.

---

## 6. What is `intuiract`, actually — positioning discussion

User asked directly: not building application software, so what is this, and is it on the right
path.

**Framing given — three overlapping patterns, none of them new individually:**
1. **Schema/contract registry** — same pattern as OpenAPI/JSON Schema/SchemaStore: define structure
   once centrally, many consumers validate against or generate from it.
2. **Shareable policy/lint config package** — same pattern as `eslint-config-airbnb` or Vale style
   packages: rules encoded as structured config, published once, with a local override mechanism.
3. **Agent runbook / skill library** — the newer, less-precedented piece: `SKILL.md` files are
   step-by-step procedures written for an LLM to execute, not data. Maps to the emerging `AGENTS.md`
   convention (already referenced in this repo's `CLAUDE.md`).

**Honest "right path" assessment:** the strongest signal isn't the design, it's that using it for
real (the `1.0.0-alpha.1` run, the content-sync placement tests) surfaced real bugs that got
tracked instead of ignored — evidence of a system being stress-tested, not just designed on paper.
**Real risk named plainly:** no existing ecosystem to borrow conventions from for free, since no
one else does this exact combination — conventions have to be invented as you go, which is exactly
why things like the field-name drift keep surfacing.

---

## 7. Which layer matters most, and should the 3 layers split into separate repos

**Layer priority — schema/contract layer (1) is the most important, clearly.** Evidence: the
field-name mismatch (§2) broke both the rules layer *and* the skill layer simultaneously, because
schema is the most upstream layer everything else depends on. A rules bug or a skill bug each break
one thing; a schema bug breaks everything downstream at once — same reasoning behind why breaking a
schema is specifically what should trigger a semver MAJOR bump.

**Should the 3 layers (schema, rules, skills) become 3 separate repos?** Direction is sound and
matches real precedent (OpenAPI spec repo vs. Spectral rulesets vs. generators; ESLint core vs.
`eslint-config-*`; Vale vs. Vale style packages) — but **recommended not yet**. Two concrete reasons:
no working repo-level versioning exists yet even within one repo (§5), and most real fixes so far
(like §2) still span all three layers in one coordinated commit — splitting before that's proven
turns every such fix into cross-repo PR choreography. Sequence: prove tagging discipline in one
repo first, observe whether the layers actually churn at different rates for this project
specifically, then split once that's observed rather than assumed.

**Layer 2 (rules) → can Vale absorb it?** Checked directly: `rules/release-notes-rule.yaml` already
declares `PRO-001`/`PRO-002`/`PRO-003` as `vale_style: ReleaseNotes/NoMarketing.yml` etc. — this is
**already the documented intent**, not a new idea. The actual gap: no `.vale.ini` and no
`styles/ReleaseNotes/` directory exist anywhere in the repo; the style files are referenced by name
but were never written. Vale can only take over the *prose* rules (`PRO-*`), not the structural
ones (`STR-*`, most `CON-*`, `SEC-001`/`003`) — those check YAML field presence/patterns *before*
rendering, which Vale (a text linter) has no mechanism to do; those need a real structural/schema
validator instead, which today doesn't exist either (the rule file is read and applied by an LLM
agent's judgment, not machine-executed).

**Bonus finding while checking this:** `SEC-002` ("No Comparative Marketing," hand-listing
`better than`/`unlike competitors`/etc.) appears to duplicate exactly what `PRO-001`'s Vale style
(`NoMarketing.yml`) is meant to check — once `PRO-001` is real, `SEC-002` is likely redundant and
could retire into it. **Not yet filed as an issue** — flagged in conversation only.

**Layer 3 (skills) → separate location, yes, lowest risk of the three.** Skills are already the
fastest-churning layer (rewritten twice in one session), carry the least blast radius if wrong
(don't corrupt the underlying contract the way a schema bug does), and the existing
`skills/<type>/SKILL.md` internal shape is already correct — only the repo-boundary question is
open, and there's no strong reason to wait on that one specifically.

---

## 8. Confirming the mental model — the 3-layer summary, corrected

User's own summary: "(1) OpenAPI-like spec for content types, (2) strict evidence-based rules
around them, (3) skills that extract → payload → render against spec and rules."

**Confirmed accurate, with two refinements:**
- The OpenAPI comparison is a *pattern* match (schema-first contract), not a scope match — OpenAPI
  describes API surface, this describes content payload shape. Same idea, different domain.
- "Rules... evidence based" is actually **two distinct mechanisms**, not one: (a) validation rules
  (checks a *finished* payload against the schema+policy — the ordinary part) and (b) evidence/
  precedence reconciliation (`source-precedence.yaml` + confidence scoring — resolves *conflicting*
  raw signals like issue vs. PR vs. commit into one trustworthy value). (b) is the genuinely novel
  piece — most schema+rules systems don't need it, because their input is hand-authored directly
  against the schema rather than derived from messy real-world signals.
- The skill isn't "extract, then render" — it orchestrates the *full* chain: extract → reconcile →
  generate against schema → validate against rules → render, refusing to skip or combine steps.

---

## 9. Precedent search — GoodDocsProject, DITA, and honest limits of what's known

**GoodDocsProject** — confirmed the user's own characterization: a curated library of Markdown
templates + writing guidance for a human writer to follow by hand. No machine-readable schema, no
validation, no evidence extraction. Same *spirit*, entirely different *mechanism*.

**Closest real prior art found: DITA** (Darwin Information Typing Architecture, OASIS standard,
originated at IBM, still used at IBM/Cisco/Adobe). Typed "information types" (concept/task/
reference — directly analogous to Release Notes/Deployment Guide/Overview), strict schemas per
type, a publishing pipeline that validates and renders. Decades of real enterprise precedent that
typed, schema-enforced content types scale. **The gap DITA never closed, and this project's actual
novel core:** DITA content is entirely hand-authored directly into structured XML — nothing in it
derives typed content from messy source-of-truth signals (git, GitHub, PO drafts) with confidence
scoring. That's the specific thing being built here that DITA doesn't do.

**Useful technique borrowed from DITA for scaling to more content types:** *specialization* — define
a base type with shared fields (product name, version, target audience, doc links — already
duplicated conceptually across the current schema), and have each content type's schema extend/
specialize that base instead of restating it, so tedium scales sub-linearly rather than linearly as
content types are added.

**On "does a leading company do this exact combination":** answered honestly — no confident
specific knowledge of one. Named as a real possibility that this is at least partly a *timing*
artifact: the specific combination (schema-typed content + evidence reconciliation from git/GitHub +
LLM-agent orchestration) has only been practically buildable since capable agentic LLMs existed,
which is very recent. Consistent with the earlier landscape research (GitHub's own automation stops
at the changelog boundary; nothing on their public roadmap goes further into product documentation).

---

## 10. Side topic — what "schema as contract" actually means, conceptually

Asked as a genuine side question, not tied to action items.

**Core concept:** a schema is a written contract about the shape of data, checkable by a machine,
so a producer and consumer can rely on it without coordinating directly or trusting blindly.
Traces to *Design by Contract* (Bertrand Meyer, Eiffel, 1980s) — originally about functions
(pre/postconditions), schemas are the data-shaped descendant of the same idea.

**Why it matters:** without a schema, "the shape of this data" only exists as tribal knowledge —
exactly the failure mode behind the Issue #1 finding (§2), where the contract existed only as
informal agreement between files and quietly diverged because nothing enforced it.

**Scale demonstration, using the user's own two examples:**
- **OpenAPI** — contract between two teams, usually within direct communication reach.
- **W3C Verifiable Credentials Data Model** — the same mechanism at a scale where direct
  coordination isn't even possible: an issuer and a verifier (e.g. Inji Verify) may never be aware
  of each other, and still interoperate correctly, purely because both independently implement the
  same public, versioned schema. This is why standards bodies exist — a neutral party to own and
  evolve the shared contract.

**Vocabulary to know:** *Interface Definition Language (IDL)* — the general category (OpenAPI, JSON
Schema, Protocol Buffers, GraphQL SDL, the W3C VC Data Model are all instances). *Schema-first*
(contract-first) development — writing the schema first and having everything derive from and
validate against it — versus *code-first*, where the schema (if any) is reverse-engineered after the
fact. Named plainly: this repo's current state is closer to code-first than contract-first for
exactly the reason Issue #1 exists.

---

## 11. Closing reflection — using this project to finally learn OpenAPI

User's observation: having delayed learning the OpenAPI spec for years as a senior technical
writer, building an "immature" analog of it themselves has now given them a concrete, motivated
reason to actually learn it properly.

**Response given:** validated as a genuinely sound learning strategy, not just a nice feeling — the
value is *earned intuition* for problems OpenAPI already solved, rather than reading solutions to
problems never personally felt. Specific connections drawn between things already hit this session
and what will click immediately when reading the real spec:

- `components/schemas` + `$ref` ↔ the DITA-specialization idea (§9) — reusable field definitions
  instead of redefinition per content type.
- Contract testing / linting in CI (e.g. Spectral) ↔ Issue #1 directly — OpenAPI's answer to "how
  do you stop the spec and reality from silently drifting apart," which is precisely what wasn't in
  place here.
- Spec versioning / breaking-change discipline ↔ Issue #3 (§5) — a mature, decades-refined answer
  to the same "what counts as breaking a contract" question already being worked through here.
- `required`/`enum`/`pattern` ↔ already hand-written in `release-notes-schema.yaml` — will read as
  recognition, not new syntax.

Named the general pattern: build a rough version of something yourself first, hit its real problems
firsthand, then read the mature spec — learning *why* it's shaped the way it is, not just *what* it
says.

---

## Open items, for future reference

| # | Item | Status |
|---|---|---|
| 1 | Schema/rendering vocabulary mismatch (§2) | [Issue #1](https://github.com/svahsek/intuiract/issues/1) — open |
| 2 | CON-008 severity vs. CON-006 precedent (§3) | [Issue #2](https://github.com/svahsek/intuiract/issues/2) — open |
| 3 | No repo-level semver tags on `intuiract`; inconsistent per-file versions (§5) | [Issue #3](https://github.com/svahsek/intuiract/issues/3) — open |
| 4 | `SEC-002` likely redundant with `PRO-001` once Vale styles are built (§7) | Not filed — flagged only |
| 5 | `.vale.ini` + `styles/ReleaseNotes/{NoMarketing,ConsistentTense,ProhibitedTerms}.yml` don't exist yet, despite being referenced by name in the rule file (§7) | Not filed — flagged only |
| 6 | Layer-split (schema/rules/skills into separate repos) — direction agreed, timing deferred until tagging (#3) is proven and layer churn rates are observed (§7) | Deferred by design |
| 7 | DITA-style specialization for shared base fields across future content types (§9) | Idea only, not started |
