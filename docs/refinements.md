# Next Set of Refinements

Planning notes for the next round of changes to this repo. Each item below is a distinct refinement to work through.

## 1. Consumer skill input handling + rendering clarity

- **Problem**: Consumer skills currently take an input file. Instead, the input could be defined inside the skill itself.
- **Action**: Define a clean flow diagram showing what is ingested, and at what stage of the pipeline.
- **Action**: The rendering template must clearly define:
  - Table columns
  - Order of topics
  - Order of columns

## 2. Add PO release notes as an input source

- Take Product Owner (PO) release notes into consideration as an additional input to the pipeline.

## 3. American vs. British English validation layer

- Add a validation layer that checks for American English vs. British English spelling.
- We maintain American English spelling only — this layer should flag any British spellings.

## 4. Broken link checking via lychee

- Integrate [lychee](https://github.com/lycheeverse/lychee), a command-line broken-link checker.
- Use it to check all links in generated/maintained content and produce a report.

## 5. Signal file generation from the payload YAML

- Generate a signal file derived from the generated content's payload YAML.
- The signal file should indicate which content types need an update, based on the generated release notes.
