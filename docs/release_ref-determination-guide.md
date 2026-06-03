# How to Determine release_ref for Your Repository

## Overview

`release_ref` is the Git branch or tag that represents the target release. It is used by the release-notes skill to compute the diff scope and extract evidence between `base_tag` and `release_ref`.

## Steps to Find Your release_ref

### 1. Check the Branches Page
- Navigate to: `https://github.com/{owner}/{repo}/branches`
- Look for release branches such as:
  - `release/*` (e.g., `release/0.14.0`)
  - `release-*` (e.g., `release-0.14.0`)
  - `develop` or `staging`
  - `main` (if releases go directly to main)

### 2. Check the Tags Page
- Navigate to: `https://github.com/{owner}/{repo}/tags`
- See if the project uses Git tags for release markers such as:
  - `v0.14.0`
  - `0.14.0`
  - `release-v0.14.0`

### 3. Check the Releases Page
- Navigate to: `https://github.com/{owner}/{repo}/releases`
- This shows the project's release history and can indicate the tagging strategy

### 4. Check the README or CONTRIBUTING Guide
- Look in `README.md` or `.github/CONTRIBUTING.md` for the project's branching or release strategy
- Many projects document their workflow explicitly

## Common Patterns

| Pattern | Example | When to Use |
|---------|---------|-------------|
| Release branch | `release/0.14.0` | Project uses dedicated release branches |
| Release tag | `v0.14.0` | Project uses Git tags as the single source of truth for releases |
| Main branch | `main` | Project releases directly to main; no separate staging |
| Develop branch | `develop` | Project uses a develop branch for integration before release |
| Semantic tag | `0.14.0` | Project tags releases without a v-prefix |

## For inji-verify Specifically

Since `inji-verify` is part of the Inji ecosystem, likely options are:

1. **Release branch pattern** (most common for Inji repos):
   - `release_ref: "release/0.14.0"`
   - This assumes the project creates a release branch for each version

2. **Git tag pattern**:
   - `release_ref: "v0.14.0"`
   - This assumes the project tags releases directly

3. **Main branch pattern**:
   - `release_ref: "main"`
   - This assumes all releases go directly to main

## How to Verify

Run this Git command to check what exists:

```bash
# Check if release branch exists
git branch -a | grep release

# Check if release tag exists
git tag | grep 0.14.0

# Check the history of main/release
git log --oneline main | head -5
git log --oneline release/0.14.0 2>/dev/null | head -5
```

## What to Enter in release-inputs.yaml

Once you determine your pattern:

```yaml
# If using release branch:
release_ref: "release/0.14.0"

# If using tag:
release_ref: "v0.14.0"

# If using main:
release_ref: "main"
```

Pair it with `base_tag` pointing to the previous release tag:

```yaml
base_tag: "v0.13.1"              # Previous release tag
release_ref: "release/0.14.0"    # Target ref for this release
```

The skill will then compute the diff as: `git log v0.13.1..release/0.14.0`

## Fallback Option

If you're unsure, you can leave `release_ref` blank and use the legacy `release_branch` field instead:

```yaml
release_ref: ""                  # Leave blank if uncertain
release_branch: "main"           # Fallback; skill will use this if release_ref is empty
```

However, this is less explicit and not recommended for production use.