# Copilot Instructions

## Repository Model

This is a central, documentation-first standards repository. It defines reusable
schemas, rendering rules, validation policy, and agent workflows for consumer
repositories; it is not an application project.

- There are no package-manager, build, test, or lint commands.
- Markdown and YAML are the primary deliverables. YAML is the automation source
  of truth; rendered Markdown is the publishable output.
- Use `README.md` only as the repository stub. Start substantive exploration in
  `AGENTS.md`, then use `docs/INDEX.md` for the planning and architecture
  record.

## Validation

There is no automated test suite. For YAML changes, parse the affected files:

```bash
ruby -e 'require "yaml"; ARGV.each { |f| YAML.load_file(f); puts "OK #{f}" }' path/to/file.yaml
```

For manifest and documentation path changes, also check for obsolete internal
mirror paths:

```bash
rg -n '\.github/templates|\.github/standards|\.github/rules|\.github/skills' docs templates standards rules skills
```

The expected result is no matches except intentionally documented historical
context. There is no single-test command because no test runner exists.

## Registry and Asset Architecture

Resolve a content type through the registry rather than hardcoding asset paths:

```text
standards/content-types.yaml
  -> standards/content-types/<type>.yaml
  -> schema, rendering, rules, examples, and skill entrypoints
```

The manifest is the contract for consumers and must stay internally consistent
with its listed files. When adding a content type or making a major extension,
update all five layers together:

1. Registry entry in `standards/content-types.yaml`
2. Manifest in `standards/content-types/`
3. Schema and rendering assets in `templates/`
4. Validation rules in `rules/`
5. Orchestration skill in `skills/`

Use repository-relative canonical paths (`standards/`, `templates/`, `rules/`,
and `skills/`) in manifests. Use `.github/` only for an asset that really
exists in this repository; consumer examples should use
`consumer-repo/.github/...`.

## Evidence Pipelines

`release-notes` and `overview` are active content types. Their skills are
orchestration contracts, while the manifests resolve the canonical assets.

- **Release notes:** preserve the sequence extraction -> reconciliation ->
  structured YAML generation -> validation -> Markdown rendering. Reconciliation
  applies field-level precedence from `standards/source-precedence.yaml`,
  records provenance, and scores confidence. The release tag boundary is the
  authority for shipped status; a milestone alone is advisory. Keep raw GitHub
  evidence separate from reconciled `release-evidence.yaml`. Do not render when
  release-notes validation fails.
- **Overview:** bootstrap a long-lived `overview.yaml` once from existing
  product content. Later release-signal runs consume resolved release-notes
  output and patch only `featureCoverage` and `upcomingFeatures`; they do not
  replace the overview wholesale. A matched shipped feature must update its
  coverage row and remove the matching upcoming-feature entry in the same
  write. Keep evidence provenance and confidence handling from the overview
  standards.
- Do not duplicate source-precedence, confidence-scoring, schema, rendering,
  or mapping logic in skills. Resolve those authorities from the content
  manifest.

For changes to these flows, read the matching manifest and `skills/<type>/SKILL.md`
before changing individual templates or rules. Use the example YAML payloads
under `templates/<type>/examples/` to verify structural compatibility.

## GitHub MCP Evidence Access

Use the configured GitHub MCP server when gathering release evidence:

- Retrieve issues for the release scope, then pull requests merged in the
  version range, and commits between the release tags.
- Preserve source identifiers and raw source data for reconciliation; do not
  decide change type, shipped status, or user impact during retrieval.
- Establish shipped status from tag reachability during reconciliation, not
  from an issue state, milestone, or merged-PR status.
- Resolve the release-notes manifest before locating output used by an overview
  release-signal run.

## Documentation Boundaries

Keep standards assets concise and structural. Link to an existing canonical
document instead of restating it in a new instruction or skill. Treat `docs/`
as planning, architecture, and course-correction material rather than
publishable generated output. Keep `_unsegregated/` as a staging area until
content is assigned to its permanent standards, templates, rules, or skills
location.
