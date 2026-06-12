# AI Approach Comparison and Framework Positioning

## How the Approaches Actually Compare

**Tier 1 — Plain markdown instructions**
A single detailed markdown file telling the LLM what to do. Most common. No schema, no validation, no precedence rules. Output quality depends entirely on how well the LLM follows prose instructions. Inconsistent across runs, breaks silently, no traceability.

**Tier 2 — Multiple markdown files / custom agents as markdown**
Better decomposition — separate files for persona, instructions, examples. Some teams call these "skills," some call them "agents," some call them "rules." Still fundamentally the same: LLM reads prose, LLM decides. The structure is cosmetic. No hard enforcement.

**Tier 3 — Mixed approaches (markdown + some code)**
Some steps are code (git extraction scripts, schema validation via a Python script), some steps are LLM instructions. Inconsistent architecture but more reliable than pure markdown because critical logic isn't LLM-guessed.

**Tier 4 — This framework**
Same execution layer as Tier 2 (SKILL.md is still markdown instructions), but the *standards layer* is fundamentally different:
- `source-precedence.yaml` defines field resolution deterministically — it just isn't *executed* deterministically yet
- `release-evidence-fields.yaml` defines a typed evidence contract
- Confidence scoring is a real model, not vibes
- Validation is a hard gate, not a suggestion
- Registry-manifest-consumer separates standards from consumption

The design is Tier 4 thinking implemented with Tier 2 execution tooling. That's the honest position.

**Tier 5 — What the best unreleased internal pipelines look like**
Reconciliation logic as actual code. LLM called only for prose generation. Evals measuring output quality over time. Prompt caching on standards files. MCP tools for GitHub extraction instead of LLM-instructed git commands.

---

## What AI Advancement Is Actually Relevant Right Now

Not everything new matters. Here's what specifically applies to this pipeline:

**Prompt caching — immediate ROI, use now**
Every time the skill runs, it loads `source-precedence.yaml`, `release-evidence-fields.yaml`, the schema, rendering rules. These are large static files that don't change between runs. Anthropic's prompt caching means you send them once, cache them, and pay near-zero tokens on subsequent runs. This directly cuts token cost on the reconciliation and generation phases.

**Structured outputs — use now for YAML generation**
The Phase 3 output (`release-notes.yaml`) should be generated with structured output mode (JSON schema enforcement), not free-form LLM generation that is then validated. The schema violation is caught at generation time, not after. Makes Phase 4 validation a confirmation step rather than a catch-all.

**MCP for GitHub extraction — relevant when building the Action**
Instead of the LLM being instructed to "run `git log v0.17.0..v0.18.0`", an MCP server exposes real GitHub tools. The LLM calls `get_issues(milestone)`, `get_prs(base_tag, target_tag)`, `get_commits(range)` as actual tool calls with real return values. More reliable than instructed shell commands.

**Evals — critical before automating further**
As the pipeline moves toward CI/CD, a way to know if a new Claude version produces worse release notes than the previous one is essential. Without evals — a small set of "given this evidence, the correct release notes look like this" test cases — the pipeline is flying blind across model upgrades.

---

## What Is NOT Missing

**Agent frameworks (LangChain, CrewAI, AutoGen)**: Add engineering overhead without giving anything not already present. SKILL.md *is* a workflow definition. Converting it to LangChain nodes before there is code logic to put in them is premature.

**RAG / vector databases**: Not relevant to this pipeline. The evidence is structured and bounded per release, not a large corpus to search.

**Fine-tuning**: Not warranted. The standards files give the model enough context. Fine-tuning is for cases where in-context learning isn't working.

**Vision/multimodal**: Not relevant yet. Becomes relevant when tackling UI flow documentation in the user guide delta.

---

## Concrete Verdict

Ahead of Tier 1-2 teams on architecture. Behind Tier 5 on execution. The gap is not about missing a new AI capability — it's about hardening the one layer where the design says "deterministic" but the implementation says "LLM, please try." That layer is reconciliation.

The three things with actual ROI right now, in order:

1. **Prompt caching** on standards files — immediate token cost reduction, one afternoon of work
2. **Structured outputs** for YAML generation — removes the largest source of silent failure
3. **Write `reconcile.py`** — move `source-precedence.yaml` application from LLM instruction to actual code

Everything else is either premature or cosmetic.
