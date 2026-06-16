## Pitch

Our framework is not just “a skill that diffs git and writes docs.” It is a thoughtfully designed, standards-driven documentation automation platform with these core strengths:

- **Centralized authority**: We built a registry/manifest architecture so consumers discover content types and validation rules from a single source of truth instead of hardcoded file paths.
- **Evidence-first release notes**: The release-notes skill is grounded in normalized release evidence, not prompt-based fluff. It reconciles issues, PRs, commits, QA, and tags into a canonical `release-evidence.yaml`.
- **Field-level precedence and trust**: We define explicit source precedence via `standards/source-precedence.yaml`, so every field has a deterministic authority chain and provenance.
- **Automated confidence scoring**: Every evidence item is scored, so the system knows what can be auto-published and what must be manually reviewed.
- **Validation-before-rendering**: We enforce schema rules, policy rules, and content checks before generating Markdown, preventing broken or risky output.
- **Reusable, composable content type model**: The `release-notes` skill is one example of a broader pattern; the same manifest/schema/rules/skill structure can be reused for features pages, overviews, API docs, and more.
- **Consumer-safe adoption**: A consumer repo can adopt the skill by fetching the manifest and entrypoints instead of copying standards — this makes upgrades safe and fast.
- **Documentation governance**: We built path conventions and architecture docs so standards authors and consumers stay aligned and avoid brittle internal references.

This is a management-ready narrative: we delivered a real documentation automation framework that is more than a “quick demo.” It is a disciplined, scalable system for safe, validated release documentation.

---

## 10-Slide Presentation Writeup

### Slide 1: Title
- Title: “Release Notes Automation Framework”
- Subtitle: “Standards-first documentation, built for reliable release publishing”
- Footnote: `intuiract` proof-of-concept

### Slide 2: Problem Statement
- Release notes are often manual, inconsistent, and error-prone
- Teams lose time reconciling issues, PRs, commits, and QA evidence
- Output quality drops when docs are generated without validation
- We needed a governed automation path, not ad hoc markdown generation

### Slide 3: Our Solution
- A standards-driven framework for release documentation
- Core capability: `release-notes` skill that:
  - extracts evidence
  - reconciles multiple sources
  - normalizes and scores items
  - validates structured output
  - renders publication-ready Markdown

### Slide 4: What Makes It Different
- Not just “git diff → doc”
- Built on:
  - `standards/content-types.yaml`
  - content type manifests
  - canonical schema and rendering rules
  - validation rules
  - human-readable skill/process documentation
- Outcome: deterministic, auditable, reusable

### Slide 5: Key Architecture
- Registry-first discovery: central `standards/content-types.yaml`
- Manifest resolution: `standards/content-types/release-notes.yaml`
- Canonical assets:
  - schema: `templates/release-notes/release-notes-schema.yaml`
  - rendering: `templates/release-notes/release-notes-rendering.yaml`
  - rules: `rules/release-notes/release-notes-rule.yaml`
  - skill: `skills/release-notes/SKILL.md`

### Slide 6: Evidence Workflow
- Phase 1: Extraction
  - issues, PRs, commits, QA results, scans
- Phase 2: Reconciliation
  - source precedence
  - conflict resolution
  - confidence scoring
- Phase 3: Content generation
  - map normalized evidence into release-notes schema
- Phase 4: Validation
  - schema + policy + confidence gate
- Phase 5: Rendering
  - Markdown output only after pass

### Slide 7: Governance & Safety
- Source precedence avoids guesswork
- Validation prevents bad releases
- Confidence thresholds enable safe automation
- Manual review only when needed
- Path conventions protect schema and rule references

### Slide 8: Benefits Delivered
- Faster, more reliable release note assembly
- Consistent structure across releases
- Better audit trail for why each release item exists
- Single authoritative standard for consumer repos
- Easier onboarding for new products/content types

### Slide 9: Why This Matters for Management
- Reduces release documentation risk
- Enables scaling from one skill to many content types
- Supports compliance and traceability
- Lowers cost of documentation creation and review
- Positions us to offer a reusable, upgrade-safe documentation standard

### Slide 10: Next Steps
- Demo the `release-notes` skill with a real release cycle
- Onboard one product team to the registry/manifest pattern
- Extend the framework to additional content types
- Track quality improvements and time saved
- Adopt the framework as the standard release documentation process

---

## Recommended Demo Narrative

1. Show the `release-notes` skill and its process from `skills/release-notes/SKILL.md`
2. Explain how the manifest architecture keeps content paths stable
3. Show the evidence reconciliation logic and confidence scoring
4. Show that validation is required before rendering
5. Illustrate how a consumer repo can adopt the skill without copying standards

> This pitch highlights that we are not selling a single tool; we are selling a standards-based documentation automation framework with strong governance, auditability, and extensibility.

If you want, I can also turn this into a polished slide deck outline with speaker notes for each slide.