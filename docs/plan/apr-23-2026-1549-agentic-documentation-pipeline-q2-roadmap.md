# Next-Quarter Plan: Agentic Documentation Pipeline Q2 2026

## Purpose

Roadmap and action items for Q2 2026 (May 1 - June 30) following successful completion of the 30-day course correction and release-notes pilot.

This plan operationalizes the release-notes automation pipeline and establishes foundations for expanding to additional content types in Q3.

---

## Q1 Outcomes (Current - Completed by April 30)

✓ Release-notes pipeline designed end-to-end (extraction → reconciliation → generation → validation → rendering)  
✓ Standards repo populated with release-evidence contract, precedence rules, field glossary, confidence scoring, validation rules  
✓ Release-notes skill refactored to orchestrate 5-phase pipeline  
✓ Pilot completed on 1 production release (v1.2.3) with documented results  
✓ 15+ field-level source precedence rules validated  
✓ Confidence scoring model proven (85+/60-84/<60 tiers)  
✓ Decisions log published; standards vs. product division clarified  

---

## Q2 2026 Strategic Goals

1. **Operationalize**: Move release-notes from pilot to production automation
2. **Integrate**: Build product-repo tooling (extraction, reconciliation engines)
3. **Validate**: Run pipeline on 4+ releases; measure precision/recall
4. **Extend**: Begin planning for features-page automation (Q3)
5. **Govern**: Establish standards update process and consumer repo pattern

---

## Week-by-Week Breakdown

### Week 1 (May 1-7): Production Readiness

**Goals**:
- Extract lessons from pilot; document gaps
- Finalize standards files (versioning, completeness check)
- Begin product-repo architecture

**Deliverables**:

1. **Pilot Results Compilation**
   - Collect pilot findings from all 5 phases
   - Calculate precision (correct items) and recall (complete coverage)
   - Identify top 3-5 blockers and gaps
   - Document operator effort (time per phase)
   - File: `docs/course-correction/pilot-results-summary.md`

2. **Standards Completeness Check**
   - Audit all standards files created (evidence contract, precedence, confidence rules, etc.)
   - Verify cross-references (manifests → schemas → examples)
   - Check for orphaned or incomplete sections
   - Create issue tracking gaps (if any)

3. **Product Repo Architecture (Scoping)**
   - Define high-level architecture for product-side tooling
   - Identify 4-5 core modules:
     - GitHub Extractor (PR/issue/commit aggregation)
     - Reconciliation Engine (applies precedence, scores confidence)
     - Generator (evidence → release-notes YAML)
     - Validator (schema + policy + confidence gate)
     - Renderer (YAML → Markdown)
   - Create RFC for product team review

---

### Week 2 (May 8-14): Product Tooling Sprint 1 - Extraction & Reconciliation

**Goals**:
- Implement extraction tool in product repo
- Implement reconciliation engine
- Integrate with standards files

**Deliverables**:

1. **GitHub Extractor Tool**
   - Reads source-precedence.yaml from standards
   - Extracts issues, PRs, commits from GitHub API (base_tag..target_tag)
   - Outputs raw evidence JSON (not normalized yet)
   - Includes error handling (rate limits, missing data)
   - Usage: `extractor --repo ORG/REPO --base-tag v1.2.0 --target-tag v1.2.3`

2. **Reconciliation Engine**
   - Reads source-precedence.yaml and release-evidence-fields.yaml
   - Takes raw evidence JSON as input
   - Applies field-level precedence
   - Resolves conflicts per rules
   - Scores confidence per confidence-rule.yaml
   - Outputs release-evidence.yaml (normalized, scored)
   - Includes test suite (verify precedence logic)

3. **Integration Test**
   - Run extractor → reconciliation on real release
   - Compare output to manually-created release-evidence.yaml from pilot
   - Measure agreement (should be 95%+ identical)

---

### Week 3 (May 15-21): Product Tooling Sprint 2 - Generation & Validation

**Goals**:
- Implement generation tool (evidence → content)
- Implement validation engine
- Integrate with standards files

**Deliverables**:

1. **Generator Tool**
   - Reads evidence-to-release-notes-mapping.yaml from standards
   - Reads operator_input (headline, summary, audience, timeline)
   - Filters evidence items by confidence/change_type
   - Maps fields to release-notes schema
   - Outputs release-notes.yaml (structured content)
   - Usage: `generator --evidence release-evidence.yaml --operator-input input.yaml`

2. **Validation Engine**
   - Reads release-notes-schema.yaml, release-notes-rule.yaml from standards
   - Performs schema validation (required fields, types, patterns)
   - Performs policy validation (structural, content, security rules)
   - Performs confidence gate (check thresholds per release-type)
   - Outputs validation report (pass/fail + field-level errors)
   - Blocks rendering if validation fails

