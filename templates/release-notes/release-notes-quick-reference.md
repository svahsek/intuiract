# Release Notes YAML Template — Agent Quick Reference
# =====================================================
# For rapid access to schema structure and validation rules
# Full documentation: release-notes-schema.yaml

## QUICK REFERENCE: YAML Structure

```yaml
metadata:           # REQUIRED: Release identification
  version:          # REQUIRED: X.Y.Z semantic version
  releaseDate:      # REQUIRED: YYYY-MM-DD format
  releaseType:      # REQUIRED: Major|Minor|Patch|Hotfix|LTS
  productName:      # REQUIRED: Product name
  productNameAbbr:  # short form (e.g., CRVS)
  rolloutTimeline:  # deployment window
  targetAudience:   # [list of who should care]

summary:            # REQUIRED: High-level context
  headline:         # REQUIRED: Benefit-driven headline
  paragraphs:       # REQUIRED: [2-4 explanatory paragraphs]
    - text:         # paragraph content
      sectionLabel: # Purpose|ThemesAndFocus|Audience|DeploymentStrategy
  keyBenefits:      # [quantified benefits]

content:            # REQUIRED: Detailed release info
  newFeatures:      # [array of new capabilities]
  enhancedFeatures: # [array of improvements to existing features]
  bugFixes:         # [array of issues resolved]
  securityUpdates:  # [array of security patches]
  apiChanges:       # [new, modified, deprecated APIs]
  technicalImprovements: # [performance, reliability, infrastructure]
  knownIssues:      # [discovered issues in release]
  deprecations:     # [features being removed]
  upgradeInstructions: # [how to upgrade]
  knownLimitations: # [features not included]
  supportResources: # [documentation and support links]
```

---

## REQUIRED FIELDS (Must Not Be Empty)

| Field | Type | Validation | Example |
|-------|------|-----------|---------|
| `metadata.version` | string | Semantic versioning X.Y.Z | `2.1.0` |
| `metadata.releaseDate` | string | YYYY-MM-DD format | `2026-02-16` |
| `metadata.releaseType` | enum | Major/Minor/Patch/Hotfix/LTS | `Minor` |
| `metadata.productName` | string | Product name | `CRVS Platform` |
| `summary.headline` | string | 10-150 chars, benefit-driven | `Self-Service OAuth2 Reduces Onboarding by 40%` |
| `summary.paragraphs` | array | Min 2 paragraphs, max 4 | `[{text:..., sectionLabel:...}]` |
| `content` | object | At least ONE subsection must have content | See next section |

---

## CONTENT SUBSECTIONS (At Least One Required)

Choose which subsections apply to this release:

### New Features
```yaml
newFeatures:
  - name: "Feature Name"              # REQUIRED: Title Case
    description: "1-2 sentence..."     # REQUIRED: 20-300 chars
    useCase: "When/why..."             # REQUIRED
    keyCapabilities: [list]            # REQUIRED: 1-5 items
    documentation: {title, url}        # REQUIRED: link to docs
    relatedIssues: [#1234]             # GitHub issue numbers
```

### Enhanced Features
```yaml
enhancedFeatures:
  - name: "Feature Name"               # REQUIRED
    improvement: "What improved..."    # REQUIRED
    previousBehavior: "..."            # REQUIRED
    newBehavior: "..."                 # REQUIRED
    impactOnExistingUsers: "..."       # REQUIRED
    migrationRequired: false           # REQUIRED: boolean
    migrationNotes: "..."              # REQUIRED if migrationRequired=true
```

### Bug Fixes
```yaml
bugFixes:
  - bugTitle: "..."                    # REQUIRED
    description: "..."                # REQUIRED
    severity: Critical|High|Medium|Low # REQUIRED
    affectedVersions: [list]           # versions affected
    workaround: "..."                  # if applicable
    relatedIssues: [list]              # GitHub issues
```

### Security Updates
```yaml
securityUpdates:
  - title: "..."                       # REQUIRED: No sensitive details
    severity: Critical|High|Medium     # REQUIRED
    description: "Non-technical impact" # REQUIRED
    cveid: "CVE-2026-0001"            # CVE identifier
    cve_url: "https://..."             # CVE link
    affectedVersions: [list]           # which versions affected
    upgradeRecommended: true           # REQUIRED: boolean
    workaround: "Temporary fix..."     # if not immediately upgradeable
```

