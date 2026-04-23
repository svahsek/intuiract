# Evidence Signals: Vale and Doc Detective Integration

## Purpose

Define how external quality signals (Vale style linting, Doc Detective content validation) integrate into the release evidence model as observable fields.

This enables release notes to be gated not just on code quality (tests, security scans) but on documentation quality as well.

**Key Insight**: Documentation quality is evidence of shipping confidence. A change without good docs suggests incomplete understanding.

---

## Integration Pattern

### Detection Layer

**Tools that generate signals**:
1. **Vale** → Style linter; produces findings by rule category (prose, formatting, terminology)
2. **Doc Detective** → Documentation validator; checks links, code examples, references
3. **CI/CD Build System** → Build logs, test results
4. **GitHub Checks** → CODEOWNERS reviews, branch protection rules

### Capture Layer

**Where signals are recorded**:
- Vale findings → PR comment (via CI workflow) OR manual scan
- Doc Detective findings → PR comment (via CI workflow)
- Both → Manually transcribed to `release_evidence.yaml` under `evidence.documentation_quality` and `evidence.content_validation`

### Reconciliation Layer

**How signals affect confidence**:
- Vale critical findings (5+) → Deduct confidence points
- Doc Detective broken links → Deduct confidence points
- Doc Detective invalid code examples → Deduct confidence points
- No findings → Bonus confidence

### Rendering Layer

**How signals appear in release notes**:
- Quality flags included in validation report
- If critical findings → Item moved to unresolved[]
- If warnings → Item included with confidence indicator

---

## Vale Integration

### What Vale Does

Vale is a rule-based prose linter that checks documentation for style consistency, terminology, and formatting.

**Example Vale Rules**:
- Sentence length > 30 words → Warning
- Passive voice usage → Info/Warning
- Incorrect terminology (e.g., "login" vs "log in") → Error
- Missing Oxford comma → Info
- Contractions in formal docs → Error

### Vale Signals Captured

For each change with documentation updates, run Vale and capture:

```yaml
evidence:
  documentation_quality:
    vale_scan_date: "2026-04-23"
    vale_scope: "release-notes.md"  # which files scanned
    
    critical_findings:  # Errors that block publication
      count: 0
      examples: []  # or: [{rule: "ProselintTerminology", message: "Use 'log in' not 'login'", line: 42}]
    
    warning_findings:  # Warnings that should be fixed
      count: 2
      examples:
        - rule: "ProselintSentenceLength"
          message: "Sentence too long (45 words)"
          line: 18
        - rule: "TerminologyTechnical"
          message: "Use 'API' not 'api' (capitalization)"
          line: 56
    
    info_findings:  # Suggestions (non-blocking)
      count: 1
      examples:
        - rule: "ProselintPassive"
          message: "Consider active voice"
          line: 9
    
    style_conformance: "82%"  # 82 of 100 style checks pass
```

### Vale Configuration (Example)

Create `.vale.ini` at repo root:

```ini
[*.md]

# Critical (errors block publication)
ProselintTerminology = error
ProselintConfusables = error
Google.Colons = error
Google.PassiveVoice = warning
Google.Semicolons = error

# Warnings (should fix but not blocking)
Google.Quotes = warning
Google.Exclamation = warning
ProselintSentenceLength = warning

# Info (suggestions)
Google.Parentheses = suggestion
```

### Integration Workflow

1. Developer writes release notes (or PR description)
2. CI/CD runs Vale: `vale release-notes.md`
3. Vale produces findings in JSON or table format
4. CI workflow posts findings as PR comment
5. Developer fixes critical/warning findings before merge
6. Final Vale scan (post-merge, pre-release) confirms clean
7. Vale results transcribed to `release_evidence.yaml` → `documentation_quality` section

### Example: Pre-Release Vale Check

```bash
# Pre-release validation
vale release-notes.md --output JSON > vale-results.json

# Integrate results into release evidence
# Extract critical findings count
# If count > 0: Move item to unresolved[]
# If count == 0 AND warnings < 5: Proceed
```

---

## Doc Detective Integration

### What Doc Detective Does

Doc Detective validates documentation content by checking:
1. **Links** → Do they work? 404s? Anchors valid?
2. **Code examples** → Can they run? Do they produce expected output?
3. **References** → Do referenced files exist?
4. **Placeholders** → No TBD/TODO/FIXME left behind
5. **Cross-references** → Links within doc set valid

### Doc Detective Signals Captured