3. **Integration Test**
   - Run generator → validator on release from Week 2
   - Compare to manually-created release-notes.yaml from pilot
   - Ensure all validations pass

---

### Week 4 (May 22-28): Product Tooling Sprint 3 - Rendering & CLI

**Goals**:
- Implement rendering engine
- Build CLI orchestrating all 5 tools
- Create end-to-end test

**Deliverables**:

1. **Rendering Engine**
   - Reads release-notes-rendering.yaml from standards
   - Takes validated release-notes.yaml as input
   - Transforms to Markdown (dates, typography, links, formatting)
   - Generates table of contents
   - Outputs release-notes.md (publication-ready)
   - Usage: `renderer --yaml release-notes.yaml --output release-notes.md`

2. **Orchestration CLI**
   - Single command: `agentic-docs release-notes --repo ORG/REPO --base-tag v1.2.0 --target-tag v1.2.3`
   - Internally: calls extractor → reconciliation → generator → validator → renderer
   - Outputs both YAML artifacts and final Markdown
   - Error handling: stops on validation failure
   - Includes `--skip-extraction` option (use pre-existing evidence)

3. **End-to-End Test**
   - Run CLI on 2-3 production releases
   - Validate output quality (precision, recall, formatting)
   - Measure execution time
   - Document any issues

---

### Week 5 (May 29 - Jun 4): Production Pilot Batch

**Goals**:
- Run full pipeline on 4 production releases
- Gather metrics and operator feedback
- Identify production blockers

**Deliverables**:

1. **Production Release Batch**
   - Select 4 releases spanning different types (Major, Minor, Patch, Hotfix)
   - Run full pipeline for each
   - Document results per release:
     - Execution time (per phase)
     - Confidence distribution
     - Validation passes/failures
     - Operator corrections needed
     - Quality of final Markdown

2. **Metrics Compilation**
   - Aggregate precision/recall across 4 releases
   - Average operator time per release
   - Automation time vs. manual time
   - Cost estimate (if tooling/infrastructure costs apply)

3. **Blockers & Gaps Inventory**
   - List any issues encountered
   - Categorize: standards gap, product bug, process issue
   - Assign owners for resolution in Week 6

---

### Week 6 (Jun 5-11): Standards Refinement & Consumer Pattern

**Goals**:
- Fix blockers identified in Week 5
- Establish consumer repo pattern (how external teams use standards)
- Begin Q3 planning (features-page automation)

**Deliverables**:

1. **Blocker Resolution**
   - For each blocker from Week 5:
     - Root cause analysis
     - Fix implementation
     - Re-test on production release
     - Update standards/product code as needed

2. **Consumer Repo Integration Guide**
   - Document how external product repos fetch and use standards
   - Provide example skill that:
     - Fetches manifests from standards repo
     - Resolves schema, rendering, rule paths
     - Calls product tools with resolved paths
   - Version pinning strategy (how to stay updated)
   - File: `docs/integration/consumer-repo-integration-guide.md`

3. **Q3 Planning (Features-Page Automation)**
   - Create features-page manifest (similar to release-notes)
   - Define features-page schema (what goes in a features page)
   - Sketch extraction/mapping/validation rules
   - Create RFC for product team

---

### Week 7 (Jun 12-18): Documentation & Training

**Goals**:
- Document the full pipeline for operators and engineers
- Create runbooks for common tasks
- Train team on using the pipeline

**Deliverables**:

1. **Operator Runbook**
   - How to generate release-notes for a new release
   - How to handle validation failures
   - How to troubleshoot low confidence scores
   - Quick reference card (1 page)

2. **Engineer Runbook**
   - How to debug extraction issues
   - How to extend standards (add new field, change precedence)
   - How to add new content types
   - Architecture overview

3. **Training Session**
   - 1-hour walkthrough of full pipeline
   - Live demo on production release
   - Q&A

---

### Week 8 (Jun 19-25): Hardening & Go-Live

**Goals**:
- Final quality checks
- Establish monitoring and alerting
- Declare release-notes automation "production ready"

**Deliverables**:

1. **Quality Audit**
   - Run pipeline on 5+ additional releases
   - Manual review of generated release-notes (grammar, accuracy, tone)
   - Confidence in automation: 95%+ precision

2. **Monitoring Setup**
   - Track execution metrics (time per phase, success rate)
   - Alert on validation failures
   - Alert on low confidence scores
   - Dashboard for pipeline visibility

3. **Go-Live Announcement**
   - Blog post: "Release-Notes Automation Live"
   - Highlights: faster release documentation, consistent quality, operator effort reduced
   - Links to operator runbook
   - Support channels for issues

---

### Week 9 (Jun 26-30): Q3 Readiness & Retrospective

**Goals**:
- Prepare product team for Q3 features-page automation
- Conduct lessons-learned retrospective
- Plan any mid-Q2 adjustments

