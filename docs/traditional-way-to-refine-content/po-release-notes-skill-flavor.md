# PO-Driven Release Notes Skill Flavor

Create a second skill as a Source-Driven flavor, not a Git-Driven flavor.

Best pattern: keep one common backend flow (schema, rules, validation, rendering), and swap only the ingestion/classification stage.

## Recommended design

1. Keep your current skill for Git/PR/releases as-is.
2. Add a new sibling skill for Product Owner content ingestion.
3. Reuse these steps unchanged from current skill:
   1. Load central manifest and standards
   2. Build final release YAML structure
   3. Run STR/CON/SEC validation
   4. Render Markdown outputs
4. Replace only the data collection/classification logic with PO content normalization.

## What the new flavor should do

1. Input first, not git first.
2. Accept PO narrative in a structured input file (or prompt fallback).
3. Normalize PO text into your canonical sections:
   - newFeatures
   - enhancedFeatures
   - bugFixes
   - securityUpdates
   - technicalImprovements
   - deprecations
   - knownIssues
4. Enforce the same rule set your current skill enforces.
5. Produce the same output files and naming pattern.

## Suggested behavior for the PO flavor

1. Step 0: Resolve Inputs (PO mode)
   1. Look for a dedicated PO input file.
   2. If missing, prompt for required fields and PO entries.
   3. Required fields remain:
      - version
      - release_type
      - product_name
      - release_date
      - plus at least one content entry
2. Step 1: Load standards from central registry (same as current).
3. Step 2: Normalize PO content
   1. If PO already labels categories, trust labels.
   2. If unlabeled, classify using the same rules engine and keywords.
   3. Convert each PO item into canonical entry format:
      - title
      - description
      - user_value
      - related_issues
      - migration_required
      - migration_notes
      - links
4. Step 3: Enrich
   1. Rewrite descriptions to your style constraints.
   2. Enforce issue formatting (JIRA/GH pattern).
   3. Generate summary headline and key benefits.
5. Step 4: Validate (same STR/CON/SEC rules).
6. Step 5: Render markdown (same rendering rules).
7. Step 6: Save YAML + Markdown + validation report.

## Key addition: input contract for PO

Use a small PO schema so content quality is stable before conversion.

### Example fields to require per PO item

1. title
2. description
3. user_impact
4. category (optional if classifier is enabled)
5. related_issue_refs (optional but recommended)
6. migration_notes (required when migration_required is true)
7. audience (optional)

## Two operating modes you should include

1. Strict mode
   1. No placeholders, no missing mandatory links, hard fail on validation errors.
2. Assist mode
   1. Allows warnings and auto-fills best-effort fields, but still blocks on structural errors.

## Why separate flavor is better than one large skill

1. Cleaner prompts and less branch complexity.
2. Lower risk of accidental git assumptions in PO workflows.
3. Easier CI integration (different pipelines for engineering-driven vs PO-driven release notes).
4. Easier future extension (e.g., Salesforce/Jira export ingestion).

## Minimal delta from your current skill

1. Remove Git data collection commands.
2. Remove base_tag/release_branch hard dependency in PO mode.
3. Add PO input schema and normalization/classification stage.
4. Keep validation and rendering exactly the same.

## Optional next step

Draft the full new skill file in the same style as your existing consumer skill so it can be dropped in as a second flavor immediately.
