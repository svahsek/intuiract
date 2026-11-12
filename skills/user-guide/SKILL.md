---
name: user-guide
description: "Markdown-first User Guide editorial model and remediation workflow."
---

# User Guide

## What This Content Type Does

User Guide improves existing Markdown documentation. It does not generate a guide from release
evidence, implement DITA/XML, or force a page into a fixed Diataxis layout.

DITA supplies topic-purpose discipline; Diataxis supplies the reader-intent compass; and The
Good Docs Project supplies practical authoring patterns. Intuiract applies those principles to
classify and remediate sections in place.

## Editorial Model

Classify each section by whether it informs action or cognition, and whether the reader is
acquiring knowledge or applying it:

| | Acquisition | Application |
|---|---|---|
| Action | Tutorial | Task / how-to |
| Cognition | Concept / explanation | Reference |

Troubleshooting is recovery-oriented action content and has its own symptom-to-resolution
contract. Mixed pages are valid; unclear transitions and low-confidence classification are what
require review.

## Workflow

1. Resolve the User Guide manifest and standards from a pinned revision.
2. Segment and classify the target Markdown page using `content_model`.
3. Emit a report that cites one rule for every finding.
4. Apply only deterministic, mechanically derived fixes. Require approval for all semantic or
   structural changes; block unsupported facts.
5. Re-run checks and report applied, deferred, and unresolved findings.

Use `review-and-remediate.skill.md` in consumer repositories.
