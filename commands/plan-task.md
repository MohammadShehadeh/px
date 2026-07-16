---
description: Plan before coding — restate the task as verifiable targets with done criteria, scope boundaries, and STOP conditions; confirm before writing code
argument-hint: <task description>
---

Plan the following task before writing any code: $ARGUMENTS

1. **Restate the task as verifiable targets.** Turn vague asks into concrete, checkable outcomes ("add validation" → "these inputs are rejected with these error keys, covered by tests").
2. **State assumptions out loud.** If the request is ambiguous, ask — do not pick an interpretation and run. Say "I don't know" instead of guessing.
3. **List the surgical change set.** Exactly which files change and why each traces back to the request. Include an explicit **out-of-scope** list — files/behaviors that look related but must not be touched.
4. **Propose the simplest approach** that solves the problem — no speculative abstractions. If a simpler approach than the one implied exists, push back and say so.
5. **Define machine-checkable done criteria.** Commands and expected results using **the repo's scripts** (e.g. `npm run typecheck` / `pnpm typecheck` exits 0, test command passes including N new tests, `grep` for the old pattern returns nothing) — never prose like "works correctly".
6. **Name the STOP conditions.** The key assumptions the plan rests on; if any turns out false during implementation, stop and report instead of improvising.
7. **Stop and wait for confirmation.** Do not write code until the approach is approved.

Be concise throughout.
