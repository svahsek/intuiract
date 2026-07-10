# GitHub Fundamentals for Release Notes — A Guide for Technical Writers

This is written for someone who needs to produce release notes from GitHub, not someone who
needs to write code. No prior git/GitHub engineering knowledge assumed. Read this once, then
keep it as a reference — you don't need to memorize any of it.

If a term below is unfamiliar later, jump to the [Glossary](#glossary) at the end.

## 1. The big picture: how a change becomes a release note

Everything in this guide is one path, start to finish:

```
Issue                PR                    Branch              Tag               Release
(what we want    →   (the actual      →   (where the      →   (a permanent   →   (tag + notes,
to do)                change, with         change lands)        pointer to        published)
                      description)                               one commit)
```

You, as a technical writer, mostly care about the **Issue** (intent — why this is happening)
and the **PR** (substance — what actually changed, usually with the best description). Branches
and tags are plumbing that gets you from "merged" to "here's a version number." You rarely need
to touch them directly, but you do need to know what they mean when someone mentions one.

## 2. Branches

A **branch** is a named line of development — a workspace where changes happen before they're
combined back into the main line.

- **Default branch** (usually called `main` or `master`): the "official" up-to-date line of
  code. In this repo it's `master`. Config files like `.github/release.yml` only take effect
  when they're on this branch — not on any other branch, no matter how current that other
  branch is.
- **Release branches** (optional): some projects cut a dedicated branch to stabilize one
  version before it ships — e.g. `release-0.18.x`. Once cut, most of the bug-fixing for that
  version happens there while `master` moves on to the next version's work. Not every project
  does this; check what this repo does by looking at the branch list.

**How to find "the branch for the upcoming release" without being an engineer:**
- Ask in your team's engineering channel — genuinely the fastest way, and normal to ask.
- Look at the repo's branch list (GitHub UI → the branch dropdown, or ask someone to run
  `git branch -a`) for anything named like `release-X.Y.x` or matching the version you're
  writing notes for.
- If nothing like that exists, the release is probably being cut straight from `master`.

You don't need to know how to *create* or *merge* branches for this work — only to recognize
which one your release notes should be based on.

## 3. Tags and Semantic Versioning (semver)

A **tag** is a permanent label on one specific point in history — think of it as a bookmark
that never moves, unlike a branch which keeps advancing. `v0.18.1` is a tag: it always points at
the exact same commit, forever.

Version numbers almost everywhere follow **semantic versioning**: `MAJOR.MINOR.PATCH`
(e.g. `0.18.1` = major `0`, minor `18`, patch `1`).

| Bump | Means | Example | Who decides |
|---|---|---|---|
| **PATCH** (last number) | Bug fixes only, nothing new, safe for anyone to adopt | `0.18.0` → `0.18.1` | Engineering — no new feature work went in |
| **MINOR** (middle number) | New features added, but nothing existing breaks | `0.18.1` → `0.19.0` | Engineering — at least one new capability shipped |
| **MAJOR** (first number) | Something breaks compatibility — old integrations may need changes | `0.x` → `1.0`, or `1.x` → `2.0` | Engineering — a deliberate, usually rare, decision |
| **Hotfix** | Not a formal semver tier — an org's own label for an urgent, out-of-cycle patch | varies by team convention | Whoever owns the incident |

**You will not be able to determine major/minor/patch purely by reading the code diff** — that's
a judgment call engineering makes (frequently while cutting the release). What you *can* do:
- If the milestone/label set includes anything marked "breaking" → it's at least a major bump.
- If it includes new features and nothing breaking → minor.
- If it's only bug fixes → patch.
- When genuinely unsure, ask — this one detail is worth a quick question rather than a guess,
  since it affects the whole framing of the release notes.

## 4. GitHub Releases

A **Release** is the thing readers actually see: it wraps one tag with a title, a description
(your release notes), and optionally attached files. Find it under the repo's **Releases** tab.

- **Draft**: saved but not visible to the public yet — safe to iterate on.
- **Pre-release**: marked as not-production-ready (betas, release candidates).
- **Published**: live, visible, usually triggers a notification to watchers.

## 5. Issues — "what we want to happen"

An **Issue** is a tracked unit of work: a bug report, a feature request, a task. Anyone (team
member or, on public repos, outside users) can open one. It has a title, a description,
comments, an assignee, labels, and can belong to a milestone.

The connection between "we said we'd do X" and "we actually did X" is a **closing keyword** in
a PR description — text like `Fixes #123` or `Closes #123`. When that PR merges, GitHub
auto-closes issue #123. This is the thread you'll follow to trace a shipped change back to its
original intent.

## 6. Labels — categorizing issues and PRs

A **Label** is a colored tag attached to an issue or PR — `bug`, `enhancement`, `security`, and
so on. Multiple labels can apply to the same item.

This matters directly to you: **`.github/release.yml` (the config I set up earlier) reads PR
labels to decide which section of the release notes a change belongs in.** You don't need to
design the label taxonomy yourself — but it helps to know it exists, and to be able to ask "are
we applying these consistently?" You can see every label currently defined in this repo without
touching git at all:

