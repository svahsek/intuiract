# Git as a Documentation Source of Truth

Every software product lives in git, and git contains far more structured documentation signal than most teams realize. The problem isn't extraction — it's knowing what to look for and how to classify it.

---

## The Git Data Model (What Exists and Where)

Git has two layers: **core git objects** and **platform extensions**.

### Layer 1 — Core Git Objects (universal, no platform dependency)

| Object | Contains | Extraction Command |
|---|---|---|
| **Commit** | message, author, timestamp, parent hash | `git log --format="%H %s %an %ae %ad"` |
| **Diff / Patch** | exact lines added/removed per file | `git diff <tag1>..<tag2>` |
| **Tree** | directory snapshot at a point in time | `git ls-tree <sha>` |
| **Tag** | named pointer to a commit (version markers) | `git tag --sort=-version:refname` |
| **Branch** | named pointer to the latest commit in a line | `git branch -a` |
| **Reflog** | history of where HEAD has been | `git reflog` |

### Layer 2 — Platform Extensions (GitHub / GitLab / Bitbucket)

| Artifact | Contains | Extraction Method |
|---|---|---|
| **Pull Request** | title, body, labels, linked issues, reviewers, merge commit | GitHub REST / GraphQL API |
| **Issue** | title, body, labels, milestone, assignee, comments | GitHub REST / GraphQL API |
| **Milestone** | grouped issues → maps to a release | GitHub Milestones API |
| **Review comments** | in-line code rationale | GitHub PR Review API |
| **Checks / CI runs** | what tests passed/failed | GitHub Checks API |
| **Deployment events** | when it was deployed, to where | GitHub Deployments API |

---

## The Software Development Lifecycle in Git

Every stage of development leaves a traceable artifact:

```
Idea → Issue
  ↓
Planning → Milestone (groups issues into a release scope)
  ↓
Development → Branch name  (feat/oauth-client-management)
  ↓
Code → Commits  (feat: add OAuth2 client creation endpoint)
  ↓
Review → PR body (describes the change, links issues, rationale)
  ↓
Merge → Merge commit (preserves PR link in message)
  ↓
Release → Tag  (v2.1.0)
  ↓
Deploy → Deployment event (environment, timestamp)
```

Each arrow is extractable. Together, they give you the full story of a release.

---

## What Is Extractable Per Documentation Type

### Release Notes

```bash
# All commits between two release tags
git log v2.0.0..v2.1.0 --format="%H|%s|%an|%ad|%b"

# Merged PRs in that range (via merge commits)
git log v2.0.0..v2.1.0 --merges --format="%s"
# e.g. "Merge pull request #234 from user/feat/oauth"
# → extract PR number → GitHub API for full PR body + labels

# Files changed
git diff --name-status v2.0.0..v2.1.0

# Stats: lines added/removed per file
git diff --stat v2.0.0..v2.1.0
```

From this you get: features, bugs, security patches, API changes, performance work, breaking changes.

---

### Changelog (CHANGELOG.md)

Same as release notes but flatter — one line per commit, grouped by type. Tools like `git-cliff` and `conventional-changelog` do this already using the **Conventional Commits** spec.

---

### API Documentation (Delta)

```bash
# Find all files where API routes changed
git diff v2.0.0..v2.1.0 -- "**/*.routes.*" "**/*.controller.*" "**/openapi.yaml"
```

New files in `routes/` = new endpoints.  
Changed files in `routes/` = modified endpoints.  
Deleted lines with `@Deprecated` = deprecated endpoints.

---

### Migration Guide (auto-triggered)

```bash
# Find commits with BREAKING CHANGE in body
git log v2.0.0..v2.1.0 --format="%B" | grep -A5 "BREAKING CHANGE"
```

Conventional commits spec requires `BREAKING CHANGE:` in the commit footer. This is the trigger for generating a migration guide section — which `CON-005` enforces.

---

### Dependency / CVE Bulletin

```bash
# Find changes to dependency files
git diff v2.0.0..v2.1.0 -- package.json package-lock.json pom.xml build.gradle

# Find removed/added packages
git diff v2.0.0..v2.1.0 -- "**/requirements.txt"
```

Cross-reference removed/added packages against CVE databases → auto-generate security update entries.

---

### Contributor / Credit Section

