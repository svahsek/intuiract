# Headway: Structured Documentation Intelligence — Next Actions

**Date:** 2026-06-13  
**Status:** Active planning document synthesised from all prior docs/plan entries

---

## Where Things Stand

The framework has a name: **Structured Documentation Intelligence (SDI)**.  
The core insight: *Standards are the brain. AI is the hand.*

Architecture is sound (Tier 4 thinking). Execution is still Tier 2 — SKILL.md is prose instructions where deterministic code should be. That gap is the only thing standing between a well-designed system and a production-reliable one.

Q2 2026 plan (April) projected Week 7 (Jun 12–18) as documentation and training week. The June 11 thinking reveals the more honest priority order: harden execution first, then document.

---

## The Three Hardening Moves (Do These First)

These were identified in `11-06-2026-ai-approach-comparison-and-positioning.md` as the highest ROI actions. Nothing else should start until these are done.

### 1. Prompt caching on standards files
**Effort:** One afternoon  
**What:** Every skill run loads `source-precedence.yaml`, `release-evidence-fields.yaml`, the schema, and rendering rules from scratch. These are large, static files. Enable Anthropic prompt caching so they are loaded once and cached across runs.  
**Why now:** Immediate token cost reduction. Applies to every future run permanently.

### 2. Structured outputs for YAML generation (Phase 3)
**Effort:** Half a day  
**What:** Replace free-form LLM generation of `release-notes.yaml` with structured output mode (JSON schema enforcement). The schema violation is caught at generation time, not after.  
**Why now:** Phase 4 validation currently catches silent failures that structured outputs would prevent entirely. Makes the pipeline more reliable without adding complexity.

### 3. Write `reconcile.py`
**Effort:** 2–3 days  
**What:** Move `source-precedence.yaml` application from LLM instruction to actual Python code. The LLM currently reads the YAML and tries to apply it — deterministic code should do this.  
**Why now:** This is the single most important gap between Tier 4 design and Tier 2 execution. Everything downstream (confidence scoring, validation) is only as reliable as this layer.

---

## Multi-Output Pipeline (After Hardening)

From `04-06-2026-13-58-future-path-2.md`: all content types should be siblings reading from `release-evidence.yaml`, not chains from `release-notes.md`.

### Step 1: Add routing layer
Create `standards/content-type-routing.yaml` — maps evidence field combinations to affected content types:

```yaml
routing_rules:
  features_page:
    condition: change_type == "feature" AND shipped == true
  user_guide:
    condition: (change_type in ["feature", "enhancement"]) AND ui_change == true
  api_reference:
    condition: api_change == true AND shipped == true
```

### Step 2: Extend the evidence model
Add two boolean fields to `standards/release-evidence-fields.yaml`:
- `ui_change` — affects a user-visible flow or screen
- `api_change` — adds/modifies/deprecates an API endpoint

**First action before writing any code:** Open a real `release-evidence.yaml` from the pilot and manually tag each item with these two fields. That exercise will reveal whether the current evidence captures enough signal to route correctly or whether the reconciliation layer needs more extraction logic first.

### Step 3: Build the features page as a full content type
This is the cleanest first addition — signal is unambiguous: `change_type == "feature" AND shipped == true`.

Full chain to build (same pattern as release-notes):
1. `standards/content-types/features.yaml` — manifest
2. `templates/features/features-schema.yaml` — schema
3. `templates/features/evidence-to-features-mapping.yaml`
4. `rules/features/` — validation rules
5. `skills/features/SKILL.md` — orchestration skill
6. Register in `standards/content-types.yaml`

### Step 4: Build the dispatch skill
Once features page exists alongside release-notes, add `skills/dispatch/SKILL.md` — loads evidence, runs routing rules, calls each content type skill, reports what was generated.

---

## Retrospective Layer (Missing, Add Now)

From `11-06-2026-13-52-agent-memory-and-learning.md`: the agent is stateless, but learning can be externalised. The retrospective layer is the only missing piece in the learning loop.

Add Phase 6 to the release-notes skill: after rendering, write a structured retrospective file.

```yaml
# docs/releases/v0.18.0-retrospective.yaml
version: "0.18.0"
run_date: "2026-06-13"
evidence_quality:
  strong_signals: []
  weak_signals: []
ambiguities_resolved: []
corrections_made: []
rules_that_should_be_updated: []
```

Next run loads the most recent retrospective as additional context. The agent does not remember — it reads. This closes the learning loop at Level 2 (per-release retrospective).

---

## Packaging Path

From `04-06-2026-12-37-future-path-1.md`:

| Step | What | When |
|---|---|---|
| **Now** | Test reconciliation on a real repo with messy data — PRs without linked issues, inconsistent labels, missing milestones | Before any packaging |
| **Next** | GitHub Action — triggered on tag push, runs pipeline end-to-end | After reconcile.py is done |
| **Then** | MCP Server — `extract_evidence`, `reconcile`, `validate`, `render` as callable tools | After GitHub Action is live |
| **Later** | CLI for non-GitHub CI/CD | When there are multiple consumers |
| **Long-term** | SaaS | When it becomes a product |

---

## Framework Positioning and Talk

From `11-06-2026-14-15-framework-naming-and-talk-title.md`:

**Name:** Structured Documentation Intelligence (SDI)  
**Tagline:** Standards are the brain. AI is the hand.

**Talk title (full):**  
> Structured Documentation Intelligence: Encoding Human Expertise for AI-Executed Documentation

**Talk title (short):**  
> Structured Documentation Intelligence: When Standards Are the Brain and AI Is the Hand

The positioning argument: this is not automation (replacing humans) — it is human expertise encoded as machine-readable standards, executed by a stateless AI every time. The intelligence is human. AI is the executor.

This framing resets the "will AI replace writers" conversation to "how do expert writers make AI do exactly what they would do."

---

## Ordered Action List

| # | Action | Effort | Dependency |
|---|---|---|---|
| 1 | Enable prompt caching on standards files in skill | Half day | None |
| 2 | Switch Phase 3 YAML generation to structured outputs | Half day | None |
| 3 | Write `reconcile.py` — move source-precedence logic to code | 2–3 days | None |
| 4 | Test reconciliation on a real messy repo | 1 day | #3 |
| 5 | Add Phase 6 retrospective to release-notes skill | Half day | None |
| 6 | Manually tag pilot `release-evidence.yaml` with `ui_change` + `api_change` | Half day | None |
| 7 | Add `ui_change` + `api_change` to `release-evidence-fields.yaml` | 1 hour | #6 |
| 8 | Create `standards/content-type-routing.yaml` | Half day | #7 |
| 9 | Build features page content type (full chain) | 2–3 days | #8 |
| 10 | Build dispatch skill | 1 day | #9 |
| 11 | Package as GitHub Action | 2–3 days | #4 |
| 12 | Package as MCP Server | 3–5 days | #11 |

---

## What This Is Not

Per `11-06-2026-ai-approach-comparison-and-positioning.md` — explicitly out of scope:

- **Agent frameworks (LangChain, CrewAI, AutoGen):** SKILL.md is already a workflow definition. No code logic to put in framework nodes yet.
- **RAG / vector databases:** Evidence is structured and bounded per release, not a large corpus.
- **Fine-tuning:** Standards files give the model enough context.
- **Vision/multimodal:** Becomes relevant only when tackling UI flow documentation in user guide delta.