```yaml
evidence:
  content_validation:
    doc_detective_scan_date: "2026-04-23"
    doc_detective_scope: "docs/release-notes/v1.2.3/"
    
    link_validation:
      total_links_checked: 24
      valid_links: 22
      broken_links: 2
      broken_link_examples:
        - url: "https://docs.example.com/api/v1/users"
          status: 404
          found_in: "release-notes.md:line 45"
        - url: "#missing-anchor"
          status: "anchor_not_found"
          found_in: "release-notes.md:line 78"
      verdict: "2 broken links require fixing"
    
    code_example_validation:
      total_examples: 5
      valid_examples: 4
      invalid_examples: 1
      invalid_example_details:
        - example_id: "oauth2-flow"
          language: "javascript"
          error: "ReferenceError: fetch is not defined"
          fix_required: true
          found_in: "release-notes.md:line 102"
      verdict: "1 example needs fixing"
    
    reference_validation:
      total_references: 8
      valid_references: 8
      missing_references: 0
      verdict: "All file references valid"
    
    placeholder_validation:
      placeholders_found: 0
      examples: []
      verdict: "No TBD/TODO/FIXME detected"
    
    cross_reference_validation:
      total_cross_refs: 12
      valid_cross_refs: 12
      invalid_cross_refs: 0
      verdict: "All internal links valid"
    
    overall_verdict: "PASS with warnings"
    critical_issues: 0
    non_critical_issues: 1  # 1 broken link (should be fixed)
```

### Doc Detective Configuration (Example)

Create `doc-detective.yml` at repo root:

```yaml
# doc-detective.yml
tests:
  # Check all links
  - name: "Validate all links"
    command: "check-links"
    options:
      scope: "docs/**/*.md"
      exclude: ["node_modules/**", ".git/**"]
      follow_external: false  # don't check external links
  
  # Validate code examples
  - name: "Validate code examples"
    command: "validate-code"
    options:
      scope: "docs/**/*.md"
      languages: ["javascript", "python", "bash"]
      timeout: 5000ms
  
  # Check for placeholders
  - name: "No TBD/TODO/FIXME"
    command: "check-placeholders"
    options:
      patterns: ["TBD", "TODO", "FIXME", "XXX"]
  
  # Validate cross-references
  - name: "Cross-reference validation"
    command: "validate-references"
    options:
      scope: "docs/**/*.md"
      exclude_external: true
```

### Integration Workflow

1. Developer writes release notes
2. CI/CD runs Doc Detective: `doc-detective run doc-detective.yml`
3. Doc Detective produces validation report (JSON)
4. CI workflow posts findings as PR comment
5. Developer fixes broken links and invalid examples before merge
6. Final Doc Detective scan confirms clean
7. Results transcribed to `release_evidence.yaml` → `content_validation` section

### Example: Pre-Release Doc Detective Check

```bash
# Pre-release validation
doc-detective run doc-detective.yml --output json > doc-detective-results.json

# Parse results
# If critical_issues > 0: Move item to unresolved[]
# If non_critical_issues == 0 AND all_verdicts=="PASS": Proceed
```

---

## Signal Integration into Confidence Scoring

### Confidence Penalty Matrix

| Signal | Finding Count | Confidence Penalty | Severity |
|--------|---------------|--------------------|----------|
| **Vale Critical** | 1-2 | -5 per finding | Error (blocks) |
| **Vale Critical** | 3+ | -20 (total) | Error (blocks) |
| **Vale Warning** | 1-3 | -2 per finding | Warning |
| **Vale Warning** | 4+ | -10 (total) | Warning |
| **Doc Detective Broken Link** | 1-2 | -5 per link | Error |
| **Doc Detective Broken Link** | 3+ | -20 (total) | Error |
| **Doc Detective Invalid Code** | 1-2 | -5 per example | Error |
| **Doc Detective Placeholder** | 1+ | -20 (total) | Error (blocks) |
| **Doc Detective Invalid Reference** | 1+ | -10 | Warning |

### Confidence Blocking Rules

```yaml
validation_gates:
  
  vale_critical_findings:
    condition: "critical_findings.count >= 1"
    action: "Move item to unresolved[]; operator review required"
    reason: "Style errors indicate incomplete or careless documentation"
  
  doc_detective_placeholders:
    condition: "placeholder_validation.placeholders_found >= 1"
    action: "Move item to unresolved[]; operator review required"
    reason: "TBD/TODO left behind indicates incomplete documentation"
  
  doc_detective_broken_links:
    condition: "link_validation.broken_links >= 2"
    action: "Move item to unresolved[]; fix links before release"
    reason: "Broken links harm user experience and SEO"
  
  doc_detective_invalid_code:
    condition: "code_example_validation.invalid_examples >= 1"
    action: "Move item to unresolved[]; fix or remove examples"
    reason: "Invalid code examples mislead users and harm credibility"
```

