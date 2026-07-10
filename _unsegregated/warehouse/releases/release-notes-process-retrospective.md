# Release Notes Generation — Process Retrospective

Case study: `/crn-iv` run that produced the `0.17.0-release-notes.*` files in this directory
(134 commits / 133 PRs, `v0.16.0..v0.17.0`). Recorded so the next run in this repo — or a
maintainer of the `/crn-iv` skill — can fix the slow parts instead of repeating them.

## Where the time went

### 1. Fetching PR data (biggest sink, ~15-20 min, should be ~1 min)

Commit subjects in this repo are terse JIRA-ticket-style strings (`[INJIVER-1523] Fixed LSH
comments - ...`), not conventional-commit messages. To write real descriptions instead of
paraphrasing subjects, GitHub PR bodies (mostly CodeRabbit auto-summaries) were needed for all
133 PRs in range. What actually happened:

1. Sequential `gh api` loop over all 133 PR numbers → hit the 3-minute tool timeout with
   **zero** output written.
2. Parallel fetch via `xargs -P8 -I{} sh -c '...'` → failed outright with
   `xargs: command line cannot be assembled, too long` (quoting bug — the inline `sh -c`
   command grew too large once `xargs` tried to substitute the shared variables).
3. Fixed the quoting by moving the fetch logic into a real script file, ran it in the
   background → succeeded, but 8 parallel processes appending to the **same output file**
   raced each other. Two responses got interleaved into one corrupted JSON line, silently
   losing 2 of 133 PR records.
4. Instead of just patching those 2 missing PRs immediately, two more "safer" background
   re-fetches were launched speculatively (a plain sequential loop, then a `mapfile`-array
   variant) — both hung indefinitely for reasons never diagnosed. The user had to ask what
   was going on before these were killed.
5. The fix that actually worked: reuse the 131 good records from attempt #3, and fetch just
   the 2 missing PR numbers with two direct one-off `gh api` calls (~3 seconds total).

**Root cause of the real delay:** step 4 — retrying blindly with new approaches instead of
diagnosing why the first background job was stuck, and not recognizing that 131/133 records
already existed and only 2 needed patching.

### 2. Noisy keyword-based commit classification (~3-4 min)

A first-pass classifier matched commit subject + PR body text against keyword lists from
`classification-rules.yaml` (e.g. `security`, `vulnerab`, `CVE`). This produced false
positives: Helm `values.yaml` changes and chart-version bumps got flagged as `securityUpdates`
because their PR bodies (or the underlying Kubernetes manifests) contain the field name
`securityContext`, which substring-matches `security`. This was caught by spot-checking PR
bodies, but the automated pass didn't save time — it added a wasted round trip before falling
back to manual review anyway, given how few commits in this repo follow clean conventional
prefixes.

### 3. WebFetch for standards files (~1-2 min)

The Step 1 manifest/schema/rendering files were first pulled with `WebFetch`, which runs page
content through a summarizing model rather than returning it verbatim. This produced a
paraphrase instead of exact YAML, and had to be redone with `curl` against the raw
`githubusercontent.com` URLs.

## Fixes for next time

| Problem | Fix |
|---|---|
| 133 sequential/parallel REST calls for PR bodies | Use `gh api graphql` with a single batched query (GraphQL can fetch dozens of PRs — title, body, labels — per request). Turns ~3 minutes of REST round-trips into a few seconds. |
| Parallel writes corrupting a shared output file | If parallel fetches are used at all, write each result to its own file (`pr-$n.json`) and `cat` them together afterward. Never have concurrent processes append to one shared file. |
| Retrying blindly instead of diagnosing a hang | If a background job produces zero output after ~10-15s, check `ps` / partial file state *before* launching a second "just in case" attempt. Two redundant background jobs were left running simultaneously and had to be killed by hand. |
| Re-fetching standards from the network every run | Cache `schema.yaml`, `rendering.yaml`, `human-rules.yaml`, `classification-rules.yaml` into `.github/standards/` in this repo. The skill's own fallback logic (Step 1/Step 2) already reads from that path if present — it's just never been populated here. |
| Naive keyword classification producing false positives | For repos like this one that don't use conventional-commit prefixes, skip the automated substring pass entirely and go straight to reading PR bodies (CodeRabbit summaries, if present, are reliable) for anything that looks feature/fix-worthy from the subject line. |
| `WebFetch` for machine-readable standards files | Always pull YAML/structured standards files with `curl`/raw content, not `WebFetch` — reserve `WebFetch` for genuinely prose content where a summary is acceptable. |

## Biggest-leverage changes

1. **GraphQL batching for PR data** — single highest-impact fix; eliminates the entire
   fetch saga above.
2. **Local `.github/standards/` cache** — removes network dependency and the WebFetch/curl
   fumbling on every run, not just this one.

Both are one-time setup costs that pay off on every future `/crn-iv` run in this repo.
