# User Guide Writing Guidelines

## Purpose

Write or improve Markdown user guides so readers can understand a concept, complete a task,
learn a workflow, look up exact information, or resolve a problem without guessing. These
guidelines are Markdown-first; DITA, Diataxis, and The Good Docs Project are editorial sources,
not output formats.

## Topic-Purpose Rubric

Classify a section by its reader need before editing it:

| Reader need | Write as | Required quality | Rule |
|---|---|---|---|
| Understand why or how something works | Concept or explanation | Context, rationale, boundaries | `UG-CLS-001` |
| Complete a goal | Task or how-to | Goal, prerequisites, ordered steps, expected result | `UG-TPC-001` |
| Learn by doing | Tutorial | Objective, safe start, guided steps, checkpoints | `UG-TPC-002` |
| Look up exact facts | Reference | Precise terms, constraints, structured facts | `UG-TPC-003` |
| Fix a known problem | Troubleshooting | Symptom, cause, resolution, escalation | `UG-TPC-004` |

Sections can mix these purposes only when transitions are explicit. Do not hide a procedure in
conceptual prose or use a task section to explain broad background (`UG-TPC-005`).

## Language and Style

- Prefer active voice and direct instructions: “Select **Save**,” not “Save should be selected.”
  Preserve passive voice when the actor is unknown or irrelevant (`UG-PRO-001`).
- Lead tasks with an outcome and use one user action per numbered step where practical.
- Keep sentences concise without turning them into fragments. Split dense sentences when they
  contain multiple conditions, actions, or outcomes.
- Replace vague verbs such as “handle,” “use,” or “manage” with the specific user action when
  the page supplies enough context.
- Define unfamiliar terms on first use and use the same term consistently afterward.
- Explain user outcomes and decisions, not implementation details, unless the section is
  reference material.
- Use descriptive link text. Do not use “click here.”
- Flag dense multi-action sentences, but do not split necessary protocol or legal statements
  (`UG-PRO-005`).

## Safe Editing Boundary

Correct grammar, spelling, punctuation, Markdown formatting, and clear non-semantic wording.
Request review before changing product claims, metrics, prerequisites, expected outcomes,
terminology, section ownership, links, or instructions. Flag missing information rather than
inventing it.

## Editorial Sources

- [DITA technical content types](https://docs.oasis-open.org/dita/dita/v1.3/os/part2-tech-content/archSpec/technicalContent/technical-content.html)
- [Diataxis](https://diataxis.fr/) and its [how-to](https://diataxis.fr/how-to-guides/),
  [tutorial](https://diataxis.fr/tutorials/), [reference](https://diataxis.fr/reference/), and
  [explanation](https://diataxis.fr/explanation/) guidance
- [The Good Docs Project templates](https://www.thegooddocsproject.dev/template), including
  [how-to](https://www.thegooddocsproject.dev/template/how-to) and
  [troubleshooting](https://www.thegooddocsproject.dev/template/troubleshooting)