---

## Automated Signal Capture Workflow

### GitHub Actions Workflow (Example)

Create `.github/workflows/vale-doc-detective.yml`:

```yaml
name: Vale + Doc Detective
on: [pull_request]

jobs:
  lint-and-validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Run Vale linter
      - uses: errata-ai/vale-action@reviewdog
        with:
          files: "docs/**/*.md, *.md"
          vale_flags: "--output JSON"
      
      # Run Doc Detective
      - name: Doc Detective
        run: |
          npm install -g @microsoft/doc-detective
          doc-detective run doc-detective.yml --output json > doc-detective-results.json
      
      # Comment on PR with findings
      - name: Post findings to PR
        uses: actions/github-script@v6
        if: always()
        with:
          script: |
            const vale = require('./vale-results.json');
            const docDetective = require('./doc-detective-results.json');
            
            let comment = '## Quality Signals\n\n';
            comment += `**Vale**: ${vale.critical.length} critical, ${vale.warnings.length} warnings\n`;
            comment += `**Doc Detective**: ${docDetective.critical_issues} critical issues\n\n`;
            
            if (vale.critical.length > 0) {
              comment += '⚠️ Critical style issues found\n';
            }
            if (docDetective.broken_links > 0) {
              comment += '⚠️ Broken links detected\n';
            }
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

---

## Evidence Signals Manifest

### Complete Signal Set

Every release item can have these signal fields:

```yaml
evidence_item:
  
  # Base evidence (from reconciliation)
  intent: {...}
  change: {...}
  evidence: {...}
  shipped: {...}
  
  # NEW: Quality signals
  quality_signals:
    
    # Code quality (pre-existing)
    code_review_status: "approved"  # approved, pending, rejected
    build_status: "passed"  # passed, failed, skipped
    security_scan_status: "passed"  # passed, warnings, failed
    
    # Documentation quality signals (NEW)
    documentation_signals:
      vale_conformance: 82  # percentage
      vale_critical_count: 0
      vale_warning_count: 2
      
    # Content validation signals (NEW)
    content_signals:
      links_valid: true
      broken_link_count: 0
      code_examples_valid: true
      invalid_code_count: 0
      placeholders_found: false
      placeholder_count: 0
      cross_references_valid: true
      invalid_reference_count: 0
    
    # Summary
    documentation_quality_score: 88  # 0-100
    overall_quality_pass: true
```

---

## Debugging & Troubleshooting

### Low Confidence Due to Quality Signals

**Symptom**: Item has good code quality but low confidence score

**Diagnosis**:
1. Check Vale results: Are there critical style issues?
2. Check Doc Detective results: Are there broken links or placeholders?
3. Check documentation_quality_score

**Recovery**:
1. Fix Vale critical issues (errors block release)
2. Fix Doc Detective broken links and invalid code
3. Run final scans
4. Re-score confidence

### Vale Not Running in CI

**Symptom**: Vale finds issues locally but CI doesn't catch them

**Diagnosis**:
1. Check `.vale.ini` exists in repo root
2. Check GitHub Actions workflow includes Vale step
3. Check Vale rule files are checked in

**Recovery**:
1. Add `.vale.ini` to root
2. Add Vale action to workflow
3. Test locally: `vale .`

### Doc Detective Timeout

**Symptom**: Doc Detective checks hang on links

**Diagnosis**:
1. Check timeout setting in `doc-detective.yml`
2. Check for circular references in documentation
3. Check for external links being checked (should be disabled)

**Recovery**:
1. Increase timeout in config
2. Exclude external links: `follow_external: false`
3. Run with `--verbose` to see which links hang

---

## Acceptance Criteria for Week 3

✓ Vale integration workflow defined  
✓ Doc Detective integration workflow defined  
✓ Signal capture fields added to release-evidence schema  
✓ Confidence penalty rules documented  
✓ GitHub Actions workflow example created  
✓ Example signal findings documented  

---

## Next Steps (Week 4)

1. Create pilot runbook that includes running Vale + Doc Detective
2. Execute pilot with real release to test signal capture
3. Document signal findings in pilot results
4. Validate confidence scoring with signals applied

---

## Version History

- **v1.0.0** (2026-04-23): Initial integration model for Vale and Doc Detective signals
