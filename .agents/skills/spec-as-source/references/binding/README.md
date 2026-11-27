# Scenario binding — reference implementation (Vitest)

The `binding` runner of `spec-as-source`: each fenced scenario of the effective spec is proven
by a Vitest test that walks its Gherkin steps. Copy these files:

| File | To | What it does |
|---|---|---|
| `spec-index.cjs` | `tools/scenarios/` | Reads the effective spec: main specs with active change deltas applied, outlines included |
| `bindings.cjs` | `tools/scenarios/` | Finds `scenarioFor(test, '<capability>')` and `scenario('<name>', …)` in test files. Set `SCENARIO_TEST_DIRS` (comma-separated) or edit `TEST_DIRS` |
| `check.cjs`, `check.test.cjs` | `tools/scenarios/` | The CI check and its tests; writes `scenarios-report/index.html` |
| `scenario.ts` | next to the test suite's fixtures | `scenarioFor(test, capability)`; fix the `require` path to `spec-index.cjs` |

Scripts:

```json
"check:scenarios": "node --test tools/scenarios/check.test.cjs && node tools/scenarios/check.cjs",
"scenarios:report": "vitest run --reporter=default --reporter=json --outputFile.json=scenarios-report/results.json; node tools/scenarios/check.cjs --results scenarios-report/results.json"
```

A test:

```ts
const scenario = scenarioFor(test, 'naming-and-numbering')
describe('naming-and-numbering', () => {
  scenario('An imported row already holds the next number', async ({ admin }, step) => {
    await step('a table "Bin" whose rows are numbered 1, 2, 3', () => makeTable(admin, ...))
    await step(/added bin 1 and imported a bin with the id 2/, async () => { ... })
    await step('the user adds another bin', async () => { ... })
    await step('it is numbered 3', () => expect(next).toBe('3'))
  })
})
```

Vitest sets up the fixtures a test destructures by reading its source, so `scenario.ts` gives
the wrapper the body's source text. Keep the fixtures destructured in the body's first argument.
