---
name: lanyan-experience-intake
description: Use when the user asks to summarize lessons, record a bug pattern, improve the personal skill library, prevent a repeated problem, or turn project knowledge into reusable CC Switch/Claude skills. Consolidate experience into the smallest relevant LanYan skill without duplicating triggers or bloating context.
---

# LanYan Experience Intake

## Capture A Reusable Lesson

Record only information that changes future decisions:

1. Trigger: the user wording, symptom, or technical condition that should activate the lesson.
2. Root cause: the specific mistaken assumption or missing boundary.
3. Guardrail: a short rule that prevents recurrence.
4. Verification: the observable check that proves the rule was applied.

## Maintain The Library

- Search the existing `lanyan-*` skill first and update the one that owns the workflow.
- Prefer one concise rule over a new skill when the trigger already belongs to an existing domain.
- Create a new skill only for a distinct recurring domain with a different trigger and workflow.
- Keep triggering examples in frontmatter descriptions; keep procedural checks in the body.
- Move raw notes, old versions, and long evidence into `_archive`; never leave them as active skills.
- Keep third-party/public skills unchanged unless the user explicitly requests their modification.

## Quality Gate

Before adding a lesson, ask: does it overlap with a current skill, can it be stated in one actionable sentence, and does it include a verifiable outcome? Merge or discard it when the answer is no.
