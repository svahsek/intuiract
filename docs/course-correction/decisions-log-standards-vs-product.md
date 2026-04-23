# Decisions Log: Standards vs. Product Division

## Purpose

Document architectural and operational decisions made during the 30-day course correction. Specifically, clarify which functionality belongs in the **standards repository** (this repo) vs. which belongs in a **product/tooling repository** (CLI, automation framework, etc.).

This log prevents future scope creep and unclear ownership boundaries.

---

## Decision Categories

- **Architecture**: How the system is designed and layers are separated
- **Governance**: Who owns what, approval workflows
- **Technology**: Language, frameworks, deployment
- **Release Cadence**: When updates happen
- **Backwards Compatibility**: Breaking change policy

---

## Decisions

### D001: Release Evidence as Canonical Intermediate Model

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Architecture

**Decision**:
Create a canonical normalized `release-evidence.yaml` file format that serves as the intermediate layer between extraction and generation. All generators (release-notes, features-page, API changes, etc.) consume the same evidence model.

**Rationale**:
- Separates extraction complexity from generation complexity
- Allows multiple generators without duplicating extraction logic
- Makes evidence quality observable and testable
- Enables confidence scoring at one point (reconciliation layer)

**Implications**:
- Standards repo owns the evidence contract (`release-evidence-contract-v1.md`) and schema
- Standards repo owns field semantics glossary (`release-evidence-fields.yaml`)
- Product repo owns extraction tools (GitHub API client, etc.)
- Product repo owns reconciliation engine (applies precedence rules)
- Product repo owns generators (evidence → content-type-specific YAML)

**Alternative Considered**:
- Direct GitHub extraction in each skill (rejected: duplicate logic, inconsistent precedence)

**Ownership**: Standards repo maintains contract; Product repo maintains extraction + reconciliation

---

### D002: Source Precedence Rules as Machine-Readable Standards

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Architecture

**Decision**:
Define field-level source precedence rules in `source-precedence.yaml` (YAML, not embedded in skill prose). This file is authoritative; all extraction/reconciliation logic reads from it, not hardcoded.

**Rationale**:
- Makes precedence visible and debuggable
- Allows precedence to evolve without changing code
- Enables non-engineers to understand conflict resolution
- Single source of truth prevents logic divergence across tools

**Implications**:
- Standards repo owns the precedence file
- Product repo reads the precedence file (not overrides it)
- Changes to precedence require approval in standards repo before tools implement
- Precedence versioning is tracked in standards repo

**Alternative Considered**:
- Hardcode precedence logic in each reconciliation tool (rejected: maintenance nightmare)

**Ownership**: Standards repo maintains; Product repo implements

---

### D003: Confidence Scoring as Standards Component

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Architecture

**Decision**:
Confidence scoring logic (`release-evidence-confidence-rule.yaml`) is a standards file. Scoring components, thresholds, and tier definitions are authored in standards repo.

**Rationale**:
- Confidence scoring reflects organizational risk tolerance (standards decision)
- Operators and reviewers need visibility into scoring rules
- Multiple tools need consistent scoring criteria
- Rules should be versioned and auditable

**Implications**:
- Standards repo owns the confidence-rule file
- Product repo implements the scoring algorithm (reads rules from standards)
- Changes to confidence rules require standards governance approval
- Confidence scores are reproducible and auditable

**Ownership**: Standards repo maintains; Product repo implements

---

### D004: Release-Notes Skill Stays in Standards Repo (as Orchestration Only)

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
Keep `.github/skills/release-notes/SKILL.md` in standards repo, but it must **orchestrate** (call external tools/functions) rather than **implement** extraction, reconciliation, or rendering.

The skill describes:
- Phase 1: Call extraction tool
- Phase 2: Call reconciliation tool
- Phase 3: Call generation tool
- Phase 4: Call validation tool
- Phase 5: Call rendering tool

The skill does **NOT** implement those phases.

**Rationale**:
- Standards repo is documentation-first, not a build pipeline
- Product repo houses implementation (extraction tools, CLI, etc.)
- Standards repo provides the workflow blueprint
- Skill acts as contract between standards and product

**Implications**:
- Skill is a procedure document with tool invocation syntax
- Product repo must implement all named tools
- When tools change, product repo updates; skill may not need to change
- Skill is read by both humans (understanding workflow) and machines (CLI agents)

**Alternative Considered**:
- Move skill to product repo (rejected: standards need to document the workflow)

**Ownership**: Standards repo owns skill as orchestration; Product repo owns tool implementations

---

### D005: Rendering Rules Stay in Standards; Rendering Implementation in Product

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Architecture

**Decision**:
Rendering rules (typography, formatting, style) are authored in standards repo (`release-notes-rendering.yaml`). The rendering engine (YAML→Markdown transformer) is implemented in the product repo.

