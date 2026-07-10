# How Mature Organizations Extract Release-Note Data from GitHub

Companion to [`release-inputs-improvement-proposal.md`](release-inputs-improvement-proposal.md).
That document mitigates the current, unlabeled state of this repo's history. This one describes
what the actual end state looks like elsewhere, and what it would take to get there.

## The core idea

This project's 0.17.0 run spent most of its time mining meaning out of 134 loosely-structured
commits and 133 PR bodies **after the fact** — reading CodeRabbit auto-summaries, checking which
"security" hits were real vulnerabilities vs. a Kubernetes `securityContext` field, cross-
referencing JIRA-style ticket prefixes that don't follow any enforced grammar. Every approach
below exists to avoid exactly that kind of after-the-fact reconstruction. The common thread:
**push classification to the moment the change is authored**, when a human already has full
context, instead of reconstructing it later from prose.

## The four dominant patterns

### 1. Structured metadata at PR-merge time, not mined after the fact

GitHub's own [native "automatically generated release notes"](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes)
feature reads a `.github/release.yml` config that maps **PR labels** to changelog categories
(e.g. label `bug` → "Bug Fixes" section, label `enhancement` → "Enhancements"). This is a
built-in GitHub feature, not a third-party tool — turned on by adding one config file.

[release-drafter](https://github.com/release-drafter/release-drafter) — one of the most widely
adopted GitHub Actions for this — does the same categorization continuously: it maintains a
*draft* release note that updates as each labeled PR merges, so by the day of release the draft
is already ~90% written. There is no "sit down and mine 130 commits" step at all.

**Requirement:** consistently applied PR labels. This repo currently has zero PR labels in use
(confirmed empirically during the 0.17.0 run — every one of the 133 PRs in scope had an empty
labels array).

### 2. Conventional Commits, enforced by CI, not inferred

Projects like Angular, and tools like [`semantic-release`](https://semantic-release.gitbook.io/)
and Google's [`release-please`](https://github.com/googleapis/release-please), require every
commit or PR title to match a strict grammar: `feat:`, `fix:`, `feat!:` / `BREAKING CHANGE:` for
breaking changes, `chore:`, `docs:`, etc. A CI bot (`commitlint` is the standard tool) blocks
any merge that doesn't comply.

Classification then becomes a **deterministic parse** — split on `:`, look up the prefix in a
table — instead of a keyword guess over free text. That distinction is exactly what caused this
run's false positive: a naive substring match on `"security"` matched the Kubernetes manifest
field `securityContext` inside a Helm `values.yaml` PR body, flagging an unrelated infra change
as a security update. A conventional-commit parse can't make that mistake because it only reads
a fixed-position prefix, never free text.

This repo's commit history (`[INJIVER-1523] Fixed LSH comments - added clientIdScheme to
data-types...`) shows no such enforced convention today — prefixes are JIRA ticket keys, not a
parseable change-type grammar.

### 3. Author-written release-note snippets, shipped with the PR

Kubernetes requires a `release-note` block inside every PR description (parsed by a dedicated
`release-notes-generator` tool); if the block literally says `NONE`, the PR is excluded from the
changelog entirely — an explicit, author-controlled include/exclude decision, not an inference.

The [Changesets](https://github.com/changesets/changesets) approach (used across the JS
ecosystem — Chakra UI, Remix, and many others) has each PR add a small markdown file describing
the change and its semver bump (patch/minor/major), merged as part of the PR itself. A release
tool then just concatenates the changeset files for everything in the release.

Both approaches move the writing burden to the person with the most context — the PR author, at
PR time — instead of an agent or maintainer reverse-engineering intent from a CodeRabbit
auto-summary months after the fact, which is what this run had to do for descriptions like "SDK
Package Migration to @injistack/react-inji-verify-sdk."

### 4. Milestones as the scoping mechanism (replaces JIRA fixVersion)

Once an org is fully on GitHub Issues, a **milestone** (e.g. `v0.18.0`) attached to every issue
and PR in scope is the direct equivalent of a JIRA fixVersion field:

```bash
gh issue list --milestone v0.18.0 --state closed
gh pr list --search "milestone:v0.18.0 is:merged"
```

Both give an authoritative, complete "what's in this release" list without any git-range
diffing or commit-message mining. This is the field this repo's `/crn-iv` runs should adopt in
place of `jira_project_key` / `jira_fix_version` — see `release-inputs.v2.yaml`'s
`evidence_source.github_milestone`.

## Comparison

| Approach | What it requires | What it replaces here | Effort to adopt |
|---|---|---|---|
| PR labels + `.github/release.yml` / release-drafter | Contributors apply labels at PR time | The manual/keyword classification pass in Step 4 of `/crn-iv` | Low — one config file + label discipline |
| Conventional Commits + `commitlint` CI gate | PR titles follow a fixed grammar, enforced automatically | Free-text keyword matching against commit subjects | Medium — needs a CI check and contributor buy-in |
| Author-written release-note block/changeset | A required PR-template field, checked in review | Reading PR bodies and inferring intent after merge | Medium — needs a PR template change + review discipline |
| Milestones for release scoping | Every issue/PR tagged with a milestone before merge | JIRA fixVersion queries | Low — GitHub milestones are already a built-in feature |

## Net effect on `/crn-iv` if adopted

If this repo adopted PR-label classification (pattern 1) and milestone-based scoping
(pattern 4), Steps 3-4 of `/crn-iv` (collect git data, reconcile evidence, classify) would
collapse from "read up to 133 PR bodies and use judgment" into "group already-labeled,
already-milestoned issues/PRs by label" — a GraphQL query and a `groupby`, not an inference
exercise. Adding pattern 2 (enforced commit/PR-title grammar) would remove the last source of
classification ambiguity even for changes that never get a label applied.

The input-file changes proposed in `release-inputs-improvement-proposal.md` are the best
available mitigation **given the current, unlabeled state of this repo's history**. This
document describes the actual fix.

## Suggested adoption order for this repo

1. **Enable GitHub's native auto-generated release notes** (`.github/release.yml`) — lowest
   effort, immediate partial value even before labels are consistently applied.
2. **Adopt a small label taxonomy** on PRs (`type:feature`, `type:bug`, `type:security`,
   `type:breaking`, `type:chore`) — matches the `label_taxonomy` already sketched in
   `release-inputs.v2.yaml`.
3. **Start using milestones** for release scoping instead of ad hoc git tag ranges.
4. **(Optional, higher effort)** Enforce a commit/PR-title convention via CI once the team is
   comfortable with labels — this is the only step that fully removes classification ambiguity,
   including for unlabeled or mislabeled PRs.
