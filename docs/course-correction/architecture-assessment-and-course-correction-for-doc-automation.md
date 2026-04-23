# Architecture Assessment and Course Correction for Documentation Automation

Your current direction is good, but it is not yet the end-state architecture.

You already have a strong standards-first content pipeline. That is better than what many teams have because you have explicit schema, validation, rendering rules, and a controlled input contract.

## Current approach: strengths and limits

Your current architecture is:

1. Inputs
2. Extraction
3. Classification
4. YAML structure
5. Validation
6. Rendering

That is solid for release notes and maybe one or two adjacent content types. The risk is that as you add Overview, Features, User Guide, Installation Guide, API Changes, QA evidence, and GitHub-native planning data, the skill becomes a large orchestration prompt with too many source-specific rules in one place.

That usually creates four roadblocks:

1. Source precedence becomes hard to reason about.
2. The same fact gets recomputed differently for different content types.
3. Validation happens too late, after narrative is already generated.
4. You get prompt sprawl instead of a stable documentation platform.

## Better architectures to compare against

### 1. Evidence Model First

This is the recommended direction if you want to go far.

Flow:

1. Extract normalized release evidence from Issues, Projects, branches, PRs, commits, tags, and QA runs.
2. Reconcile into one canonical model.
3. Run confidence and validation on that model.
4. Generate multiple content types from the same model.

This means the primary artifact is not release-notes YAML. It is something like release-evidence.yaml or docs-events.yaml.

Why it is better:

1. One source of truth for all downstream docs.
2. Easier precedence handling.
3. Easier debugging when docs are wrong.
4. Easier to add new content types without rewriting extraction.

This is the architecture closest to where mature teams end up.

### 2. Docs Fragment Architecture

This is what teams like Python and OpenStack effectively do for release notes.

Flow:

1. Every change contributes a small structured fragment.
2. Fragments are reviewed near the code change.
3. Release assembly composes fragments into outputs.

Why it works:

1. High accuracy.
2. Low ambiguity at release time.
3. Very strong auditability.

Why it may not be enough for your goal:

1. It works best for release notes and changelogs.
2. It does not naturally solve Overview, User Guide, or Installation Guide unless the fragment taxonomy is heavily extended.

This is a very safe architecture, but narrower than your ambition.

### 3. Knowledge Graph or Entity Graph Architecture

This is the more advanced variant of evidence-first.

Flow:

1. Model entities such as feature, issue, PR, commit, tag, test case, module, API surface, and doc page.
2. Create edges between them.
3. Generate content from graph queries and templates.

Why it is powerful:

1. Best for cross-page consistency.
2. Best for answering what changed and where it should appear.
3. Best long-term if you want document intelligence across many products.

Why it may be too much now:

1. Higher setup cost.
2. Harder to operationalize early.
3. Easy to over-engineer before taxonomy is stable.

Unless there is platform investment, this is not the starting point.

### 4. Retrieval plus LLM Summarization Pipeline

Flow:

1. Gather all raw artifacts.
2. Retrieve relevant pieces for each page type.
3. Ask the LLM to synthesize.

Why teams try it:

1. Fast to prototype.
2. Flexible across content types.

Why it is dangerous as a core architecture:

1. Weak determinism.
2. Hard to audit precedence.
3. More hallucination and inconsistency risk.
4. Harder to enforce policy at scale.

This is useful as a fallback layer, not as the foundation.

## Recommendation

Do not throw away the current architecture. Course-correct it one level down.

Keep:

1. Manifest-driven standards
2. Schema validation
3. Rendering rules
4. Content-type-specific outputs

Change:

1. Move extraction and reconciliation into a separate canonical evidence layer.
2. Make content generators consume the evidence layer, not raw git extraction directly.
3. Keep the release-notes skill as one consumer of that model.

This is the cleanest evolution path.

## What better than current looks like

A better architecture for this use case is:

1. Source adapters
   1. GitHub Issues and Projects
   2. PRs
   3. Commits
   4. Branches
   5. Release tags
   6. QA test evidence
2. Canonical evidence model
   1. Release item
   2. Type
   3. Scope
   4. Inclusion status
   5. Evidence links
   6. QA status
   7. Confidence
   8. Affected doc areas
3. Reconciliation engine
   1. Precedence rules by field
   2. Conflict rules
   3. Missing-evidence warnings
4. Content generators
   1. Release notes
   2. Features page
   3. Overview page
   4. User guide deltas
   5. Installation guide deltas
   6. API changes
5. Shared validators
   1. Structure
   2. Policy
   3. Security
   4. Completeness
   5. Cross-page consistency

## Where you stand relative to top teams

Relative to public, mature teams:

1. You are ahead on standards rigor.
   1. Most public examples have fragments or release-note automation, but fewer have explicit human rules, rendering rules, schema, and structured validation this clearly designed.
2. You are behind on source normalization.
   1. Mature teams usually do not rely on a single orchestration prompt to reason across all raw artifacts. They either normalize into fragments or into a structured intermediate representation.
3. You are directionally aligned with the right future state.
   1. The ambition is right. The likely course correction is architectural separation, not a full reset.

The right conclusion is to avoid abandoning what is already built, while stopping the skill itself from becoming the platform.

## What to change now to avoid future roadblocks

1. Introduce a canonical intermediate file now.
   1. Name it something like docs/release-evidence.yaml or docs/docs-events.yaml.
2. Move precedence rules out of the prose skill and into machine-readable config.
   1. For example, standards/source-precedence.yaml.
3. Add explicit field-level ownership.
   1. For example, change_type from issues and PR labels, shipped_status from tag reconciliation, qa_status from test evidence.
4. Split one big skill into layers.
   1. One skill for extraction and reconciliation, one for release-note generation, and later more generators for other content types.
5. Add confidence and review gates now.
   1. Do not wait until long-form docs are automated.

## Blunt assessment

You are not on the wrong track. But if the current skill keeps expanding without introducing an evidence layer, it will eventually become a brittle system that is impressive in demos and expensive in production.

The safe course correction is:

1. Preserve standards and templates.
2. Insert canonical evidence modeling before content generation.
3. Treat every content type as a renderer over normalized evidence.

That is the architecture to defend.

## Possible next steps

1. Draft a target-state architecture diagram and component breakdown.
2. Define the canonical docs-events schema.
3. Write the source precedence model as a machine-readable YAML standard.
