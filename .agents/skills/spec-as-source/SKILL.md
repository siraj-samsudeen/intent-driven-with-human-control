---
name: spec-as-source
description: Use when a user or project explicitly adopts OpenSpec spec-as-source, fenced Gherkin specifications as executable acceptance tests, or acceptance-test-first OpenSpec tasks.
---

# Spec As Source

## Overview

Use OpenSpec `spec.md` files as the executable source of truth. When this skill is active, fenced-Gherkin authoring, acceptance-first task ordering, spec-first implementation, and BDD zone isolation are one workflow.

**REQUIRED SUB-SKILL:** Use `gherkin-authoring` when drafting or modifying fenced Gherkin.

**REQUIRED SUB-SKILL:** Use `acceptance-test-authoring` when configuring or modifying the acceptance runner, extractor, linting, step definitions, or page objects (the `bdd` runner).

**REQUIRED SUB-SKILL:** Use `product-tours` for anything that needs a browser (the `binding` runner).

## Activation

This workflow is opt-in. Apply it only when the user or project instructions explicitly require `spec-as-source`.

Once active, the BDD zone rules below are mandatory. Do not activate or disable zone isolation separately.

## Runners

`acceptance_runner:` in `openspec/config.yaml` says what executes the scenarios.

- **`binding`** (the default for `javascript`): each fenced scenario is proven by a test in the
  project's own test runner, at the fastest layer that can prove it: a server test, or a
  component test against the real server. No browser and no second runner. The browser belongs
  to tours only (`product-tours`).
- **`bdd`**: a BDD runner (cucumber-js or behave) in `acceptance-tests/`, driving the browser
  (`acceptance-test-authoring`).

### The binding contract

1. **The test's name is the scenario's name**, inside a group named after its capability. A
   name the effective spec doesn't have fails when the test file loads.
2. **The test walks the scenario's Gherkin steps in order.** Each step names the next line, by
   its exact text (without its keyword) or a pattern. A step out of order, changed or left out
   fails the test, in the spec's own words. A Scenario Outline runs once per Examples row,
   with that row's values filled into the step text.
3. **The effective spec is the source.** Main specs, with the active changes' ADDED, MODIFIED
   and REMOVED requirements applied, read from the `spec.md` files directly. No generated
   `.feature` files.
4. **A check in CI** fails when a fenced scenario of the effective spec has no test, or when a
   test names a scenario the spec doesn't have. It writes a report: every capability,
   requirement and scenario, its Gherkin, where it runs and whether it passed, prose scenarios
   included.
5. **One runner per capability while moving from `bdd`.** A capability with any bound test is
   the binding's: every one of its fenced scenarios needs a test, and the BDD runner leaves
   the capability out. When the last capability has moved, `acceptance-tests/` goes.

The reference implementation (Vitest, `javascript`) is in [references/binding/](references/binding/).
Copy it and set its test folders; for `python` there is none yet, so use `bdd`.

## Artifact References

- When this skill is active, its artifact references override the corresponding
  templates from the configured OpenSpec schema.
- Draft each `spec.md` from [references/spec.md](references/spec.md), not from
  `openspec/schemas/intent-driven/templates/spec.md`.
- Draft `tasks.md` from [references/tasks.md](references/tasks.md) (`bdd`) or
  [references/tasks-binding.md](references/tasks-binding.md) (`binding`), not from the
  schema task template, preserving acceptance-test-first ordering.

## BDD Zone Rules

1. Code is written only for an active change under `openspec/changes/` whose scenarios describe the behavior.
2. A unit of work edits either the specs zone (non-`tasks.md` files under `openspec/`) or the code zone (non-`tasks.md` files outside `openspec/`), never both while changes are uncommitted.
3. Before crossing zones, finish and commit or stash the current zone.
4. Never commit specs-zone files and code-zone files together.
5. Any file named `tasks.md` is zone-neutral: it may be edited alongside and committed with either zone. Task updates do not select or cross a zone and do not require a separate commit or stash.
6. Never revert or discard user-owned changes without explicit permission.

Use `git status --porcelain` before editing. Ignore files named `tasks.md` when selecting the zone. Of the remaining changed paths, those under `openspec/` select the specs zone; those outside it select the code zone. If only `tasks.md` files are changed, no zone is selected.

## Workflow

1. Draft and validate the proposal and fenced-Gherkin delta specs.
2. Commit or stash specs-zone work before touching code.
3. Establish the failing scenario before implementation: a pending step (`bdd`) or a failing bound test (`binding`).
4. Implement the smallest change that makes the scenario pass. During OpenSpec apply, keep the active change's `tasks.md` updated as work completes, marking each completed task immediately. These progress updates may accompany implementation edits and commits.
5. Keep the effective acceptance suite green: no pending or undefined steps (`bdd`), every fenced scenario bound and passing (`binding`). If the proposal names a tour chapter, update it last (`product-tours`).
6. Commit or stash code-zone work before returning to specifications for sync or archive.

## Red Flags

- Implementing behavior with no active spec change.
- Editing specs-zone and code-zone files in the same uncommitted unit of work (`tasks.md` is zone-neutral).
- Writing implementation tasks before acceptance-test tasks.
- Treating generated `.feature` files as the source instead of `spec.md`.
- Running source-of-truth specs without active deltas when validating a change.
- (`binding`) A browser test that isn't a tour, or a scenario proven in a browser that a server or component test could prove.
- (`binding`) A scenario's capability running on both runners.

Stop and restore the workflow boundary when any red flag occurs.
