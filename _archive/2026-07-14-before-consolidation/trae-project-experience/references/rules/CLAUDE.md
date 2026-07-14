# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Comment What Matters

**Comments must explain business intent, not restate obvious code.**

When writing or editing code:
- Keep straightforward CRUD code lightly commented or uncommented if the code is already self-explanatory.
- Add plain, explicit comments for non-CRUD logic such as business rules, state transitions, mapping/aggregation, fallback handling, scoring, matching, or compensation paths.
- Prefer comments that explain why the logic exists, what business problem it solves, and how to understand the branch.
- For multi-method business call chains, comment not only the entry method but also the called key methods where business meaning would otherwise be hidden behind helper names.
- For method-level comments, explain the responsibility of that method in the chain, the business input/output it is stabilizing, and why this layer exists.
- Avoid noisy comments that only paraphrase the code line-by-line.

Good default:
- If a future reader might ask "why is this branch needed?" or "what business rule is encoded here?", add a direct comment above it.
- If a reviewer has to keep jumping into downstream methods to guess "what does this method really do in the business flow?", add a short method comment there too.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
