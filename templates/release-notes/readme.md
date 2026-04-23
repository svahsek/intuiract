# Release Notes YAML Schema System
## For Agents: Strict, Deterministic Release Notes Generation

**Status**: Production Ready  
**Last Updated**: 2026-02-16  
**Maintained By**: Documentation Team

---

## Overview

This system provides agents with a **strict, YAML-based schema** for generating release notes that are:

- ✅ **Precise**: Deterministic validation rules prevent ambiguity
- ✅ **Consistent**: All release notes follow the same structure
- ✅ **Portable**: YAML format works across projects and tools
- ✅ **Standards-Aligned**: References TGDP, Microsoft Style Guide, semantic versioning
- ✅ **Agent-Ready**: Designed for automated generation with human-in-the-loop validation

**Key Difference from Markdown Templates:**
- **Markdown templates** are for humans to write content (flexible but inconsistent)
- **YAML schemas** are for agents to structure data (strict but precise)

---

## Quick Start for Agents

### 1. You Need to Create Release Notes?

**Step 1:** Gather release information
- Version number (semantic versioning: X.Y.Z)
- Release date (YYYY-MM-DD format)
- Features added, bugs fixed, security updates, API changes, etc.
- Documentation links for each major change

**Step 2:** Choose your starting template

| Scenario | Template | Time to Fill |
|----------|----------|--------------|
| Patch release (1-3 bug fixes) | [`minimal-release-notes.yaml`](#minimal-example) | 15-30 min |
| Minor release (a few features) | Copy schema structure | 1-2 hours |
| Major release (many components) | [`comprehensive-release-notes.yaml`](#comprehensive-example) | 2-4 hours |

**Step 3:** Fill YAML structure with your data

**Step 4:** Run validation checks (see [Validation](#validation))

**Step 5:** Convert YAML to Markdown using rendering rules

### 2. How This System Works

```
┌─────────────────────────────────────────────┐
│  AGENT: Fill YAML Schema with Release Data  │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│  DETERMINISTIC VALIDATOR: Check All Rules   │
│  - Version format (X.Y.Z)                   │
│  - Required fields present                  │
│  - No placeholder text (TBD, TODO, FIXME)  │
│  - URLs are accessible                      │
│  - Migration guides for breaking changes    │
└────────────────┬────────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
   ✅ PASS          ❌ FAIL
        │                 │
        ▼                 ▼
   RENDER to          REPORT
   MARKDOWN           ERRORS
        │                 │
        ▼                 ▼
   OUTPUT:            Fix YAML,
   - release-notes.   re-validate
     schema.yaml      
   - RELEASE_NOTES.md
   - validation-report
```

---

## File Structure

### Schema & Rendering Rules

```
templates/release-notes/
├── release-notes-schema.yaml              # AUTHORITY: Complete schema with all fields
├── release-notes-rendering.yaml           # HOW-TO: Convert YAML → Markdown
├── agent-quick-reference.md              # QUICK: Copy-paste structure for agents
├── readme.md                              # THIS FILE: Architecture overview
└── examples/
    ├── comprehensive-release-notes.yaml   # FULL: All sections populated
    ├── minimal-release-notes.yaml         # MINIMAL: Bare minimum valid
    └── sample-output.md                   # OUTPUT: How rendered Markdown looks
```

### Related Files

```
rules/release-notes/
├── release-notes-rules.md                 # Human-readable validation rules
└── (Rules implemented as deterministic checks in schema)

standards/
├── references.yaml                        # External standards (TGDP, Microsoft, etc.)
├── content-types.yaml                     # Content-type registry for consumers
└── content-types/
  └── release-notes.yaml                 # Consumer-facing manifest for release notes
```

---

## Key Concepts for Agents

### 1. Source of Truth: YAML, Not Markdown

**Why YAML?**
- Structured data (machines can parse and validate)
- Schema validation (type checking, required fields)
- Reproducible rendering (same input → same output every time)
- Portable (works across repos, tools, languages)

**Workflow:**
```
AGENT WRITES YAML → VALIDATOR CHECKS → RENDERER CREATES MD → PUBLISHES MD
  (source)         (deterministic)    (templates)           (readable)
```

### 2. Deterministic Validation

These rules **must pass** before publishing:

| Rule | Type | Example |
|------|------|---------|
| Version format | Pattern | `✓ 2.1.0` vs `✗ 2.1` or `✗ 2.1.0-beta` |
| Required fields | Presence | All fields marked REQUIRED must exist |
| No placeholder text | Content | `✗ TBD`, `✗ TODO`, `✗ FIXME`, `✗ [UPDATE]` |
| Date format | Pattern | `✓ 2026-02-16` vs `✗ Feb 16` or `✗ 02/16/26` |
| Migration for breaking changes | Conditional | If `migrationRequired=true`, then `migrationNotes` must populated |
| Security best practices | Content | No internal emails, IPs, confidential code names in text |
| URL accessibility | External | All URLs must return HTTP 200; tested before submission |

See [`release-notes-schema.yaml`](release-notes-schema.yaml) for the complete validation rule list.

### 3. External Standards

These provide **guidance** (not hard requirements):

| Standard | Applies To | Link |
|----------|-----------|------|
| TGDP Release Notes Template | Content structure and section organization | https://github.com/thegooddocsproject/templates |
| Microsoft Style Guide | Writing style, voice, terminology | https://docs.microsoft.com/style-guide |
| Semantic Versioning 2.0 | Version number format | https://semver.org |

Example TGDP reference in schema:
```yaml
# Extract from schema showing external reference:
appliesTo:
  - "content.newFeatures"
  - "content.enhancedFeatures"
usage: "Use TGDP structure for organizing content sections"
```

---

## Using the Templates

### Template 1: Minimal Release Notes
**For:** Patch releases, security fixes, small updates (1-3 items)  
**Time:** 15-30 minutes  
**File:** `examples/minimal-release-notes.yaml`

```yaml
metadata:
  version: "2.1.1"
  releaseDate: "2026-02-16"
  releaseType: "Patch"
  productName: "My Product"

summary:
  headline: "Security Patch: Critical Vulnerability Fixed"
  paragraphs:
    - text: "Description of what was fixed..."
      sectionLabel: "Purpose"

content:
  securityUpdates:
    - title: "Vulnerability Title"
      severity: "Critical"
      # ... more fields
```

**Use When:**
- 🔴 Critical security patch needed
- 🐛 Single important bug fix
- ⚡ Performance improvement
- 📚 Small documentation update

### Template 2: Comprehensive Release Notes
**For:** Minor/major releases with multiple features, changes, and API updates  
**Time:** 2-4 hours  
**File:** `examples/comprehensive-release-notes.yaml`

**Includes sections:**
- New features (multiple with use cases)
- Enhanced features (before/after comparisons)
- Bug fixes (security and functional)
- API changes (new, modified, deprecated)
- Technical improvements (performance, reliability)
- Known issues and limitations
- Upgrade instructions
- Support resources

**Use When:**
- 🆕 New major features being released
- 🔄 Multiple API changes in one release
- 🚀 Quarterly feature release
- 🛠️ Infrastructure upgrades included

### Template 3: Quick Reference
**For:** Agents who need fast lookup of structure and rules  
**File:** `agent-quick-reference.md`

**Contains:**
- YAML structure overview (copy-paste template)
- List of required fields
- Validation rules summary
- Common mistakes and fixes
- Examples

---

## Validation <a name="validation"></a>

### Pre-Publication Checklist

**Before converting YAML to Markdown:**

```bash
# Automated checks (deterministic)
✓ Version is X.Y.Z format
✓ Release date is YYYY-MM-DD
✓ All REQUIRED fields populated
✓ No placeholder text (TBD, TODO, FIXME)
✓ URLs are absolute and accessible
✓ No internal emails or sensitive info
✓ Migration guides present for breaking changes
✓ Quantified metrics (not vague terms)

# Manual review (advisory)
✓ Headline is benefit-driven
✓ Language follows Microsoft Style Guide (no superlatives)
✓ Breaking changes explained clearly
✓ Security updates have workarounds
✓ Documentation links exist and work
```

### Run Validation

```python
# Pseudo-code for validation process
def validate_release_notes(yaml_file):
    doc = load_yaml(yaml_file)
    errors = []
    warnings = []
    
    # Deterministic checks (ERROR if fail)
    if not matches_semver(doc.metadata.version):
        errors.append("RULE-002: Version must be X.Y.Z format")
    
    if not matches_date_format(doc.metadata.releaseDate):
        errors.append("RULE-003: Date must be YYYY-MM-DD format")
    
    if contains_placeholders(doc):
        errors.append("RULE-005: Found TBD, TODO, or FIXME text")
    
    if missing_migration_guide_for_breaking_changes(doc):
        errors.append("RULE-006: Breaking change without migration guide")
    
    # Render to markdown
    if not errors:
        markdown = render_to_markdown(doc)
        return markdown, errors, warnings
    else:
        return None, errors, warnings
```

---

## Rendering YAML to Markdown

### Overview

The schema provides **deterministic rendering rules** that convert YAML data into publication-ready Markdown.

### Key Rendering Rules

| Rule | Input (YAML) | Output (Markdown) |
|------|--------------|-------------------|
| Date format | `2026-02-16` | `16 February 2026` |
| Version header | `metadata.version: "2.1.0"` | `# Release Notes`<br/>`**Version**: 2.1.0` |
| Headings hierarchy | Sections in `content` | H2 for sections (##)<br/>H3 for subsections (###)<br/>H4 for items (####) |
| API endpoints | `{method: "POST", endpoint: "/api/v2/oauth/clients"}` | `` `POST /api/v2/oauth/clients` `` |
| Severity badges | `severity: "Critical"` | `🔴 Critical` |
| Tables | Before/after data | Markdown table format |
| Links | `{title: "Guide", url: "https://..."}` | `[Guide](https://...)` |

Full rendering rules: see [`release-notes-rendering.yaml`](release-notes-rendering.yaml)

### Example: How YAML Becomes Markdown

**YAML Input:**
```yaml
metadata:
  version: "2.1.0"
  releaseDate: "2026-02-16"

summary:
  headline: "Self-Service OAuth2 Reduces Onboarding by 40%"
  keyBenefits:
    - "40% reduction in partner onboarding time"

content:
  newFeatures:
    - name: "Self-Service OAuth2 Client Management"
      description: "Partners can now create OAuth2 clients independently"
      documentation:
        title: "OAuth Client Management Guide"
        url: "https://docs.example.com/oauth"
```

**Rendered Markdown Output:**
```markdown
# Release Notes

**Version**: 2.1.0  
**Released**: 16 February 2026

## Release Summary

### Self-Service OAuth2 Reduces Onboarding by 40%

**Key Benefits**:
- ✨ 40% reduction in partner onboarding time

## What's New

### New Features

#### Self-Service OAuth2 Client Management

Partners can now create OAuth2 clients independently

- **Documentation**: [OAuth Client Management Guide](https://docs.example.com/oauth)
```

---

## Common Patterns

### Pattern 1: Security Patch Release

**Scenario:** Critical vulnerability discovered; need rapid release

**YAML Structure:**
```yaml
metadata:
  version: "X.Y.(Z+1)"  # Patch bump
  releaseType: "Patch"

content:
  securityUpdates:
    - title: "..."      # CVE-style title
      severity: "Critical"
      upgradeRecommended: true
      workaround: "..."  # Always include for urgent patches
```

### Pattern 2: Feature Release

**Scenario:** New features for users

**YAML Structure:**
```yaml
metadata:
  version: "X.(Y+1).0"  # Minor bump
  releaseType: "Minor"

summary:
  keyBenefits: [...]     # Highlight user benefits

content:
  newFeatures: [...]     # Primary focus
  enhancedFeatures: [...]  # Additional improvements
```

### Pattern 3: Breaking Changes

**Scenario:** API or behavior change; migration required

**YAML Rules:**
```yaml
# MANDATORY checks:
- migrationRequired: true              # ← MUST be true
  migrationNotes: "..."                # ← MUST be populated
  migrationGuide: {title, url}         # ← REQUIRED

# Will FAIL RULE-006 if absent
```

---

## Examples

### Minimal Example (Patch Release)

**File:** `examples/minimal-release-notes.yaml`

```yaml
metadata:
  version: "2.1.1"
  releaseDate: "2026-02-16"
  releaseType: "Patch"
  productName: "CRVS Platform"

summary:
  headline: "Security Patch: Critical SQL Injection Fixed"
  paragraphs: [...]

content:
  securityUpdates:
    - title: "SQL Injection in Partner Search"
      # ... (see file for full example)
```

**Stats:** ~50 lines YAML → ~2 page Markdown

### Comprehensive Example (Feature Release)

**File:** `examples/comprehensive-release-notes.yaml`

Includes:
- 3 new features
- 2 enhanced features
- 3 bug fixes
- 2 security updates
- 3 new APIs + 2 modified + 1 deprecated
- 3 performance improvements
- 2 known issues
- Complete upgrade instructions

**Stats:** ~500 lines YAML → ~8-10 page Markdown

### Copy-Paste Starter

**File:** `agent-quick-reference.md`

Quick reference with:
- Bare YAML structure
- Required fields checklist
- Validation rules summary
- Common mistakes

---

## Architecture Decisions

### Decision 1: YAML as Source, Not Markdown

**Why?**
- ✅ Machines can validate (schema constraints)
- ✅ Reproducible output (same data → same markdown)
- ✅ Portable (works across repos/tools)
- ✅ Versionable (diff shows actual changes, not formatting)

**Tradeoff:**
- Agents must think structurally (not "write prose")
- Learning curve (but Quick Reference helps)

### Decision 2: Strict Schema with Deterministic Validation

**Why?**
- ✅ Prevents ambiguity or incomplete data
- ✅ Catches errors before publishing
- ✅ "Shift left" quality assurance

**Rules enforced:**
- Required fields must be present
- Version format is X.Y.Z (not "v2.1" or "2.1.0-beta")
- Placeholder text is prohibited
- Breaking changes must include migration guides

### Decision 3: External Standards as Guidance, Not Sources

**Why?**
- ✅ TGDP templates are for humans; we adapt for agents
- ✅ Microsoft Style Guide informs writing quality; we validate it
- ✅ Semver is standard; we require it

**What we DON'T do:**
- ❌ Copy full TGDP template structure
- ❌ Implement MS style guide as code (subjective)
- ❌ Parse semver library; just pattern match X.Y.Z

### Decision 4: Rendering Rules Separate from Schema

**Why?**
- ✅ Schema defines WHAT data (the truth)
- ✅ Rendering rules define HOW to display (the presentation)
- ✅ Can change Markdown output without changing schema

**Example:**
- Schema says: `metadata.releaseDate: "2026-02-16"` (YYYY-MM-DD)
- Rendering rule says: Display as "16 February 2026" (DD Month YYYY)
- Agent doesn't need to worry; it's automatic

---

## For Agents: Implementation Checklist

**When creating release notes, follow this checklist:**

```
[ ] 1. GATHER DATA
    [ ] Release version (X.Y.Z semantic versioning)
    [ ] Release date (YYYY-MM-DD format)
    [ ] Features, fixes, API changes, etc.
    [ ] Documentation links for each major change
    [ ] CVE IDs for security updates
    
[ ] 2. CHOOSE TEMPLATE
    [ ] Patch (1-3 items) → minimal-release-notes.yaml
    [ ] Minor/Major (4+ items) → comprehensive-release-notes.yaml OR
    [ ] Quick copy → agent-quick-reference.md
    
[ ] 3. FILL YAML STRUCTURE
    [ ] Complete metadata section (all REQUIRED fields)
    [ ] Write compelling headline (10-150 chars, benefit-driven)
    [ ] Write 2-4 summary paragraphs
    [ ] Populate relevant content subsections
    [ ] Remove sections that don't apply
    
[ ] 4. VALIDATE ALL RULES
    [ ] Run validation checks against schema
    [ ] Fix all ERROR-level violations
    [ ] Address WARNING-level issues
    [ ] Verify no placeholder text (TBD, TODO, FIXME)
    [ ] Test all URLs (curl or browser)
    
[ ] 5. RENDER TO MARKDOWN
    [ ] Use release-notes-rendering.yaml rules
    [ ] Generate both YAML and Markdown outputs
    [ ] Review Markdown for readability
    [ ] Check links are clickable
    
[ ] 6. DELIVER
    [ ] YAML file (source of truth)
    [ ] Markdown file (published version)
    [ ] Validation report (list any issues)
```

---

## Troubleshooting

### "Validation failed: Version must be X.Y.Z"
**Problem:** Version is `2.1` or `2.1.0-beta` or `v2.1.0`  
**Solution:** Change to `2.1.0` (exactly 3 numbers separated by periods)

### "No placeholder text allowed"
**Problem:** Release notes contain `TBD`, `TODO`, `FIXME`, or `[UPDATE THIS]`  
**Solution:** Replace with actual content, or omit the section if content not ready

### "Date must be YYYY-MM-DD format"
**Problem:** Date is `Feb 16, 2026` or `2/16/26` or `16 Feb 2026`  
**Solution:** Use `2026-02-16` (ISO 8601 format in YAML)

### "Breaking change requires migration guide"
**Problem:** Feature with `migrationRequired: true` but no `migrationNotes`  
**Solution:** Add `migrationNotes` field explaining what users must do

### "URL not accessible"
**Problem:** Documentation links return 404 or timeout  
**Solution:** Test URL with `curl -I <url>` before submitting; ensure absolute URLs

---

## Related Documentation

- **Templates Directory:** `templates/release-notes/`
- **Validation Rules:** `rules/release-notes/release-notes-rules.md`
- **Standards Reference:** `standards/references.yaml` (external resources)
- **Documentation Standards:** [Integration Guide](../..)

---

## Support & Questions

**Agent Questions:**
- Schema structure: See `release-notes-schema.yaml` (field definitions + examples)
- Validation rules: See `release-notes-rendering.yaml` (all 40+ rules)
- Quick lookup: See `agent-quick-reference.md`

**Implementation Questions:**
- Rendering output: See `examples/sample-output.md`
- Comprehensive example: See `examples/comprehensive-release-notes.yaml`
- Minimal example: See `examples/minimal-release-notes.yaml`

---

**Last Checked:** 2026-02-16  
**Maintained By:** Documentation Team  
**Status:** ✅ Production Ready
