# Core Principles

How to work, before any code is written.

## Working style

- **Be concise** in all interactions and commit messages.
- **Plan before coding.** State the approach and confirm it before writing code. Turn vague instructions into verifiable targets first: "add validation" becomes "write tests for invalid inputs, then make them pass."
- **Say "I don't know"** instead of guessing. If the request is ambiguous, ask. If you're confused, name what is unclear — do not pick one interpretation and run.
- **Push back** when a simpler approach exists. State assumptions out loud.

## Code philosophy

- **Simplicity first.** Write the minimum code that solves the problem. No speculative abstractions. No flexibility nobody asked for. The test: would a senior engineer call this overcomplicated?
- **Surgical changes.** Touch only what the task requires. Do not improve neighboring code. Do not refactor what is not broken. Every changed line should trace back to the request.
- **Return early instead of building conditional mazes.** Validate and bail at the top; keep the happy path unindented. Avoid `else` when a guard clause works.
- **Name the business meaning, not the technical accident.** `isEligibleForRenewal`, not `checkFlag2`.
- **Separate decisions from actions.** Pure functions decide; thin imperative shells act. Decision logic should be testable without mocking I/O.
- **Make errors useful to the next person.** An error message should say what failed, where, and with what input — see [errors.md](errors.md).
- **Single source of truth.** Do not introduce duplicated state, configuration, or mappings that require updating multiple files for the same change. Prefer deriving values dynamically from existing sources over maintaining manual synchronization points.
