# Release Notes YAML System — Architecture Summary

## System Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                   AGENT WORKFLOW                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. GATHER             2. CHOOSE             3. FILL YAML  │
│  ─────────────────────────────────────────────────────────  │
│  - Version (X.Y.Z)     - Patch release?       metadata:    │
│  - Release date        - Feature release?       version:    │
│  - Features/fixes      - Use template           releaseDate │
│  - API changes         - Quick ref?             releaseType │
│  - Docs links          - Examples               productName │
│  - CVEs (if security)  - Start filling       summary:      │
│                                                headline:    │
│  4. VALIDATE           5. RENDER             6. OUTPUT    │
│  ──────────────────────────────────────────────────────── │
│  - Version X.Y.Z? ✓    - YAML → Markdown      - YAML file │
│  - Required fields? ✓  - Date conversion      - .md file  │
│  - No TBD/TODO? ✓      - Heading levels       - Report    │
│  - No broken URLs? ✓   - Emoji badges        │
│  - Migrations? ✓       - Link formatting      │
│                        - Style consistency    │
│                                              │
└──────────────────────────────────────────────────────────────┘
```

## Core Components

```
release-notes-schema.yaml (source of truth)
│
├─ metadata section
│  ├─ version (semantic versioning X.Y.Z)
│  ├─ releaseDate (YYYY-MM-DD)
│  ├─ releaseType (Major/Minor/Patch/Hotfix/LTS)
│  └─ product info
│
├─ summary section
│  ├─ headline (benefit-driven)
│  ├─ paragraphs (2-4 paragraphs)
│  └─ keyBenefits (quantified)
│
├─ content section
│  ├─ newFeatures
│  ├─ enhancedFeatures
│  ├─ bugFixes
│  ├─ securityUpdates
│  ├─ apiChanges (new/modified/deprecated)
│  ├─ technicalImprovements
│  ├─ knownIssues
│  ├─ deprecations
│  ├─ upgradeInstructions
│  ├─ knownLimitations
│  └─ supportResources
│
└─ validation rules (deterministic checks)
   ├─ RULE-001: Required sections
   ├─ RULE-002: Version X.Y.Z format
   ├─ RULE-003: Date YYYY-MM-DD format
   ├─ RULE-005: No placeholder text
   ├─ RULE-006: Migration guides for breaking changes
   ├─ RULE-008: No superlatives
   ├─ RULE-011: Documentation links required
   └─ RULE-013: No sensitive content
```

## Data Flow

```
┌──────────────────┐
│  AGENT: Create   │
│  Release Notes   │
│      YAML        │
└────────┬─────────┘
         │
         ▼
┌──────────────────────────────────┐
│  SCHEMA VALIDATOR               │
│  ─────────────────────────────── │
│  • Parse YAML structure          │
│  • Check required fields         │
│  • Validate data types           │
│  • Run deterministic checks      │
│  • Verify URLs accessible        │
└────────┬────────────────┬────────┘
         │                │
    ✅ PASS          ❌ FAIL
         │                │
         ▼                ▼
     ┌───────┐      ┌──────────┐
     │RENDER │      │REPORT    │
     │TOMD   │      │ERRORS    │
     └───┬───┘      └──────────┘
         │                │
         ▼                ▼
  ┌────────────┐     AGENT
  │ RENDERING  │     FIXES
  │  ENGINE    │     YAML
  │            │       │
  │ - Date fmt │       └─────┐
  │ - Headings │             │
  │ - Tables   │         RE-VALIDATE
  │ - Badges   │             │
  │ - Links    │             │
  └────┬───────┘             │
       │<────────────────────┘
       ▼
  ┌──────────────────┐
  │  OUTPUT READY    │
  │  ────────────────│
  │ • release-notes. │
  │   schema.yaml    │
  │ • RELEASE_      │
  │   NOTES.md       │
  │ • validation-    │
  │   report.txt     │
  └──────────────────┘
```

## Reference Standards Integration

```
┌─────────────────────────────────────────────────────────┐
│  EXTERNAL REFERENCE STANDARDS                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  The Good Docs Project (TGDP)                           │
│  https://github.com/thegooddocsproject                  │
│  └─ Applied to: Content structure, section types       │
│                                                         │
│  Microsoft Style Guide                                  │
│  https://docs.microsoft.com/style-guide                │
│  └─ Applied to: Voice, tone, clarity, word choice      │
│                                                         │
│  Semantic Versioning 2.0.0                              │
│  https://semver.org                                     │
│  └─ Applied to: Version number format (X.Y.Z)          │
│                                                         │
│  ► Standards inform WHAT we validate                    │
│  ► Schema enforces HOW strictly we validate             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## File Organization

