<!--
Copy this into a project root as CLAUDE.md.

Preferred setup: install the px-* skills (plugin install, or copy skills/* into
.claude/skills/) — they load the full rules on demand, keeping this file tiny.

Fallback (no skills): copy skills/px-conventions/references/ into the repo as
rules/ and uncomment the @-imports below. This injects ~800 lines into every
session — prefer the skills.
-->

# Code Conventions

House conventions ship as the `px-conventions` skill (with `px-debug`, `px-nextjs-page`, `px-feature-package` for their workflows). Follow them for all TypeScript/React/Next.js code in this repo.

Non-negotiables: plan before coding and confirm the approach; be concise; say "I don't know" instead of guessing; simplest code that works; surgical diffs only.

<!-- Fallback @-imports — only if the skills are not installed:
@rules/core-principles.md
@rules/naming.md
@rules/typescript.md
@rules/components.md
@rules/hooks-state.md
@rules/forms.md
@rules/nextjs.md
@rules/styling.md
@rules/ui-composition.md
@rules/icons.md
@rules/services.md
@rules/optimistic-ui.md
@rules/errors.md
@rules/structure.md
@rules/testing.md
-->
