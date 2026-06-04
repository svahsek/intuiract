---
name: known-issues-table-fallback
description: Planned fallback for known issues rendering if table formatting breaks with long descriptions
metadata:
  type: project
---

If the 5-column known issues table in `templates/release-notes/release-notes-rendering.yaml` (ISS-001) causes formatting problems due to long description text, fall back to a 2-column table: **Issue | Description** only.

Change `issueTemplate` to: `| {title} | {description} |`
Change table header in `template` to: `| Issue | Description |` / `|-------|-------------|`

**Why:** Deferred until after real-repo testing. Multi-sentence descriptions collapse badly in markdown tables.
**How to apply:** User will return to this after testing the current 5-column layout on a real repo.