```
gh label list
```

or in the UI: Issues tab → Labels.

## 7. Milestones — "what's in this release"

A **Milestone** is a named bucket (e.g. `v0.19.0`) that issues and PRs get assigned to, with an
optional due date and a visible progress bar (`12 of 20 closed`). This is the direct GitHub
equivalent of what JIRA calls a "Fix Version."

**This is the single most useful thing for you once your team adopts it.** Once issues/PRs are
consistently assigned to a milestone, you can filter to exactly "what's planned or shipped for
this release" without understanding branches or tags at all:

- UI: Issues tab → Milestones → click the version.
- CLI: `gh issue list --milestone v0.19.0 --state closed`

## 8. Projects — the board view (optional, heavier)

A **GitHub Project** is a Kanban-style board (Todo / In Progress / Done, or custom columns) that
can pull in issues from multiple repos with custom fields. It's more flexible than a milestone,
but for release-notes purposes specifically, a milestone is usually all you need — Projects are
more of a day-to-day planning tool for the engineering team than a release-notes source.

## 9. Pull Requests — where the actual substance is

A **Pull Request (PR)** is a proposed code change with its own title and description. It's
almost always your richest raw material for writing — PR descriptions (especially the
auto-generated CodeRabbit summaries visible in this repo's PRs) usually explain *what changed
and why* far better than the terse commit messages underneath them.

## 10. How it all connects

Once your team is disciplined about labels and milestones, the full path looks like this:

```
Issue #123, labeled "type:feature", milestone "v0.19.0"
   → PR #456, description says "Closes #123", same labels applied
      → merged into master (or a release branch)
         → tag v0.19.0 cut at that point in history
            → GitHub Release drafted, notes auto-categorized by PR label via release.yml
```

Right now, this repo has the *mechanism* (labels exist, `.github/release.yml` is set up) but not
yet the *discipline* (PRs in the last release carried zero labels). That's a team habit to build,
not something you need to fix yourself.

## 11. Two layers of release-notes extraction (and how they fit together)

You mentioned you already built `intuiract` — a manifest/schema-driven registry that classifies
and produces structured release notes. Here's how that relates to `.github/release.yml`:

| | `.github/release.yml` (Layer 1) | `intuiract` + `/crn-iv` (Layer 2) |
|---|---|---|
| **What it needs** | PR labels only | Git history, PR bodies, optionally issues/milestones |
| **What it produces** | A flat, label-grouped list of merged PRs | Structured YAML with use cases, key capabilities, migration notes, security severity, confidence scoring, validation against style rules |
| **Setup cost** | One config file + label discipline | A schema + rendering + validation registry (already built) |
| **Best for** | A quick, always-available first draft the moment labels exist | The polished, publishable final document |

**Recommended order:** get Layer 1 clean first — it's cheap, it's already GitHub-native, and it
gives your team an instant "what merged since last time, roughly categorized" view with zero
extra tooling. Once labels and milestones are consistently applied, Layer 2 (`/crn-iv`) can
*consume* that already-categorized PR list as a trusted input instead of re-inferring categories
from raw commit history and PR prose the way it had to for the 0.17.0 release notes — that's
the part that took the most time in that run, precisely because nothing was labeled yet.

## 12. Your checklist: "what's in the next release?" (no engineering knowledge required)

1. Check the **Milestones** page for the upcoming version — shows a progress bar and the full
   issue list.
2. Search merged PRs for that milestone: `gh pr list --search "milestone:vX.Y.Z is:merged"`.
3. If you're unsure which **branch** the release is being cut from, ask — don't guess.
4. If you're unsure whether it's **major/minor/patch**, ask — don't guess. This one detail
   changes how the whole document should be framed (e.g. patch releases shouldn't list "new
   features" at all).
5. Read the PR descriptions of anything user-facing — that's almost always better written than
   the commit messages underneath them.

## Glossary

| Term | Plain-language meaning |
|---|---|
| **Repository (repo)** | The project's whole codebase + history, in one place |
| **Branch** | A named line of development; the default branch (`master`/`main`) is the official current state |
| **Commit** | One saved snapshot of changes, with a message describing it |
| **Tag** | A permanent bookmark on one specific commit, usually a version number like `v0.19.0` |
| **Pull Request (PR)** | A proposed change, with a title/description, reviewed and then merged |
| **Issue** | A tracked task/bug/request — the "why," separate from the "how" |
| **Label** | A colored category tag on an issue or PR (`bug`, `feature`, `security`...) |
| **Milestone** | A named bucket grouping issues/PRs into one target release |
| **Project** | A Kanban-style board, optional, heavier-weight than milestones |
| **Release** | The published wrapper around a tag: title + notes + optional files |
| **Semver** | The `MAJOR.MINOR.PATCH` version numbering convention |
| **Changelog** | The running list of what changed across versions (release notes, collectively) |