**Rationale**:
- Rendering rules are organizational style decisions (standards authority)
- Rendering engine is tooling implementation (product authority)
- Rules should be versionable and auditable
- Different tools may implement rendering; all should conform to same rules

**Implications**:
- Standards repo owns `release-notes-rendering.yaml`
- Product repo owns rendering command/function
- Product repo must read rendering rules from standards at runtime
- Changes to rendering rules don't require code changes in product

**Ownership**: Standards repo maintains rules; Product repo implements engine

---

### D006: Validation Rules Owned by Standards; Validator Implementation in Product

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Architecture

**Decision**:
Validation rules (`release-notes-rule.yaml`) are standards files. Validation engine is product implementation.

**Rationale**:
- Validation rules define organizational quality gates (standards authority)
- Validator is a tool that applies those rules (product authority)
- Multiple tools may need to validate; all must apply the same rules

**Implications**:
- Standards repo owns rule files and updates them as standards evolve
- Product repo owns validator engine and reads rules at runtime
- Rule changes don't require product code changes

**Ownership**: Standards repo maintains rules; Product repo implements validator

---

### D007: Consumer Repository Pattern

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
Consumer repositories (e.g., actual product repos using this standard) fetch standards from this repo via:
1. Git submodule OR
2. HTTPS fetch at build time OR
3. Published package/tarball

Consumer repos create local skills that:
1. Fetch the manifest first
2. Resolve schema, rendering, and rule paths from the manifest
3. Call product tools with resolved paths

**Rationale**:
- Standards repo is the single source of truth
- Consumers stay up-to-date as standards evolve
- No hardcoding of paths in consumer skills
- Versioning is explicit (can pin to release tag or commit)

**Implications**:
- Standards repo must maintain a stable manifest format
- Consumer repos must implement manifest-aware skill fetching
- Backwards compatibility policy needed when manifest changes

**Ownership**: Standards repo maintains manifest contract; Consumers implement fetching

---

### D008: Extraction Tools Belong in Product Repo

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Architecture

**Decision**:
Tools that extract data from GitHub (GitHub API client, PR/issue aggregator, commit analyzer) belong in the product repo, not standards.

**Rationale**:
- Extraction is implementation (not standards)
- Different teams may implement extraction differently (GitHub CLI, API client, webhook parser)
- Standards defines the *format* of extracted data (release-evidence contract)
- Product repo is the right place for opinionated tooling

**Implications**:
- Standards repo documents the extraction contract but doesn't implement it
- Product repo builds and maintains extraction tools
- Consumer repos may bring their own extractors; must conform to release-evidence contract
- Extraction tool bugs are product repo issues, not standards issues

**Ownership**: Product repo owns extractors; Standards repo owns contract

---

### D009: Reconciliation Engine in Product Repo

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Architecture

**Decision**:
The reconciliation engine (which applies source precedence, resolves conflicts, scores confidence, normalizes) belongs in product repo. Standards repo owns the precedence rules and confidence rules.

**Rationale**:
- Reconciliation is complex algorithmic work (product domain)
- Standards repo is not a build system or runtime
- Reconciliation engine is a critical tool; needs active maintenance and optimization
- Different implementations may optimize differently; all must read the same rules

**Implications**:
- Standards repo owns `source-precedence.yaml` and `release-evidence-confidence-rule.yaml`
- Product repo owns the reconciliation algorithm
- Reconciliation runs on extracted evidence; outputs normalized evidence
- Rule changes are coordinated between standards and product repo

**Ownership**: Product repo maintains reconciliation engine; Standards repo maintains rules

---

### D010: Multi-Content-Type Support Path

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
Release-notes is the **first** content type to go through full automation. Other content types (features-page, API changes, overview) will be added **milestone-wise** after release-notes is proven.

**Rationale**:
- Limited capacity to implement multiple content-type pipelines at once
- Release-notes is highest-ROI target (most frequently generated, critical for customers)
- Lessons learned from release-notes pilot will inform other content-type implementations
- Standards can be extended per content-type without rework

**Implications**:
- Standards repo will grow to include manifests for each content-type as they're piloted
- Product repo will grow to support generators for each content-type
- Shared infrastructure (extraction, reconciliation, validation, rendering) is reusable
- Pilot success is gates Q2 and Q3 expansions

**Ownership**: Standards repo curates content-type manifests; Product repo implements per-type support

---

### D011: Standards Repo Is Documentation-First, Not Build System

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
Standards repo is a collection of Markdown, YAML, and reference examples. It is **not** a build system, runtime engine, or deployment target. It is read-only from a production perspective.