```
.github/
└── templates/
    └── release-notes/
        ├── readme.md                        ← Start here (architecture)
        ├── agent-quick-reference.md         ← Fastest way to start
        ├── release-notes-schema.yaml        ← AUTHORITY (all fields)
        ├── release-notes-rendering.yaml     ← HOW to render YAML→MD
        └── examples/
            ├── minimal-release-notes.yaml   ← Copy for patch releases
            ├── comprehensive-release-notes.yaml  ← Full feature release
            └── sample-output.md             ← Rendered markdown example

.github/
└── rules/
    └── release-notes/
        └── release-notes-rules.md           ← Human-readable rules

.github/
└── standards/
    ├── references.yaml                      ← External standards index
    ├── content-types.yaml                   ← Shared content-type registry
    └── content-types/
        └── release-notes.yaml               ← Consumer-facing release-notes manifest
```

## Key Design Principles

### 1. YAML as Single Source of Truth
- Structured, machine-readable format
- Schema validation catches errors early
- Reproducible rendering (same input = same output)
- Portable across projects and tools

### 2. Deterministic Validation
- Rules are explicit and verifiable
- Pattern matching (not subjective)
- All-or-nothing enforcement (no ambiguity)
- Clear error messages for failures

### 3. External Standards as Guidance
- TGDP provides structure, not templates
- Microsoft provides style principles, not rules
- Semver is industry standard, we enforce it
- We layer local policy on top, don't replace standards

### 4. Separation of Concerns
- Schema = WHAT data (content authority)
- Rendering = HOW to display (presentation)
- Validation = WHETHER to proceed (quality gate)

### 5. Agent-Friendly Design
- Structured templates (not free-form prose)
- Clear validation messages (not vague errors)
- Copy-paste examples (not lengthy documentation)
- Quick reference card (80% of needs, 20% of reading)

## Usage Patterns

### Pattern 1: Quick Patch Release (15 min)
```
Agent: "I need to release a security patch"
Step 1: Copy minimal-release-notes.yaml
Step 2: Fill metadata + summary + security updates
Step 3: Run validator
Step 4: Render to Markdown
Output: RELEASE_NOTES.md
```

### Pattern 2: Feature Release (2-4 hours)
```
Agent: "I have 5 new features + 3 API changes to document"
Step 1: Copy comprehensive-release-notes.yaml
Step 2: Fill all sections
Step 3: Run validator (may have strict failures at first)
Step 4: Fix violations (add migration guides, quantify metrics)
Step 5: Re-validate
Step 6: Render to Markdown
Output: RELEASE_NOTES.md + validation report
```

### Pattern 3: Quick Lookup (5 min)
```
Agent: "What's the headline character limit?"
Resource: agent-quick-reference.md → 10-150 characters
Agent: "Is my version format correct?"
Resource: release-notes-schema.yaml → pattern ^X.Y.Z
```

## Validation Rules Summary

### ERROR (Must Fix)
- ❌ Invalid version format (not X.Y.Z)
- ❌ Invalid date format (not YYYY-MM-DD)
- ❌ Missing required fields
- ❌ Contains placeholder text (TBD, TODO, FIXME)
- ❌ Breaking changes without migration guide
- ❌ New features without documentation
- ❌ Contains sensitive information

### WARNING (Should Fix)
- ⚠️ Vague metrics ("faster" instead of "40% faster")
- ⚠️ Marketing superlatives
- ⚠️ Missing issue tracker links

### INFO (Nice to Have)
- ℹ️ Comprehensive examples
- ℹ️ Complete support resources

## When to Use Each Resource

| Question | Resource | Time |
|----------|----------|------|
| How does the system work? | readme.md | 10 min |
| I need to fill the YAML | agent-quick-reference.md | 5 min |
| I need full field docs | release-notes-schema.yaml | 20 min |
| How do I render? | release-notes-rendering.yaml | 10 min |
| Show me a real example | examples/comprehensive-release-notes.yaml | 20 min |
| I just need a patch | examples/minimal-release-notes.yaml | 5 min |
| What are the rules? | release-notes-rules.md | 15 min |

## System Benefits

✅ **For Agents:**
- Clear, structured format (no ambiguity)
- Deterministic validation (pass/fail, not subjective)
- Copy-paste examples (fast start)
- Rendering is automatic (no manual formatting)

✅ **For Users:**
- Consistent release notes (same structure every time)
- Better organization (predictable section order)
- Clearer information (benefit-driven, not jargon-heavy)
- Accessible docs (always linked and verified)

✅ **For System:**
- Portable across repos (YAML works everywhere)
- Versionable (diff shows actual changes)
- Machine-processable (can extract data, generate reports)
- Standards-aligned (TGDP, Microsoft, semver)

---

**Status:** ✅ Production Ready  
**Last Updated:** 2026-02-16  
**Architecture Version:** 1.0.0
