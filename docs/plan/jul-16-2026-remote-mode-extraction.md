# Remote-Mode Extraction — Running the Consumer Skill Without a Local Clone

## What this document is

The release-notes consumer skill (`skills/release-notes/consumer-release-notes.skill.md`)
currently assumes it's invoked from inside a local, fully-cloned working tree of the target repo —
Step 3a's extraction is built entirely on local `git` commands. This document is the plan for a
second mode: pointing the skill at a public GitHub repo URL and running it from an unrelated local
directory, with no fork, no clone, no branch checkout — extracting everything through the GitHub
API instead.

This is a plan, not a build. Nothing here is implemented yet.

## Why this matters specifically for MOSIP/Inji

MOSIP and Inji are open-source projects with publicly accessible GitHub repos. That's precisely
what makes remote-mode extraction viable without any special access: every API call this plan
relies on works against a public repo with no write permissions and no fork required — the whole
point of the repo being public is that read access is already there for anyone, including a tool
that never touches a local checkout at all.

## The core problem

Step 3a's extraction today is built on local `git` commands:

```bash
git log {base_tag}..{target_ref} --pretty=format:"COMMIT|%H|%s|%an|%ad" --date=short --no-merges
git rev-parse --verify <release_ref>
git branch --show-current      # Step 0b auto-detect
git describe --tags --abbrev=0 # Step 0b auto-detect
```

All of these require a real, locally-cloned working tree with full history. Point the skill at
just a URL with no local clone, and none of them have anything to run against.

## What replaces each one

GitHub's REST/GraphQL API has a direct remote equivalent for almost everything already needed:

| Local git command (today) | Remote replacement | Notes |
|---|---|---|
| `git log {base_tag}..{target_ref}` | `gh api repos/{owner}/{repo}/compare/{base_tag}...{target_ref}` | One call returns the full commit list *and* the diff for every changed file (`files[].patch`) — strictly more capable than local `git log` alone, since Step 3d's targeted-diff step becomes free instead of a separate call |
| `git rev-parse --verify <ref>` | `gh api repos/{owner}/{repo}/commits/{ref}` (404 = doesn't exist) or `git ls-remote <url> <ref>` | Same validation, no clone; `ls-remote` is the lighter option for a plain existence check |
| `git branch --show-current` / `git describe --tags --abbrev=0` (Step 0b auto-detect) | Not applicable in remote mode — no local branch to read | Always fallback-only for convenience; in remote mode the operator fills `version`/`base_tag`/`release_ref` explicitly in Step 0a. `base_tag` *could* still auto-detect remotely via `gh api repos/{owner}/{repo}/tags`, if that convenience is worth keeping |
| `test -f docs/releases/po-notes/{version}.md` (Step 3c probe) | `gh api repos/{owner}/{repo}/contents/docs/releases/po-notes/{version}.md` | 404 = not found (same probe-and-skip behavior), 200 = fetch + base64-decode |
| `.github/release.yml` probe (Step 3b) | **No change needed** | Already implemented via `gh api repos/{owner}/{repo}/contents/...` — already remote |
| GraphQL PR/issue batch fetch (Step 3a) | **No change needed** | Already implemented via `gh api graphql` — already remote |

The actual surface area of change is narrower than it first looks: **only Step 0b's auto-detect
and the raw commit/diff-fetching part of Step 3a genuinely need rewriting.** Steps 1, 2, 4, 5, 6,
7, 8 (standards loading, classification, reconciliation, validation, rendering) don't care where
the evidence came from — they operate on whatever Step 3 handed them.

## Config addition needed

Step 0a would need a `github_repo: "inji/inji-verify"` field (owner/name), since without a local
`git remote -v` to read, there's no other way to know which repo to point the API calls at. This
exact field was already proposed once before, in the pre-session
`release-inputs-improvement-proposal.md` (see
[`_unsegregated/warehouse/releases/release-inputs-improvement-proposal.md`](../../_unsegregated/warehouse/releases/release-inputs-improvement-proposal.md)),
for almost the identical reason — "removes the need to infer this from `git remote -v`" — so this
isn't a new idea, just finally has a concrete use case.

## Where outputs land

Since the skill is no longer inside the target repo's own working tree, `release-artifacts/{version}/`
needs to be written into wherever the skill is actually being run from (the operator's own local
workspace, not the target repo) — which is simpler for this use case, since committing into
MOSIP's/Inji's repo directly was never the plan anyway. Output filenames should be prefixed with
the repo name once local-repo and target-repo aren't the same thing anymore, e.g.
`inji-verify/1.0.0-alpha.1-release-notes.yaml` instead of assuming a single implicit repo.

## Practical caveats (not blockers)

- **Use `gh api`, not raw unauthenticated `curl`.** Unauthenticated GitHub API calls cap at 60
  requests/hour; authenticated caps at 5,000/hour. A real release range easily uses more than 60
  calls once PR batching is counted.
- **The compare API truncates on very large diffs** (roughly 300 files). Real usage so far
  (134–140 commits for `1.0.0-alpha.1`) is comfortably under anything that would hit this, but
  worth knowing it exists if a release range ever gets very large.

## Open question

Whether to implement this as a "remote mode" toggle directly in
`skills/release-notes/consumer-release-notes.skill.md` — new `github_repo` field in Step 0a, and
Step 3a/3b/3c/3d branching on local-vs-remote — is not yet decided. Not started.