### API Changes
```yaml
apiChanges:
  newApis:                             # New endpoints in this release
    - endpoint: "/api/v2/..."          # REQUIRED: API path
      method: GET|POST|PUT|PATCH|DELETE # REQUIRED
      purpose: "What it does"          # REQUIRED
      keyParameters: [list]            # important params
      responseFormat: JSON|XML|CSV     # response type
      apiDocumentation: {format: OpenAPI, url: ...} # REQUIRED

  modifiedApis:                        # Changes to existing endpoints
    - endpoint: "/api/v1/..."          # REQUIRED
      changes: "What changed"          # REQUIRED
      backwardCompatible: true         # REQUIRED: boolean
      migrationRequired: false         # REQUIRED: boolean
      migrationGuide: {title, url}     # REQUIRED if migrationRequired=true

  deprecatedApis:                      # Being phased out
    - endpoint: "/api/v1/..."          # REQUIRED
      deprecatedSince: "1.9.0"         # version deprecated
      removalVersion: "3.0.0"          # REQUIRED: target removal version
      replacement: "Use /api/v2/..."   # REQUIRED: what to use instead
      migrationGuide: {title, url}     # REQUIRED: link to migration docs
```

### Technical Improvements
```yaml
technicalImprovements:
  performance:                         # Performance optimizations
    - area: "Component"                # what was optimized
      improvement: "How..."            # what changed
      quantifiedImpact: "40% faster"   # REQUIRED: must use %, x, ms, GB, etc.
      affectedComponents: [list]       # what components affected

  reliability:                         # Reliability/stability improvements
    - area: "..."
      improvement: "..."
      impact: "..."

  infrastructure:                      # Infrastructure/deployment changes
    - change: "..."
      description: "..."
```

### Known Issues
```yaml
knownIssues:
  - title: "..."                       # REQUIRED: issue title
    description: "..."                # impact
    workaround: "..."                  # REQUIRED: how to work around
    fixedIn: "2.1.1"                  # planned fix version
    issueTracker: "https://..."        # link to GitHub/JIRA
```

### Deprecations
```yaml
deprecations:
  - item: "OAuth 1.0"                  # REQUIRED: what's being removed
    reason: "Why..."                   # REQUIRED: why it's going away
    removalVersion: "3.0.0"            # REQUIRED: when it's removed
    replacement: "Use OAuth 2.0"       # REQUIRED: what to use instead
    migrationGuide: {title, url}       # REQUIRED: migration instructions
```

### Upgrade Instructions
```yaml
upgradeInstructions:
  estimatedDowntime: "3-5 minutes"    # maintenance window
  backupRequired: true                # REQUIRED: boolean
  upgradeSteps:
    - stepNumber: 1                   # REQUIRED: numeric order
      action: "What to do"            # REQUIRED: action description
      details: "More info..."         # optional: additional details
      commands:                       # optional: shell commands
        - "command 1"
        - "command 2"
  rollbackProcedure: "How to rollback..." # how to undo if it fails
```

---

## VALIDATION RULES (Deterministic Checks)

**ERROR (blocks release):**
- ❌ STR-001: Missing required sections (release header or at least one content section)
- ❌ STR-002: Version does not match semantic versioning (X.Y.Z or vX.Y.Z)
- ❌ STR-003: Release date not in accepted format (e.g., 25th March, 2026)
- ❌ CON-001: Missing overview paragraph
- ❌ CON-005: Breaking change has no migration notes or affected version range
- ❌ CON-006: Missing compatible modules table
- ❌ CON-007: Missing repository tags table
- ❌ CON-008: Missing feature docs or API docs links in documentation section
- ❌ SEC-001: Contains internal info (team names, personal names, internal URLs without context)
- ❌ SEC-003: Release name in header does not match productName and version fields

