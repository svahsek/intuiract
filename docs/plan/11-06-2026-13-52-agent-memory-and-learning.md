# Agent Memory, Learning, and System Optimization

## How the Agent Uses Your Skill in a Code Repo

When someone invokes the consumer skill, here's exactly what happens:

```
User triggers skill
      ↓
SKILL.md loaded into context window  ← the agent's "working memory"
      ↓
Agent reads instructions, starts Phase 1
      ↓
Fetches referenced YAML files (source-precedence.yaml, schema, etc.)
→ These are read from disk and placed into context window
      ↓
Calls tools (git log, GitHub API, file reads)
→ Results placed into context window
      ↓
Executes all 5 phases within same context window
      ↓
Outputs release-notes.yaml + release-notes.md to disk
      ↓
Session ends → context window is completely wiped
```

---

## What the Context Window Actually Is

The context window is **temporary working memory** — it exists only for the duration of one session. Think of it as RAM, not disk. Everything the agent "knows" during a run — the YAML files, the git log output, the evidence it built, the decisions it made — lives there only while the session is active.

When the session ends, it's all gone. The only things that persist are what got written to disk: `release-notes.yaml`, `release-notes.md`, `release-evidence.yaml`.

There is no caching between runs. The next run starts completely blank and reloads everything from scratch.

---

## Do Agents Learn After Task Completion?

**No. Not by default, and not automatically.**

This is the most important thing to understand. The LLM's weights — its "knowledge" — were frozen at training time. When Claude generates release notes, that experience does not update the model. Claude Sonnet 4.6 tomorrow is identical to Claude Sonnet 4.6 today regardless of how many release notes it generated in between.

This is not a bug. It's by design. It means:
- Behaviour is predictable and reproducible
- One user's mistakes don't corrupt another user's experience
- The model you tested is the model you deploy

---

## Where "Learning" Actually Lives in Agentic Systems

Since the agent itself doesn't learn, **learning must be externalised** — encoded into artifacts the agent reads at the start of the next run. This is the fundamental design pattern.

In this framework, learning already has a home — it just hasn't been fully activated yet:

| What the agent learns | Where it should be encoded | Already exists? |
|---|---|---|
| How to resolve field conflicts | `standards/source-precedence.yaml` | ✅ |
| What fields mean and their valid values | `standards/release-evidence-fields.yaml` | ✅ |
| What good output looks like | `templates/release-notes/examples/` | ✅ partial |
| What validation rules to enforce | `rules/release-notes/` | ✅ |
| What went wrong last run | `docs/releases/retrospectives/` | ❌ missing |
| Product-specific terminology and patterns | `templates/release-notes/release-notes-quick-reference.md` | ✅ |

The gap is the retrospective layer — the mechanism that captures what happened in a run and makes it available to the next run.

---

## How to Make the Agent Learn for Next Runs

### Level 1 — Standards improvement loop (most durable)

After every release, when the agent's output is corrected, ask: "why did it get this wrong?" Then encode the answer into the standards files:

- Agent misclassified a commit type → add a rule or example to `source-precedence.yaml`
- Agent wrote marketing language → add the phrase to the prohibited list in `rules/release-notes/`
- Agent missed a migration requirement → add a detection pattern to the rules
- Agent was confused about a product-specific term → add it to the quick reference

This is the highest-value loop because the fix is permanent, versionable, and applies to every future run. The agent doesn't learn — the standard gets smarter.

### Level 2 — Per-release retrospective file

Add a Phase 6 to the skill: after rendering, the agent writes a brief structured retrospective:

```yaml
# docs/releases/v0.18.0-retrospective.yaml
version: "0.18.0"
run_date: "2026-06-11"
evidence_quality:
  strong_signals: ["All PRs had linked issues", "QA results present for all features"]
  weak_signals: ["3 commits had no conventional prefix", "2 PRs missing description"]
ambiguities_resolved:
  - field: change_type
    item: "REL-007"
    resolved_as: "enhancement"
    note: "PR had no label — inferred from commit prefix 'refactor:'"
corrections_made:
  - "Manually overrode qa_status for REL-003 — QA system was down"
rules_that_should_be_updated:
  - "Add 'refactor:' prefix → technical_improvement mapping to classification-rules.yaml"
```

Next run, the skill loads the most recent retrospective as additional context. The agent doesn't remember — it reads.

### Level 3 — Few-shot examples from real runs

The best generated release notes are more valuable than synthetic examples. After each good run, add the evidence → output pair to `templates/release-notes/examples/`. The agent learns by seeing what good looks like, not by being told rules. This is the most effective form of in-context learning.

### Level 4 — Evals (when moving to automation)

Before full CI/CD automation, build a small eval set: 5-10 known evidence inputs with verified correct outputs. Run the skill against them before any pipeline change. If quality drops, don't ship the change. This is how you measure whether standards improvements actually worked.

---

## The Right Mental Model for This Framework

```
Run N completes
      ↓
Human reviews output, identifies corrections
      ↓
Corrections encoded into:
  - standards/source-precedence.yaml     (logic improvements)
  - rules/release-notes/                 (new validation rules)
  - templates/release-notes/examples/   (new good examples)
  - docs/releases/retrospectives/        (run-specific notes)
      ↓
Run N+1 starts fresh BUT loads smarter standards
→ Agent performs better not because it learned
→ But because the system it reads from got smarter
```

The agent is stateless. The standards repository is the memory. Every improvement made to the standards is a permanent upgrade to every future run. That is why getting the standards architecture right matters more than anything the agent does in the moment.
