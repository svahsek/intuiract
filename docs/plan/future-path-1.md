# Future Path: Packaging the Documentation Intelligence Framework

## What makes this framework different

Most "AI docs" tools are just "ask Claude/GPT to summarize the diff." This framework is different in one critical way: **deterministic evidence-first** — field-level source precedence, confidence scoring, validation before rendering. That's the moat. Any packaging decision should preserve and expose that.

---

## The possibility space

### 1. GitHub Action (nearest, highest ROI)

The evidence model is already GitHub-native — issues, PRs, commits, tags. A GitHub Action triggered on tag push (`on: push: tags: 'v*'`) runs the pipeline end-to-end.

```yaml
uses: intuiract/doc-intel@v1
with:
  version: ${{ github.ref_name }}
  output: docs/releases/
```

Consumer repos get docs on every release with zero manual work. This is the most natural first packaging because `skills/release-notes/SKILL.md` is already written as a workflow contract.

### 2. MCP Server (highest leverage for AI-native world)

Package the 4 pipeline phases as MCP tools:
- `extract_evidence(repo, version_range)`
- `reconcile(evidence, source_precedence)`
- `validate(release_notes_yaml, schema)`
- `render(validated_yaml, rendering_rules)`

Any Claude agent (Claude Code, Claude Desktop, or a custom agent) can call these as tools. This is the most composable path — agents orchestrate, the server does the deterministic heavy lifting. This separates AI creativity from structured validation.

### 3. CLI tool

```bash
doc-intel generate release-notes --from v1.2.0 --to v1.3.0
```

Works in any CI/CD (not just GitHub), testable locally, scriptable. The right move once there are more consumers and the need to support non-GitHub evidence sources (Jira, GitLab, etc.).

### 4. VS Code / IDE extension

"Generate release notes" in the command palette. Tight loop for writers and PMs who want to draft before release. Lower strategic value than the others but good for adoption.

### 5. SaaS / documentation platform

Connect a repo, configure, get a live docs site that regenerates on every tag. Think Mintlify but evidence-driven and AI-native. Biggest scope, most defensible, furthest away.

---

## Recommended progression

**Right now**: Keep the standards repo as-is. It *is* the core — it just needs more consumer validation. Use it on 2-3 real repos and let the pain points surface.

**Next step**: GitHub Action. Smallest delta from the current state — `SKILL.md` is already a workflow contract, the YAML schema is already defined. Essentially wrapping execution around what is already a spec.

**Step after that**: MCP Server. Once the pipeline is packaged as callable tools, any agent can use it without knowing the internal structure. This future-proofs the framework as agentic coding workflows become standard.

**Long-term**: CLI for broad CI/CD adoption, SaaS if this becomes a product.

---

## The one thing to nail first

Before any packaging: **test the reconciliation layer on a real repo with messy data** — PRs without linked issues, inconsistent labels, missing milestones. That's where `standards/source-precedence.yaml` either earns its keep or shows gaps. The framework is only as good as that layer works against real noise. Get that right before investing in packaging.