**WARNING (quality issues):**
- ⚠️ CON-002: Bug fix or known issue entry missing issue reference (GitHub #123 or JIRA ID)
- ⚠️ CON-003: Entry description below 10 words or above 200 characters
- ⚠️ CON-004: Section header present with no entries
- ⚠️ CON-009: Non-patch release missing test report link
- ⚠️ CON-010: Known issues section missing external tracker URL
- ⚠️ CON-011: Patch release contains new features
- ⚠️ CON-012: Feature release (Major/Minor) missing user stories table
- ⚠️ SEC-002: Contains comparative marketing language
- ⚠️ PRO-001–PRO-003: Prose style violations (applied via Vale on rendered Markdown)

**INFO (nice to have):**
- ℹ️ Comprehensive issue tracker links
- ℹ️ Detailed workarounds for known issues

---

## EXTERNAL REFERENCE STANDARDS

All text must follow these style guides:

| Standard | Applies To | Key Rules |
|----------|-----------|-----------|
| **TGDP Release Notes** | Structure of sections and content organization | Use TGDP template sections for release types |
| **Microsoft Style Guide** | All descriptive text (headlines, descriptions, paragraphs) | Active voice, simple words, no jargon, scannable |
| **Semantic Versioning** | `metadata.version` field | MAJOR.MINOR.PATCH format (https://semver.org) |

---

## AGENT WORKFLOW

**Step 1: Gather Release Data**
- Collect all changes from this release (features, fixes, API changes, etc.)
- Ensure you have dates, version numbers, documentation links

**Step 2: Populate YAML Structure**
- Fill in `metadata` section completely (all required fields)
- Fill in `summary` section (headline + 2-4 paragraphs + benefits)
- Populate relevant `content` subsections

**Step 3: Validate Against Rules**
- Run all STR-* checks: version format, date format, required sections
- Run all CON-* checks: overview present, issue references, description lengths, compatibility table, repository tags, documentation links, patch scope
- Run all SEC-* checks: no internal content, release name matches product and version
- Verify no placeholder text (TBD, TODO, FIXME, WIP)
- Ensure all URLs are absolute and accessible
- Full rule definitions: `.github/rules/release-notes/release-notes-rule.yaml`

**Step 4: Convert to Markdown**
- Use `release-notes-rendering.yaml` for rendering rules
- HDR-001: Convert date YYYY-MM-DD → DD Month YYYY format
- ENH-001: Use markdown tables for before/after comparisons
- API-001: Format API endpoints as `` `METHOD /path` ``
- BUG-001 / RSEC-001: Add emoji badges for severity levels (🔴🟠🟡🟢)
- RSEC-002: Link CVE IDs to https://nvd.nist.gov/vuln/detail/{cveid}
- IMP-001: Quantified metrics required — use %, x, ms, GB (never vague terms)

**Step 5: Deliver Outputs**
- YAML file: `release-notes-schema.yaml` (source)
- Markdown file: `RELEASE_NOTES.md` (published)
- Validation report: any errors/warnings encountered

---

## COMMON MISTAKES TO AVOID

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| Placeholder text | `[UPDATE THIS]`, `TBD`, `FIXME` | Actual content |
| Vague metrics | "50% faster", "much improved" | "50% faster (2s → 1s average)" |
| No migration guide | "We changed the API" | "Breaking change: See migration guide [link]" |
| Superlatives | "Revolutionary new feature" | "New feature reduces onboarding time by 40%" |
| Bad date format | "Feb 16" or "2026-02-16T10:30:00Z" | `2026-02-16` (YAML) → "16 February 2026" (Markdown) |
| Incomplete headlines | "New Features Added" | "Self-Service OAuth2 Management Reduces Onboarding by 40%" |
| Missing security details | "Security patch released" | "SQL Injection in /api/endpoint fixed; upgrade recommended; workaround available" |
| Broken links | `https://docs.example.com` | Test link works with curl/browser before submitting |

---

## EXAMPLE OUTPUTS

**Minimal Release (Patch - Single Fix):**
```yaml
metadata:
  version: "2.1.1"
  releaseDate: "2026-02-16"
  releaseType: "Patch"
  productName: "CRVS Platform"

summary:
  headline: "Security Patch: SQL Injection Fixed"
  paragraphs:
    - text: "This patch addresses a critical SQL injection vulnerability in the partner search API. All users should upgrade immediately."
      sectionLabel: "Purpose"

content:
  securityUpdates:
    - title: "SQL Injection in /api/partners/search"
      severity: "Critical"
      description: "Malicious search queries could bypass authentication"
      cveid: "CVE-2026-0842"
      cve_url: "https://nvd.nist.gov/vuln/detail/CVE-2026-0842"
      upgradeRecommended: true
```

**Comprehensive Release (Minor - Many Features):**
See: `.github/templates/release-notes/examples/comprehensive-release-notes.yaml`

---

## FILE LOCATIONS

These are YOUR canonical files. Always reference these:

| File | Purpose | Status |
|------|---------|--------|
| `.github/templates/release-notes/release-notes-schema.yaml` | **Schema authority** - complete field documentation | ✅ Source of truth |
| `.github/templates/release-notes/release-notes-rendering.yaml` | YAML → Markdown conversion rules | ✅ Rendering guide |
| `.github/templates/release-notes/examples/comprehensive-release-notes.yaml` | Full example with all sections | ✅ Reference example |
| `.github/rules/release-notes/release-notes-rules.md` | Human-readable validation rules | Original rules guide |

---

## HOW TO USE THIS GUIDE

1. **First Time?** Read this entire Quick Reference
2. **Creating Release Notes?** Copy the YAML structure above and fill in your data
3. **Validation Questions?** Check the "VALIDATION RULES" section
4. **Need Details?** Refer to `release-notes-schema.yaml` for expanded information
5. **Rendering Questions?** Check `release-notes-rendering.yaml`

---

## KEY PRINCIPLES

- **YAML is source of truth** — Store release notes in YAML format
- **Rendering is deterministic** — YAML → Markdown conversion follows strict rules
- **Validation must pass** — All ERROR rules must pass before publishing
- **External standards inform** — Microsoft, TGDP, semver provide guidance (not requirements)
- **Quantify everything** — Use concrete metrics, not vague terms
- **Help users upgrade** — Always include migration guides and workarounds

---

**Questions?** See `release-notes-schema.yaml` for full documentation with examples.