```bash
git shortlog v2.0.0..v2.1.0 --summary --numbered --email
```

---

## The Conventional Commits Standard — The Critical Enabler

This is what makes automated classification work. The spec defines a commit message format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Types that map directly to the release notes schema categories:**

| Conventional Commit Type | Schema Category |
|---|---|
| `feat:` | `newFeatures` |
| `fix:` | `bugFixes` |
| `perf:` | `technicalImprovements.performance` |
| `refactor:` | `technicalImprovements` |
| `security:` *(unofficial but common)* | `securityUpdates` |
| `deprecate:` *(unofficial)* | `deprecations` |
| `BREAKING CHANGE:` in footer | triggers `CON-005` migration requirement |
| `chore:`, `test:`, `ci:`, `build:` | `internal` (excluded from output) |

The `classification-rules.yaml` is a superset of this standard — adding keyword matching for teams that don't use conventional commits.

---

## The Full Automated Documentation Pipeline

```
1. TRIGGER
   Tag push (v2.1.0) or manual run or PR merge to main

2. RESOLVE SCOPE
   git describe --tags --abbrev=0  → finds previous tag (v2.0.0)
   git log v2.0.0..v2.1.0          → all commits in range

3. ENRICH (platform layer)
   For each merge commit → extract PR number → GitHub API → PR title, body, labels, linked issues

4. CLASSIFY
   classification-rules.yaml + classification-overrides.yaml
   → each commit/PR → category (newFeatures, bugFixes, etc.)

5. STRUCTURE
   Build YAML payload matching release-notes-schema.yaml
   Populate fields from commit + PR data

6. VALIDATE
   STR-*, CON-*, SEC-* rules → report violations with fix suggestions

7. RENDER
   YAML → Markdown via release-notes-rendering.yaml
   HDR-*, FEA-*, BUG-* rules

8. OUTPUT
   docs/releases/v2.1.0/release-notes.yaml  (source of truth)
   docs/releases/v2.1.0/release-notes.md    (published)
   docs/releases/v2.1.0/validation-report.md (audit trail)
```

---

## What Cannot Be Fully Automated (Requires Human Judgment)

| Gap | Why | Solution |
|---|---|---|
| **Business context** | Git doesn't know why something matters to users | `summary.headline` and `paragraphs` fields in `release-inputs.yaml` |
| **Quantified impact** | `perf:` commit says "optimize query" but not "40% faster" | `IMP-001` rule prompts for metrics |
| **Target audience framing** | Same feature means different things to API consumers vs. admins | `targetAudience` in `release-inputs.yaml` |
| **Known issues** | Discovered post-merge, rarely in commits | Manual addition or issue label `known-issue` |
| **Marketing framing** | Stakeholders want narrative, not commit messages | `PRO-001` Vale rule prevents over-marketing; human writes headline |

This is exactly why the hybrid input model (`release-inputs.yaml` → auto-detect → prompts) is the right architecture. Machines extract the structure; humans provide the judgment layer.

---

## Where the Consumer Skill Sits in This Pipeline

The `consumer-release-notes.skill.md` handles **steps 4–8** of the full pipeline above. The gap is **steps 1–3** — specifically the GitHub API enrichment layer:

- Git log extraction (step 2) ✅
- Commit classification (step 4) ✅
- **GitHub API enrichment for PR bodies/labels (step 3)** — not yet implemented

PR bodies often contain the richest documentation signal:
- The "why" behind a change (not in commits)
- Screenshots, before/after behavior descriptions
- Links to the original issue (which has acceptance criteria)
- QA notes

This is why the `features.template.md` in `unsegregated/` matters — a structured PR body template makes step 3 machine-extractable.

---

## Related Files

| File | Role |
|---|---|
| `standards/classification-rules.yaml` | Central baseline for commit → category mapping |
| `templates/release-notes/examples/classification-overrides.yaml` | Consumer-specific keyword/prefix overrides |
| `.github/skills/release-notes/consumer-release-notes.skill.md` | Full orchestration skill (steps 4–8) |
| `templates/release-notes/examples/release-inputs.yaml` | Human judgment layer (business context) |
| `templates/release-notes/release-notes-schema.yaml` | YAML payload structure |
| `unsegregated/features.template.md` | PR body template (enables step 3 enrichment) |
