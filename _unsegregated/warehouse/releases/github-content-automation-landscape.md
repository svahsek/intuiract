# What GitHub Is Doing for Content Automation — and Where `intuiract` Fits

You asked what GitHub itself is building for content automation — not just release notes — so
you can position `intuiract` correctly. Short version up front, detail below:

> **GitHub's automation stops at code-adjacent content** (commit messages, PR descriptions,
> changelogs, release notes). Nothing it ships or has on its public roadmap extends into
> **product documentation content types** — Capabilities/Features pages, Overview pages, User
> Guides, Deployment Guides. That gap is exactly where `intuiract` sits, and it's a genuine
> white space, not a place where you're rebuilding something GitHub already does.

I pulled this partly from live data (GitHub's public roadmap repo, `github/roadmap`, queried
just now) rather than only my training knowledge, since GitHub ships fast and my training data
has a cutoff — flagging that clearly in the caveat section at the end so you know what to
double-check yourself.

## 1. What GitHub already ships today (stable, shipped)

Grouped by what role each plays in a content-automation pipeline — this mirrors the same
shape `intuiract` is built around: **structured intake → extraction → generation → publishing**.

### Structured intake (capturing signal cleanly, at the source)
- **Issue Forms** — YAML-based issue templates with typed fields (dropdowns, checkboxes,
  required fields), not just free-text templates. This is GitHub's own bet that structured
  intake beats free-text mining — the same argument behind recommending a label taxonomy for
  this repo.
- **Labels + Milestones** — covered in the fundamentals guide; the metadata layer everything
  else reads.
- **PR/Issue templates** — free-text but structured-by-convention (e.g. Kubernetes'
  `release-note` block).

### Extraction & generation (turning signal into drafted content)
- **`.github/release.yml`** — auto-categorizes merged PRs by label into a changelog. Already
  covered in depth; this is GitHub's only *committed, config-driven* content-generation feature
  for a documentation-adjacent artifact.
- **Copilot-drafted PR descriptions** — when you open a PR, Copilot can draft the description
  from the diff. This is GitHub generalizing the same idea one level upstream: auto-draft
  content from a structured signal (the diff), not just categorize existing content.
- **Copilot Chat "explain this repo / this PR / this diff"** — ad hoc, on-demand summarization.
  Not a committed pipeline artifact (nothing gets saved anywhere automatically), but the same
  underlying capability.
- **Dependabot PR bodies** — a narrow but real example of fully automated, template-driven
  content generation (dependency name, version bump, changelog link, compatibility notes) with
  zero human authoring. Worth studying as a "fully closed loop" example even though it's
  single-purpose.

### Publishing & distribution
- **GitHub Pages** (including one-click deployment, now GA) — the usual home for a docs site
  built from a repo.
- **Wikis** — lighter-weight, less structured, generally not used for polished product docs.
- **Discussions** — community Q&A/announcements; a plausible downstream distribution channel
  for release-notes summaries, not a generation mechanism itself.

### The automation substrate underneath all of it
- **REST API, GraphQL API, Webhooks, GitHub Actions** — this is what `/crn-iv` and `intuiract`
  already build on. GitHub isn't hiding this layer; it's designed to be built on top of, which
  is exactly what you're doing.

## 2. What's active on GitHub's roadmap right now (pulled live)

Queried `github/roadmap` directly rather than relying on memory. Most relevant items, as of
this session:

- **Copilot coding agent / "Agent issue assignment and session management" [GA]** — you assign
  an *issue* to Copilot, it works autonomously and opens a PR. This is GitHub's clearest
  statement of direction: **issue in, structured artifact out, with minimal human authoring in
  between** — conceptually the same shape as what you're doing with release notes → downstream
  content types, just applied to code instead of docs.
- **GitHub Copilot for Jira [Public Preview]** — notable given your team's move *away* from
  JIRA: GitHub itself is building bridges into JIRA, not away from it. Worth knowing this
  exists, even though it doesn't change your team's direction.
- **Copilot Custom Models [GA]**, **Copilot Vision in Copilot Chat**, **more intelligent
  auto model selection** — all about improving the underlying generation quality, not new
  content-automation surface area.
- **Immutable Releases [GA/Preview]** — about tamper-proofing release artifacts (integrity,
  supply-chain security), not content generation. Mentioned only so you don't confuse it with a
  content feature if you see it mentioned elsewhere.

Nothing on the current public roadmap adds a documentation-content-type feature (guides,
overview pages, capability pages) beyond what's listed above. The pattern is consistent: GitHub
keeps generalizing "structured signal → auto-generated artifact" one code-adjacent content type
at a time (release notes in 2022, PR descriptions since, issue-to-PR agentic flow now) — but it
hasn't moved into product documentation itself.

## 3. Where this puts `intuiract`

Every industry pattern covered earlier in this conversation (GitHub's own `release.yml`,
release-drafter, Conventional Commits, Changesets, Kubernetes' release-note blocks) — and
everything on GitHub's own roadmap — stops at **the changelog/release-note boundary**. None of
them ask "now that we know what shipped, what else needs to change?" That's the specific idea
you described: use release notes as the canonical, already-classified signal, then propagate it
into Capabilities/Features pages, the Overview page, User Guides, Deployment Guides.

That's a legitimately different position from anything GitHub ships or is building, and
different from the third-party changelog tools too — they're all still solving "how do we
generate *one* content type (the changelog) well," not "how do we keep a whole documentation
surface area in sync with what shipped." Positioning `intuiract` as **the layer above the
changelog**, not a competitor to `release.yml` or Copilot, is accurate and defensible.

## 4. Caveat on currency, and what to watch yourself

My training data has a cutoff, and GitHub ships new Copilot/automation features frequently —
the roadmap query above is live as of this session, but by the time you're reading this it may
already be stale. Two things worth watching directly rather than relying on me:

- **`github.blog/changelog`** — GitHub's official day-to-day changelog of shipped features.
- **`github.com/github/roadmap`** — the public roadmap repo I queried above; browsable as
  GitHub Issues, filterable, and you can `gh api /repos/github/roadmap/issues` yourself the same
  way I just did if you want to re-check this periodically.