**Rationale**:
- Standards should be stable and version-controlled separately from product
- Standards should be consumable by multiple products/teams
- Separating standards from implementation enables flexibility
- Standards repo is easier to audit and maintain as documentation

**Implications**:
- No Makefile, build.gradle, or npm scripts in standards repo
- No database, deployment, or runtime in standards repo
- Standards repo publishes versioned releases (tags); consumers fetch and version themselves
- Standards is maintained by documentation team; Product is maintained by engineering team

**Ownership**: Standards repo curated by documentation team; Product repo owned by engineering

---

### D012: Evidence Versioning & Deprecation Policy

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
Release evidence schema and standards versions are tracked separately. When breaking changes are needed:
1. Create new version file (e.g., `release-evidence-contract-v2.md`)
2. Maintain old version for backward compatibility (minimum 2 releases)
3. Emit warnings when old versions are used
4. Document migration path

**Rationale**:
- Evolving standards without breaking existing releases
- Gives consumers time to upgrade
- Maintains auditability (can always look up what v1.2.3 used)

**Implications**:
- Standards repo will accumulate versioned files
- Product repo must support multiple evidence versions during migration
- Consumers must be able to opt-in to new versions

**Ownership**: Standards repo owner decides versioning policy; Product repo implements version detection

---

### D013: Validation Failure Policy

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
If release evidence or release notes fail validation, do **NOT** render or publish. Instead:
1. Report failing field(s) and line numbers
2. Move failing items to `unresolved[]` array
3. Operator reviews and approves manually OR fixes source data and re-runs

**Rationale**:
- Validation gates quality; partial or broken output is unacceptable
- Operator intervention is acceptable; automated partial publish is not
- Traceability (which operator approved which items)

**Implications**:
- Validation is mandatory, not optional
- Operators must review unresolved items before publication
- Build must fail if validation fails (no silent degradation)

**Ownership**: Standards repo owns validation rules; Product repo implements gate

---

### D014: Manual Input Required For Release Notes

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
The following fields **cannot be automated** and must be provided by operator (usually release manager or technical writer):
- Release headline
- Summary paragraphs (2-4)
- Target audience
- Rollout timeline

**Rationale**:
- These fields require human judgment (messaging, positioning, timing)
- Different audiences need different messaging
- Automation risk: auto-generated headlines may be inaccurate or off-brand

**Implications**:
- Operator workload is ~15 minutes per release (manual input only)
- Manual input is captured in operator_input section of evidence YAML
- Cannot achieve fully hands-off automation; still requires human review

**Ownership**: Release manager/technical writer provides input; Product repo captures and validates

---

### D015: Roadmap Milestones

**Status**: DECIDED  
**Date**: 2026-04-23  
**Category**: Governance

**Decision**:
Publish quarterly milestones for content-type automation:
- **Q2 2026** (Day 30): Release-notes pipeline complete and proven
- **Q3 2026**: Features-page automation
- **Q4 2026**: API changes automation
- **Q1 2027**: Overview page automation
- **Beyond**: Other content types as business priorities dictate

**Rationale**:
- Clear roadmap reduces scope creep
- Allows product team to plan capacity
- Proves value incrementally

**Implications**:
- Each milestone includes extraction, reconciliation, generation, validation, rendering for that content-type
- Standards repo will grow with new manifests, schemas, rendering rules per milestone
- Product repo will grow with new generators and validators

**Ownership**: Product leadership owns roadmap priority; Standards repo curates plans

---

## Pending Decisions (To Be Made After Pilot)

| Decision ID | Topic | Status | Target Date |
|---|---|---|---|
| D016 | Error handling & retry policy for extraction failures | PENDING | After pilot |
| D017 | Audit trail: Who generated what version of release-notes | PENDING | After pilot |
| D018 | Consumer repo governance (how they extend/override standards) | PENDING | Q2 planning |
| D019 | Security scanning integration with evidence (secrets, CVE scan results) | PENDING | Q3 planning |
| D020 | Rollback policy: How to correct published release-notes | PENDING | Q2 planning |

---

## Decision Review Schedule

- **Weekly**: Decisions log reviewed in team sync (first 5 minutes)
- **Monthly**: Decision audit (correct/update/retire old decisions)
- **Quarterly**: Strategic review (major shifts in strategy)

---

## Decision Appeal Process

If a decision needs to be revisited:
1. File an issue in standards repo with tag `decision-appeal`
2. Include:
   - Decision ID (e.g., D005)
   - Current decision text
   - Reason for appeal (what changed? What didn't work?)
   - Proposed alternative
3. Review in weekly team sync
4. If approved, create new decision (e.g., D005-Rev1) with superseding language

---

## Version History

- **v1.0.0** (2026-04-23): Initial decisions log with D001-D015
