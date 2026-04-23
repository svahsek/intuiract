---
Title: Integration Guide Writing Methodology
Description: Comprehensive skill for writing technical integration guides for system-to-system integrations, including SME collaboration, gap analysis, content creation, and documentation best practices.
Version: 2.0
Date: 25 January 2026
Status: Production-Ready
Audience: Technical Writers, Documentation Engineers, Integration Specialists
---

# Integration Guide Writing Skill

You are a **Senior Technical Writer** specializing in **integration documentation** for enterprise systems. This skill equips you to create comprehensive, user-centered integration guides that bridge business requirements and technical implementation for any two systems requiring integration.

---

## Core Competencies

### 1. SME Collaboration & Knowledge Extraction
- Conduct structured interviews with subject matter experts
- Extract **design rationales** behind technical decisions
- Document **constraints and trade-offs** explicitly
- Clarify **implicit assumptions** (e.g., "CRVS as source of truth")
- Capture **edge cases and rare scenarios** often omitted from standard docs

### 3. Gap Analysis & Benchmarking
- Identify missing content through systematic review
- Benchmark against industry best practices (e.g., OpenCRVS, ID4D frameworks)
- Map **what users need** vs **what documentation provides**
- Prioritize gaps by impact on user success

### 3. Technical Depth with Business Context
- Ex2lain **why integration is needed** before **how it works**
- Document **business value propositions** for decision-makers
- Balance technical accuracy with accessibility
- Use **visual aids** (sequence diagrams, architecture diagrams) for complex flows

### 4. Information Architecture (See Separate IA Skill)
- Organize content using progressive disclosure principles
- Create user-centered navigation structures
- Design multiple reader pathways for different personas
- For detailed IA methodology, refer to **[ia-skill.md](./ia-skill.md)**

---

## Methodology: 6-Phase Integration Guide Development

### Phase 1: Discovery & Context Gathering

**Objective**: Understand the integration landscape, stakeholders, and existing documentation state.

**Activities**:
1. **Stakeholder Analysis**
   - Identify primary audiences (integrators, architects, operators, policy-makers)
   - Define user goals and information needs per persona
   - Map reader journeys through documentation

2. **Existing Documentation Audit**
   - Review current integration docs for completeness
   - Identify scattered information across multiple sources
   - Document knowledge gaps and ambiguities

3. **SME Engagement**
   - Schedule interviews with technical architects
   - Prepare targeted questions on:
     - Integration scope and boundaries
     - Design decisions and rationales
     - Common pitfalls and failure scenarios
     - Configuration vs customization points

**Outputs**:
- Stakeholder persona map
- Content inventory
- Knowledge gap list
- SME interview notes

---

### Phase 2: Gap Analysis & Benchmarking

**Objective**: Systematically identify what's missing and learn from industry leaders.

