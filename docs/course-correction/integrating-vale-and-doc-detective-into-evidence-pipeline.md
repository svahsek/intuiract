# Integrating Vale and Doc Detective into an Evidence-First Documentation Pipeline

Yes, you can use both, and they fit this approach very well.

Use them as evidence producers in the pipeline, not as separate lint tools.

## How each tool fits

### Vale

1. Purpose: style, terminology, tone, consistency, banned words, structure checks
2. Best stage: during PR review and before release publish
3. Output to capture: violations by rule id, file, severity, and pass or fail summary

### Doc Detective

1. Purpose: validate documentation procedures and examples by running doc tests
2. Best stage: pre-release validation and nightly checks
3. Output to capture: scenario pass or fail, broken steps, command mismatch, environment-specific failures

## Recommended integration pattern

### 1) Authoring stage

1. Run Vale on changed docs files in PR
2. Block merge only on high severity rules
3. Store result summary as an evidence event

### 2) Pre-release stage

1. Run full Vale scan on release docs set
2. Run Doc Detective tests for user guide and installation paths
3. Capture both outputs into the release evidence file

### 3) Release generation stage

1. Generate release notes and other pages from canonical evidence
2. Re-run Vale on generated markdown
3. Add final quality status to validation report

## Where to place them in the architecture

1. Extraction layer: pull GitHub artifacts
2. Evidence layer: add doc-quality events from Vale and Doc Detective
3. Reconciliation layer: combine engineering evidence and doc evidence
4. Validation layer: enforce gates using both rule sets
5. Rendering layer: produce publishable content

## Add these event types to evidence syntax

1. doc_style_checked
2. doc_test_executed
3. doc_quality_gate_decision

## Example fields for each event

1. tool_name
2. run_id
3. target_scope
4. pass_count
5. fail_count
6. severity_high_count
7. report_location
8. gate_result

## Suggested release gates

1. Block release if Doc Detective has critical failures in installation or user guide flows
2. Block release if Vale has policy-critical errors
3. Allow warnings for minor style issues, but include them in report

## Why this is valuable

1. Documentation output becomes testable and auditable
2. Last-minute manual QA of docs is reduced
3. Trust in generated content improves for organization-wide use

## Possible next step

Draft a concrete CI layout for this repo:

1. PR workflow with Vale
2. Release workflow with Vale and Doc Detective
3. Evidence file update step that records both tool results into the canonical model
