# Testing & Tooling

## Testing

- Vitest + Testing Library. Tests are **colocated** `*.test.ts` siblings of what they test.
- Test the **pure logic** (parsers, mappers, schema generators, calculators) — the "decisions" half of decisions-vs-actions. Don't unit-test thin component wiring.
- BDD style: `describe('<subject>')` + `it('should ...')`.

## Tooling

- Formatter/linter enforced pre-commit: single quotes, semicolons, 2-space indent, ~120 line width, `noExplicitAny` and unused imports/vars as errors, organized imports.
