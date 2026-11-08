## 1. First-time setup — skip this whole section if the binding is already set up

- [ ] 1.1 Confirm `acceptance_runner: binding` and `stack: javascript` in `openspec/config.yaml`. If absent, ask and add them — a specs-zone edit, committed on its own before any code
- [ ] 1.2 Copy the `spec-as-source` skill's `references/binding/` files into the project (see that folder's README) and set the test folders the check scans
- [ ] 1.3 Add `check:scenarios` (the check and its unit tests) to the project's scripts and its CI fast checks; gitignore `scenarios-report/`

## 2. Scenarios — one task per fenced scenario, in the order the proposal lists its capabilities; each = a bound test that fails for the right reason → implement → passes → commit

- [ ] 2.1 <!-- capability › scenario --> (server | component) — red → green → commit
- [ ] 2.2 <!-- capability › scenario --> (server | component) — red → green → commit

## 3. Tour — only when the proposal names a tour chapter

- [ ] 3.1 <!-- tour › chapter -->: add or change the chapter, through the screen, within its time limit (`product-tours`)

## 4. Completion

- [ ] 4.1 `check:scenarios` passes: every fenced scenario of the effective spec is bound, and none twice
- [ ] 4.2 The full suite and every tour pass; the scenario report shows each of this change's scenarios passed