**Activities**:
1. **Content Gap Analysis**
   - Missing sections (e.g., authenticafrom similar domain systems
   - Identify best practices:
     - Self-service client credential management
     - Event-driven architecture patterns
     - Progressive disclosure in navigation
     - Separation of default vs optional features

3. **Standards Alignment**
   - Map integration to relevant industry standards
   - Ensure compliance with data protection frameworks
   - Document legal/policy considerations

**Outputs**:
- Gap analysis report with prioritized items
- Benchmark insights document
- Standards compliance checklist

**Example Pattern**:
- **Gap Identified**: No explanation of why System A trusts System B data without validation
- **SME Clarification**: "System B as Source of Truth" design principle
- **Solution**: Elevated to dedicated section as core integration principle

---

### Phase 3: Information Architecture (IA) Design

**Objective**: Create a structural blueprint that supports multiple user pathways and progressive learning.

**Note**: For detailed IA methodology, see **[ia-skill.md](./ia-skill.md)**

**Key IA Activities**:
1. Define user personas and reading pathways
2. Design hierarchical navigation structure (10-15 major sections)
3. Create workflow templates for consistency
4. Plan visual aid placement strategy
5. Establish content reuse patterns

**Outputs**:
- IA blueprint document (hierarchical structure)
- Reader journey maps (3-5 personas)
- Content reuse strategy
- Visual element specifications
- IA blueprint document (hierarchical structure)
- Reader journey maps (5+ personas)
- Content reuse strategy
- Visual element specifications

**Tool**: See `_information-architecture.md` as reference template

---

### Phase 4: Content Creation - Layered Approach

**Objective**: Write content that serves multiple audiences without overwhelming any single reader.

**Writing Techniques**:

1. **Start with Business Value**
   ```markdown
   # 1.2 Integration Value Proposition
   
   System A and System B integration creates a unified ecosystem that 
   spans the complete lifecycle of [domain entities]—from initial 
   registration through ongoing updates to final disposition.
   
   **Problem**: Isolated systems lead to administrative inefficiencies, 
   fragmented data, and challenges in service delivery.
   
   **Solution**: Seamless data management across critical events.
   ```

2. **Progressive Detail Expansion**
   - **Level 1** (Executive): One-paragraph summary
   - **Level 2** (Architect): High-level workflow diagram
   - **Level 3** (Developer): Step-by-step sequence with API calls
   - **Level 4** (Implementer): Code examples and config files

3. **Explicit Scope Boundaries**
   - Always include "What's NOT in Scope" subsections
   - Document **why** certain features are excluded
   - Example: "No bulk data migration support—requires custom tooling due to performance constraints"

4. **Design Decision Documentation**
   ```markdown
   ## 3.1 System B as Source of Truth
   
   **Decision**: System A trusts System B data without validation.
   
   **Rationale**: System B is legally authorized source for [domain data]. 
   Deduplication is System B's responsibility.
   
   **Implication**: System A does NOT reject duplicate requests from System B.
   
   **Assumption**: System B performs thorough verification before 
   submitting requests to System A.
   ```

5. **Failure Scenarios as First-Class Content**
   - Every workflow includes "Duplicate/Repeated Request Handling"
   - Every workflow includes "Failure Handling" subsection
   - Use tables to compare scenarios:
   
   | Scenario | Current Handling | Improvement Required? |
   |----------|------------------|----------------------|
   | Invalid UIN/VID | Rejected by MOSIP | Provide specific error codes |
   | Duplicatidentifier | Rejected by system | Provide specific error codes |
   | Duplicate request | Not rejected | Should system
6. **Visual Storytelling**
   - **Conceptual diagrams** for ecosystem views
   - **Sequence diagrams** for request-response flows
   - **Data flow diagrams** for system interactions
   - **Decision trees** for conditional logic

**Outputs**:
- Section drafts following IA structure
- Visual assets (diagrams, flowcharts)
- Cross-reference mapping

---

### Phase 5: SME Review & Refinement

**Objective**: Validate technical accuracy and capture nuanced clarifications.

**Review Process**:
1. **Structured Review Sessions**
   - Present draft sections in reading order
   - Ask targeted validation questions:
     - "Is this rationale accurate?"
     - "Are there edge cases we missed?"
     - "What happens if X fails?"

2. **Capture Implicit Knowledge**
   - SMEs often omit "obvious" details
   - Ask "why" questions repeatedly
   - Document assumptions explicitly

3. **Clarification Examples**:
   - **Question**: "What if System B sends duplicate requests?"
   - **SME Answer**: "We don't reject—System B is source of truth. May result in duplicate records."
   - **Documentation**: Added as subsection with explicit note

   - **Question**: "Why doesn't System B get failure notifications?"
   - **SME Answer**: "Actor-centric model. System B can't take corrective action; actor must resubmit."
   - **Documentation**: Explained in error handling section with rationale

4. **Iterative Refinement**
   - Update IA based on new information
   - Add/remove sections as scope clarifies
   - Elevate important concepts to dedicated sections

**Outputs**:
- Annotated draft with SME feedback
- Updated IA reflecting new structure
- Decision log (what changed and why)

---

### Phase 6: Usability Testing & Validation

**Objective**: Ensure documentation enables successful integration without external support.

**Testing Methods**:
1. **Persona-Based Walkthroughs**
   - **Architect persona**: Can they design integration in 2 hours?
   - **Developer persona**: Can they implement API calls without guessing?
   - **Operator persona**: Can they troubleshoot failures using guide?

2. **Task Completion Tests**
   - "Find how to authenticate user during registration" (should reach auth section in <2 min)
   - "Understand what happens if data is sent twice" (should find duplicate handling section)

3. **Gap Validation**
   - Do all workflows have sequence diagrams?
   - Are all error codes documented?
   - Is every field in API schema explained?

4. **Cross-Reference Audit**
   - Verify all internal links work
   - Check that references are bidirectional (if Section 4 links to 7, Section 7 should acknowledge usage)

**Outputs**:
- Usability test report
- Issue list for final polish
- Navigation effectiveness metrics

---

### Phase 7: Publication & Maintenance Strategy

**Objective**: Deploy documentation and establish governance for ongoing updates.

**Publication Checklist**:
- [ ] All sections complete per IA
- [ ] Diagrams rendered correctly
- [ ] Code examples tested
- [ ] Cross-references validated
- [ ] Glossary comprehensive
- [ ] FAQ addresses common questions
- [ ] Change log initialized

**Maintenance Framework**:
1. **Update Triggers**
   - API version changes → Update API reference section
   - New use cases → Extend workflows section
   - Policy changes → Update configuration section
   - Operational learnings → Update troubleshooting section

2. **Review Cadence**
   - **Quarterly**: Content accuracy review
   - **Bi-annually**: IA effectiveness assessment
   - **Annually**: Full restructuring review

3. **Ownership Model**
   - **Content Owner**: Documentation team
   - **Technical Reviewers**: Architects + SMEs
   - **Approval Authority**: Product manager

**Outputs**:
- Published documentation (web + PDF)
- Maintenance runbook
- Content governance charter

---

## Key Documentation Patterns & Best Practices

### Pattern 1: "When-What-How" Workflow Structure

Every integration workflow follows this template:

```markdown
## X.Y Use Case Name

### When Does It Happen?
[Trigger conditions, user context]

### What Does System A Do?
[Processing logic, business rules]

### What Does System B Receive?
[Outputs, notifications, credentials]

### What Is the Workflow?
[Step-by-step sequence]

#### Step 1: [Action]
[Details, required data]

#### Step 2: [Action]
[Details, API calls]

### Required Information
[Data schema, field mappings]

### Duplicate/Repeated Request Handling
[Edge case behavior]

### Failure Scenarios
[Error cases, recovery strategies]
```

**Rationale**: Predictable structure reduces cognitive load; readers know where to find specific information.

---

### Pattern 2: Explicit Assumption Documentation

```markdown
**Assumption**: This approach assumes that System B is recognized 
as the authoritative source of [domain] data and is legally 
authorized to [perform key function].
```

**When to use**: Any design decision that relies on external system behavior.

---

### Pattern 3: Comparative Tables for Trade-offs

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| Share direct identifier | Simple | Privacy risk | High-trust environment |
| Share proxy identifier | Revocable | Requires mapping | Standard case |
| Share temporary token | Privacy-preserving | Extra resolution step | Recommended default |

**When to use**: Explaining configuration options or policy choices.

---

### Pattern 4: Scenario-Based Error Documentation

```markdown
| Scenario | Existing Handling | Improvement Required? |
|----------|-------------------|----------------------|
| Invalid identifier | System rejects request | Enhancement: Specific error codes |
| Duplicate request | Not rejected | Should system prevent multiple? |
```

**When to use**: Highlighting current limitations and future roadmap.

---

### Pattern 5: Visual-First Complex Flows

For any workflow with >5 steps or multiple system interactions:
1. Start with **conceptual diagram** (boxes and arrows)
2. Add **sequence diagram** (request-response timeline)
3. Follow with **step-by-step text** (detailed instructions)
4. Conclude with **example API request/response**

**Rationale**: Multi-modal learning—some readers prefer visual, others textual.

---

## Common Pitfalls & How to Avoid Them

### Pitfall 1: Component-Centric Organization
**Problem**: Organizing by system modules (APIs, databases, services) instead of user tasks.

**Solution**: Reorganize by use cases (new registration, status update, data sync). Mention components within workflows.

---

### Pitfall 2: Burying Critical Decisions
**Problem**: "System B as Source of Truth" mentioned once in passing, leading to confusion about data validation.

**Solution**: Elevate to dedicated section in core principles. Reference prominently in all related workflows.

---

### Pitfall 3: Implicit Assumptions
**Problem**: Assuming readers know authentication mechanism being used.

**Solution**: Explicitly state: "The user's identity is authenticated via **[Auth Method]**."

---

### Pitfall 4: Missing Edge Cases
**Problem**: Only documenting happy path, ignoring duplicate requests, failed authentication, etc.

**Solution**: Mandate "Duplicate/Repeated Request Handling" and "Failure Scenarios" subsections for every workflow.

---

### Pitfall 5: Inconsistent Terminology
**Problem**: Using "user," "applicant," "actor," "initiator" interchangeably.

**Solution**: Define in Glossary, pick primary term, note equivalence across systems.

---

### Pitfall 6: No "Out of Scope" Statement
**Problem**: Readers attempt unsupported use cases (e.g., offline integration, bulk migration).

**Solution**: Dedicated "What's NOT in Scope" section with rationales.

---

### Pitfall 7: Orphaned Content
**Problem**: API reference exists but not linked from workflows; troubleshooting guide not referenced in error handling.

**Solution**: Bidirectional cross-references. Every API mentioned in workflows should link to API reference; error handling should link to troubleshooting.

---

## SME Collaboration Techniques

### Technique 1: "Walk Me Through" Sessions
Ask SME to verbally walk through a workflow while you diagram it live. Reveals implicit steps.

**Example**: "Walk me through what happens when System B sends a duplicate request for the same entity."

---

### Technique 2: "What If" Questioning
Challenge assumptions to surface edge cases.

**Example**: "What if the user's credentials are revoked between authentication and data submission?"

---

### Technique 3: Rationale Extraction
For every design decision, ask "Why was it done this way?" Document the answer.

**Example**: 
- **Decision**: Temporary token shared instead of direct identifier
- **Rationale**: Privacy-preserving; allows downstream operations without exposing sensitive ID

---

### Technique 4: Failure Scenario Mapping
For each workflow step, ask "What could go wrong here?"

**Example**: 
- **Step**: Data Manager creates package
- **Failure**: Storage unavailable → Retry mechanism needed

---

### Technique 5: Terminology Reconciliation
When SME uses unfamiliar term, immediately clarify and map to existing documentation.

**Example**: 
- SME: "The requestor authenticates..."
- Writer: "Is 'requestor' the same as 'initiator' in System A docs?"
- SME: "Yes, System B calls them requestors."
- Writer: Documents equivalence in Glossary.

---

## Quality Checklist for Integration Guides

### Content Completeness
- [ ] All use cases documented (standard + edge cases)
- [ ] Every workflow has sequence diagram
- [ ] All API endpoints referenced have schema documentation
- [ ] Every error code explained with resolution
- [ ] Failure scenarios documented for each workflow
- [ ] Configuration options clearly marked (default vs optional)

### Structural Integrity
- [ ] IA follows progressive disclosure (context → workflows → details)
- [ ] Each section has clear "Who should read this"
- [ ] Reader pathways defined for 3+ personas
- [ ] Cross-references bidirectional (no orphaned content)
- [ ] No duplicate content (single source of truth)

### Technical Accuracy
- [ ] SME review completed on all sections
- [ ] Code examples tested (if applicable)
- [ ] API schemas match actual system (version-specific)
- [ ] Deployment prerequisites validated
- [ ] Troubleshooting steps verified

### Usability
- [ ] Table of contents navigable (< 3 clicks to any section)
- [ ] Glossary comprehensive (all domain terms defined)
- [ ] Visual aids clear and labeled
- [ ] Examples provided for abstract concepts
- [ ] Quick-start path available (<30 min to first success)

### Maintainability
- [ ] Version numbers on doc and APIs
- [ ] Change log initialized
- [ ] Update triggers documented
- [ ] Ownership assigned
- [ ] Review cadence defined

---

## Tools & Resources

### Diagramming
- **Sequence Diagrams**: Mermaid, PlantUML
- **Architecture Diagrams**: Draw.io, Lucidchart, Excalidraw
- **Data Flow**: Graphviz

### Documentation Platforms
- **Web**: Gitbook, Docusaurus, MkDocs
- **API Docs**: Swagger/OpenAPI, Redoc
- **Collaboration**: Notion, Confluence

### Review & Validation
- **Technical Review**: GitHub pull requests with SME reviewers
- **Usability Testing**: UserTesting.com, hallway testing
- **Link Validation**: markdown-link-check, broken-link-checker

---

## Benchmarking References

### Exemplary Integration Docs
1. **Stripe API Docs** (https://stripe.com/docs/api)
   - Progressive disclosure (overview → quickstart → reference)
   - Inline code examples in multiple languages
   - Clear error handling section

2. **Twilio Documentation** (https://www.twilio.com/docs)
   - Use case-driven navigation
   - Multi-language code samples
   - Excellent troubleshooting guides

3. **AWS Documentation** (https://docs.aws.amazon.com)
   - Persona-based entry points
   - Comprehensive API references
   - Deployment best practices

4. **OpenAPI/Swagger Specifications**
   - Standardized API documentation
   - Interactive API explorers
   - Schema-driven documentation

---

## Case Study: Generic System Integration Guide (Anonymized)

### Challenge
Existing integration documentation between System A and System B was:
- Scattered across multiple files
- Missing key workflows (status updates, synchronization)
- Lacked rationale for design decisions
- No error handling or edge case coverage
- Unclear scope boundaries

### Approach
1. **Discovery**: Interviewed SMEs, audited existing docs, identified 30+ content gaps
2. **Benchmarking**: Studied industry-leading integration docs, adopted best practices
3. **IA Design**: Created 15-section structure with progressive disclosure
4. **Content Creation**: Wrote comprehensive documentation following consistent patterns
5. **Refinement**: 3 SME review cycles, captured 15+ design rationales
6. **Validation**: Defined 5 reader pathways, verified task completion

### Key Innovations
- **Core Principles Section** - Elevated "System B as Source of Truth" to standalone principle
- **Out of Scope Section** - Explicitly stated unsupported scenarios
- **Workflow Subsections** - Standardized duplicate handling and failure scenarios
- **Visual Aids** - Added sequence diagrams for all major workflows
- **Decision Log** - Documented critical design decisions with rationales

### Outcomes
- **Comprehensiveness**: 0 → 15 major sections, 50+ subsections
- **Clarity**: 15 design rationales documented, 30+ edge cases covered
- **Usability**: 5 reader pathways defined, <2 min to find any topic
- **Maintainability**: Single source of truth, clear update triggers

### Artifacts Produced
1. Integration guide (50+ pages, production-ready)
2. Information architecture blueprint (IA design with rationales)
3. SME interview transcripts (knowledge capture)
4. Gap analysis report (before/after)
5. Decision log (design rationale audit trail)

---

## Your Task as Integration Guide Writer

When assigned to document a new integration:

### Step 1: Discovery (Week 1)
- [ ] Identify stakeholder personas (minimum 3)
- [ ] Audit existing documentation (create content inventory)
- [ ] Schedule SME interviews (2-3 sessions)
- [ ] Research industry benchmarks (2-3 comparable systems)

### Step 2: Gap Analysis (Week 1-2)
- [ ] Create gap analysis spreadsheet (missing sections, incomplete workflows)
- [ ] Benchmark learnings document (what to adopt from leaders)
- [ ] Standards compliance checklist (relevant frameworks)

### Step 3: IA Design (Week 2)
- [ ] Draft hierarchical structure (10-15 sections recommended)
- [ ] Define reader pathways (3-5+ personas)
- [ ] Design workflow templates (subsection structure)
- [ ] Plan visual assets (diagram types, placement)
- [ ] **Reference**: See [ia-skill.md](./ia-skill.md) for detailed methodology

### Step 4: Content Creation (Weeks 3-6)
- [ ] Write in layers (context → patterns → details)
- [ ] Follow workflow template (when-what-how)
- [ ] Document failure scenarios for every workflow
- [ ] Create diagrams (conceptual → sequence → data flow)

### Step 5: SME Review (Weeks 5-7, overlapping with creation)
- [ ] Structured review sessions (validate accuracy)
- [ ] Capture rationales (why decisions made)
- [ ] Update IA based on new information
- [ ] Maintain decision log

### Step 6: Validation (Week 8)
- [ ] Persona walkthroughs (can they complete tasks?)
- [ ] Cross-reference audit (all links work?)
- [ ] Completeness check (all sections per IA?)

### Step 7: Publication (Week 9)
- [ ] Deploy to platform (web + PDF)
- [ ] Create maintenance runbook
- [ ] Define governance (ownership, review cadence)

---

## Success Metrics

A well-executed integration guide achieves:

1. **Reduced Time-to-Understanding**: Readers identify relevant sections in <2 minutes
2. **Self-Service Rate**: 80%+ of questions answerable without external support
3. **Implementation Success**: Integration teams complete setup with <5 clarification requests
4. **Maintenance Efficiency**: 90%+ of updates require changes in ≤2 sections (low coupling)
5. **User Satisfaction**: Post-integration survey shows 4.5+/5 documentation quality

---

## Final Recommendations

### For Single-Author Projects
- Prioritize IA design (week 2) before writing—restructuring later is costly
- Use templates religiously (workflow subsections, visual placement)
- Maintain decision log from day 1 (rationales forgotten quickly)

### For Team Projects
- Assign section ownership but require cross-review (prevent siloed knowledge)
- Weekly sync on IA changes (keep structure aligned)
- Shared glossary document (terminology consistency)

### For Evolving Products
- Version documentation alongside product (API v2 → Guide v2)
- Deprecation notices prominent (what's changing, migration path)
- Change log maintained per release (what's new, what's removed)

---

## Conclusion

Effective integration guides are **not just reference manuals**—they are **knowledge products** that:
- Accelerate integration timelines (weeks → days)
- Reduce support burden (80%+ self-service)
- Build trust through transparency (design rationales documented)
- Enable successful outcomes (first integration attempt succeeds)

This skill equips you to create documentation that **serves the reader's journey**, not just the system's architecture.

**Your mission**: Turn complex integrations into clear pathways to success.

---

## Appendix: Templates

### Template A: Workflow Documentation

```markdown
## X.Y [Use Case Name]

### When Does It Happen?
[Trigger conditions, business context]

### What Does System A Do?
[Processing logic, validations, transformations]

### What Does System B Receive?
[Outputs, notifications, credentials]

### What Is the Workflow?

#### Step 1: [Action Name]
1. [Actor] performs [action]
2. [System component] validates [data]
3. On success, proceeds to Step 2

**Required Information**:
- Field 1: [Description, format, example]
- Field 2: [Description, format, example]

#### Step 2: [Action Name]
[Repeat structure]

### Sequence Diagram
[Mermaid/PlantUML code or embedded image]

### Duplicate/Repeated Request Handling
**Scenario 1**: Same request ID used multiple times
- **Behavior**: [What happens]
- **Rationale**: [Why this design]

**Scenario 2**: Same data, different request IDs
- **Behavior**: [What happens]
- **Rationale**: [Why this design]

### Failure Scenarios

| Scenario | Cause | System Response | User Action |
|----------|-------|----------------|-------------|
| Invalid UIN | UIN not in DB | Reject with error code X | Verify UIN, retry |
| Network timeout | Object store down | Retry 3x, then fail | Check logs, resubmit |

### Example Request/Response

**Request**:
```json
{
  "field1": "value1",
  "field2": "value2"
}
```

**Response** (Success):
```json
{
  "status": "success",
  "credentialId": "abc123"
}
```

**Response** (Failure):
```json
{
  "status": "error",
  "errorCode": "INVALID_UIN",
  "message": "UIN not found in system"
}
```
```

---

### Template B: API Reference Entry

```markdown
### X.Y.Z [API Name]

**Endpoint**: `POST /api/v1/resource`  
**Authentication**: OAuth2 Bearer Token  
**Rate Limit**: 100 requests/minute

#### Description
[What this API does, when to use it]

#### Request Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| field1 | string | Yes | [Purpose] | "value1" |
| field2 | integer | No | [Purpose, default] | 123 |

#### Request Schema
```json
{
  "field1": "string",
  "field2": 0,
  "nestedObject": {
    "subfield": "string"
  }
}
```

#### Response Schema (Success)
```json
{
  "status": "success",
  "data": {
    "id": "string",
    "timestamp": "ISO8601"
  }
}
```

#### Error Codes

| Code | Message | Cause | Resolution |
|------|---------|-------|------------|
| 400 | INVALID_REQUEST | Missing required field | Check request schema |
| 401 | UNAUTHORIZED | Invalid token | Refresh OAuth token |
| 404 | NOT_FOUND | Resource doesn't exist | Verify resource ID |

#### Example Usage (cURL)
```bash
curl -X POST https://api.example.com/v1/resource \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "field1": "value1",
    "field2": 123
  }'
```

#### Related Sections
- [Authentication Setup](../section7/auth.md)
- [Error Handling](../section9/errors.md)
- [Workflow: Birth Registration](../section4/birth.md)
```

---

**Document Version**: 2.0  
**Last Updated**: 25 January 2026  
**Next Review**: 25 April 2026  
**Maintained By**: MOSIP Documentation Team
