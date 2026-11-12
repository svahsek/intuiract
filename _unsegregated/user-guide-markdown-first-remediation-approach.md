# Markdown-First User Guide Quality System

## Approach

The User Guide content type stays Markdown. DITA, Diataxis, and The Good Docs Project
become the editorial rubric, not XML, maps, or a required publishing architecture.

## What To Build First

Build a User Guide standards and remediation system for existing developer- or PO-authored
pages. Its purpose is to identify where content violates technical-writing principles and then
safely improve it.

The system reads an existing Markdown page and classifies each meaningful section by its job:

| Content job | Editorial model used | What good Markdown needs |
|---|---|---|
| Explain a concept or rationale | DITA concept + Diataxis explanation | Clear purpose, context, why it matters, boundaries; not a disguised procedure |
| Complete a real-world goal | DITA task + Diataxis how-to | Goal, prerequisites, ordered actionable steps, expected result, next step |
| Learn a workflow | Diataxis tutorial + TGDP tutorial | Learning objective, a safe starting state, guided steps, checkpoints, verification |
| Look up exact facts | DITA reference + Diataxis reference | Precise terminology, tables/lists, constraints, examples; no narrative padding |
| Resolve a failure | DITA troubleshooting + TGDP troubleshooting | Symptom, likely cause, resolution, escalation or next action |

A page may contain several types. The skill does not make authors create separate files or impose
a DITA structure. It uses the classification to assess whether each Markdown section is doing its
job well.

## Audit Dimensions

1. **Purpose and structure:** Does the page and each major section have one clear user purpose?
   Are explanation, procedure, reference, and troubleshooting content mixed in a confusing way?
2. **Task quality:** Are steps atomic, ordered, imperative, and testable? Are prerequisites placed
   before actions? Is there an expected outcome?
3. **Language and readability:** Detect passive voice, vague verbs, weak headings, long dense
   sentences, choppy fragments, filler, unexplained jargon, inconsistent terminology, and
   descriptions that explain implementation rather than user outcome.
4. **Technical-writing quality:** Check scanability, heading hierarchy, lists, link text, code and
   UI formatting, warnings and notes, accessibility, and placeholders.
5. **Content safety:** Flag unclear claims, missing facts, unsupported metrics, ambiguous ownership,
   or a required decision. Do not invent product behavior or silently change meaning.

## Remediation Behavior

- **Auto-fix:** spelling, punctuation, grammar, obvious passive-to-active rewrites that preserve
  meaning, Markdown/list formatting, and heading-level formatting where anchors are unaffected.
- **Review-required:** restructure a section, split an overloaded section, rewrite a task or feature
  claim, add prerequisites or expected results, alter terminology, or change links.
- **Blocked/uncertain:** a fact, product behavior, metric, target audience, or missing procedure
  must be supplied by a developer or product owner.

The workflow remains: **audit -> suggest -> apply approved changes -> re-audit**. Every finding
cites the relevant User Guide rule or guideline and is reported line by line.

## User Guide Content-Type Assets

- Registry entry and manifest
- Markdown-oriented User Guide writing guidelines that encode the topic-purpose rubric
- A concise structure/rules file for section-purpose checks, task checks, language/style, links,
  and safety
- A User Guide remediation consumer skill, modeled on the Features review skill but using the
  User Guide rubric
- Minimal and comprehensive Markdown examples showing mixed concept/task/reference content correctly

## Deliberate Non-Goals

Do not start with release-evidence routing, guide maps, XML, or automatic whole-guide generation.
Evaluate those later, after the quality workflow proves useful.