**Deliverables**:

1. **Q3 Scope Definition**
   - Features-page manifest (similar structure to release-notes)
   - Features-page schema and extraction mapping
   - Confidence rules for features content
   - Product team capacity allocation

2. **Retrospective Report**
   - What worked well in Q2
   - What was harder than expected
   - Recommended process changes
   - Estimated velocity for Q3 features-page

3. **Adjustments Planning**
   - Any hotfixes needed for release-notes pipeline
   - Improvements based on retrospective
   - Roadmap refinements

---

## Success Metrics for Q2

| Metric | Target | Status |
|--------|--------|--------|
| Release-notes pipeline end-to-end working | ✓ Complete | Baseline |
| Precision (correct items / total items) | 95%+ | To be measured |
| Recall (items found / items in release) | 98%+ | To be measured |
| Operator time per release | < 30 min | To be measured |
| Confidence scoring accuracy | 90%+ | To be measured |
| Product tooling (5 core modules) | ✓ Implemented | Milestone |
| Zero validation bypasses | ✓ Required | Mandatory |
| Consumer repo pattern documented | ✓ Complete | Milestone |
| Production batch (4+ releases) | ✓ Complete | Milestone |
| Go-live declaration | ✓ Complete | Milestone |

---

## Resource Allocation

| Role | Time % | Key Activities |
|---|---|---|
| Technical Writer | 40% | Standards refinement, documentation, operator runbooks |
| Backend Engineer | 40% | Extraction, reconciliation, CLI tools |
| Frontend Engineer | 20% | Rendering, CLI UI/UX |
| QA Engineer | 30% | Integration testing, metrics, validation |
| Product Manager | 10% | Planning, roadmap coordination |

---

## Risk Register

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Extraction logic incomplete (missing edge cases) | Medium | High | Extensive testing on diverse releases; fallback to manual review |
| Confidence scoring not accurate | Medium | Medium | Pilot validation; adjust thresholds based on data |
| Operator too-large time investment | Low | High | Automate all non-strategic inputs; measure and optimize |
| Breaking changes to standards mid-Q | Low | Medium | Versioning policy; advance notice for consumers |
| Product repo not ready by mid-Q | Low | High | Parallel workstreams; escalate if slipping |

---

## Dependencies

- ✓ Standards repo complete (D001-D015 decisions finalized)
- ✓ Release-notes pilot complete with documented results
- ⟳ Product repo created and scaffolded (Week 1)
- ⟳ GitHub API access and CI/CD pipeline (Week 1-2)
- ⟳ CLI framework selected (Week 2)

---

## Go/No-Go Decision Gate (End of Week 5)

**Criteria for Go-Live (Week 8)**:
1. Production batch (4+ releases) completed with metrics
2. Precision >= 95%, Recall >= 98%
3. Zero validation bypasses
4. Operator time < 30 min per release
5. Consumer pattern documented and tested
6. No critical blockers unresolved

If any criterion not met by end of Week 5, extend Q2 timeline or defer features-page to Q3-2.

---

## Q3 2026 Preview (Out of Scope for This Plan)

**Q3 Goals** (June planning):
- Features-page automation (similar 5-phase pipeline)
- API changes automation
- Evidence model extended for features content-type
- Product repo supports 2 content-types

**Q3 Timeline**:
- Weeks 1-4: Features-page pipeline design and build
- Weeks 5-6: Production pilot and hardening
- Weeks 7-8: Go-live and training
- Weeks 9-10: API changes scoping

**Q3 Roadmap** to be published by June 15 for Q3 team planning.

---

## Appendix: Standards Files to be Created in Q2

As standards evolve based on Q2 learnings:

- [ ] `release-notes-changelog.md` – Version history of release-notes standard changes
- [ ] `consumer-repo-integration-guide.md` – How to integrate standards in external repos
- [ ] `release-evidence-v1.1.md` – Minor updates to evidence contract based on pilot
- [ ] `confidence-scoring-tuning-guide.md` – How to adjust confidence thresholds per organization
- [ ] `release-notes-examples/` – Additional examples beyond pilot (Major, Patch, Hotfix)

---

## Appendix: Product Repo Deliverables Checklist

- [ ] GitHub Extractor tool (Week 2)
- [ ] Reconciliation engine (Week 2)
- [ ] Generator tool (Week 3)
- [ ] Validator engine (Week 3)
- [ ] Renderer tool (Week 4)
- [ ] CLI orchestrator (Week 4)
- [ ] Integration tests (Weeks 2-4)
- [ ] Production batch runs (Week 5)
- [ ] Monitoring & alerting (Week 8)
- [ ] Operator & engineer runbooks (Week 7)

---

## Version History

- **v1.0.0** (2026-04-23): Initial Q2 plan based on 30-day course correction outcomes
