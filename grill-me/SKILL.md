---
name: "grill-me"
description: "Use when the user wants to stress-test a plan, get grilled on a design, challenge an architecture, poke holes in an idea, or asks what is missing before implementation."
---

# Grill Me

Stress-test the user's plan or design through structured, one-at-a-time questioning. The goal is not to interrogate, but to reach shared understanding by walking through each branch of the decision tree, resolving dependencies, and surfacing blind spots.

## Questioning Strategy

Start by understanding the full scope of what the user is proposing. Then drill into each branch systematically. Do not jump between unrelated topics.

For each turn:

1. Ask one focused question about a specific aspect of the plan.
2. Explain briefly why this question matters.
3. Offer a recommended answer based on the codebase and current context.
4. Wait for the user's response before moving on.

One question at a time forces depth. A wall of questions leads to shallow answers.

## What To Probe

Focus on the areas that usually hide risk:

- **Assumptions**: what is being taken for granted that may not hold?
- **Edge cases**: what happens on unhappy paths, bad input, or unusual states?
- **Dependencies**: does one decision constrain another?
- **Trade-offs**: what is being sacrificed for the chosen approach?
- **Scope boundaries**: what is in scope, out of scope, and why?
- **Rollback and failure modes**: if this goes wrong, what is the recovery path?
- **Alternatives considered**: what other approaches were rejected?

You do not need to cover every category. Read the plan and focus on where the real risk lives.

## Codebase-Aware Questioning

If a question can be answered by exploring the codebase, explore the codebase first and present the finding instead of asking the user. Say what you checked so the user knows where the recommendation comes from.

Use the user's time for design intent, business constraints, and team context, not for facts that can be verified locally.

## Progress Tracking

After every 3 to 4 resolved questions, print a short status block:

```text
Resolved: [list of settled decisions]
Open: [list of remaining branches to explore]
```

This keeps the session focused and shows progress.

## Handling "I Don't Know"

When the user does not have an answer:

- Offer 2 to 3 concrete options with trade-offs.
- If the decision is not blocking, mark it as open and continue.
- Return to open items at the end.

The point is to identify gaps. Finding a gap is progress.

## Wrapping Up

When the major branches are resolved, or the user wants to stop, summarize:

1. **Decisions made**: the concrete commitments that were settled.
2. **Open items**: unresolved questions and recommended next steps.
3. **Key risks**: the top risks to watch during implementation.

Keep the summary tight so it can be used as a working reference.
