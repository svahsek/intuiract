# Product vs Standards Strategy

Short answer: do both, in sequence.

Best path is a staged strategy:

1. Keep standards, rules, skills, and agent workflows as the source of truth now.
2. Productize only the stable core into a tool once patterns are proven.
3. Keep agentic logic for judgment-heavy tasks even after tooling.

## Why this is the strongest approach

1. Your current assets are high-value IP: taxonomy, precedence, validation, templates.
2. If you jump to product too early, you risk hard-coding unstable assumptions.
3. If you never productize, adoption in organizations will stay limited and manual.

## Practical decision rule

1. Stay standards-first until you see repeatability across 3 to 5 release cycles.
2. Productize when:
   1. Inputs are consistent enough
   2. Precedence conflicts are known and documented
   3. Validation rules are stable
   4. Teams ask for CLI and CI integration
3. Keep agent mode for:
   1. Ambiguous classification
   2. Narrative quality improvements
   3. Exception handling and remediation suggestions

## What to build as product first

1. Extraction
2. Reconciliation
3. Validation
4. Artifact generation
5. CI command surface

## What to keep non-productized for longer

1. Advanced narrative writing choices
2. Cross-content editorial judgment
3. New content type experimentation

## Recommendation

1. Continue current approach now.
2. Start a thin CLI wrapper around the stable parts only.
3. Treat tool as execution engine; treat standards + rules + skills as the brain.

It is worthwhile to turn this into software, but not as a replacement. Build it as an operational layer on top of your existing standards architecture.
