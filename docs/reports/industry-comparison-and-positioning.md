# Industry Comparison and Framework Positioning

## What Most "Best" Teams Are Actually Doing

The majority of top documentation teams — including those at well-regarded companies — fall into one of these buckets:

**Bucket 1 — Manual + polished (Stripe, Vercel)**
Exceptional docs, but largely handcrafted by dedicated writers. Heavy investment in style guides and review processes. No real automation of release-time content. The quality comes from discipline, not systems.

**Bucket 2 — API reference generation (most enterprise)**
Solved the structured-code → structured-docs problem well: OpenAPI → API reference, TypeDoc/Javadoc → SDK reference. But this is schema-to-docs, not evidence-to-docs. Release notes, guides, and feature pages are still manual or LLM-prompted.

**Bucket 3 — Lazy AI prompting (most "AI-native" doc tools)**
Mintlify, Gitbook AI, Readme AI — "give us your PR or diff, we'll write something." No source precedence, no confidence scoring, no validation gate. Output quality is inconsistent and the LLM guesses fields it doesn't actually know. This is the current mainstream "AI docs" approach and it's architecturally weak.

**Bucket 4 — A handful of large orgs with internal pipelines**
Google, Meta, Microsoft have internal tooling that goes deeper. But what they've solved is mostly at the API reference and changelog level, and it's code-driven (Python/Go pipelines), not agent-skill-driven.

---

## Where This Framework Actually Stands

### What is genuinely uncommon

- **Field-level source precedence**: A `source-precedence.yaml` that deterministically resolves conflicts between issue labels, PR labels, commit prefixes, and QA results. Nobody in the public tooling space has this. Most teams either pick one source or let the LLM guess.
- **Confidence scoring on evidence**: This is data engineering applied to documentation. It's closer to an ETL pipeline with quality gates than a typical doc workflow. Rare.
- **Validation before rendering as a hard gate**: Most tools render first, check later (or never). This pipeline blocks output if validation fails. That's the right architecture.
- **Registry-manifest-consumer pattern**: Cleanly separating "what the standard is" from "how a consumer uses it" is good software architecture. Most doc standards repos are just a folder of markdown files people copy.

### What the gap is between this and best-in-class

- **The pipeline is instruction-driven, not code-driven.** `skills/release-notes/SKILL.md` tells an LLM to apply `standards/source-precedence.yaml` — a capable LLM follows it well, but it's not deterministic execution. The best-in-class version has the reconciliation layer as actual code (Python/Node) where precedence rules are literally `if/else` logic, and the LLM is only involved in the creative/compositional steps (writing prose descriptions, summarising user impact). Currently the LLM does both, which works but isn't fully deterministic.
- **Pilot depth**: The architecture is sound but tested on limited real-world data. The messy cases (140 commits, missing labels, conflicting signals) are where the evidence model earns its keep or shows gaps. More reps needed.
- **Feedback loop**: The best pipelines have metrics — doc coverage rate, confidence score distributions, how often manual review is triggered. That instrumentation doesn't exist yet.

---

## Is This the Right Direction?

Yes, for one clear reason: **the lazy prompting approach hits a quality ceiling.** At some release cadence and team size, "ask the LLM to summarise the diff" produces inconsistent, unverifiable output that erodes trust. This approach produces output that is traceable — every field has a provenance. That's what scales.

The one thing to internalise: **the LLM should do the creative work, the code should do the logic.** Right now the LLM does both. As the framework moves toward packaging (CLI, GitHub Action, MCP server — see `docs/plan/future-path-1.md`), migrating the reconciliation and validation layers to actual code is what elevates this from "a well-designed set of instructions" to "a documentation intelligence system."

**Summary**: Ahead of most teams on architecture. Behind on execution depth and tooling maturity. That's exactly the right position to be in at this stage.
