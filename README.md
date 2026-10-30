# Intent-Driven With Human Control

A fork of [intent-driven-dev/intent-driven-template](https://github.com/intent-driven-dev/intent-driven-template)
for Claude Code, with one change of philosophy: **the human approves every step**, and the git
history records each step well enough that a reviewer — or a learner — can follow exactly how
the code was built.

The upstream template brings the human in at the beginning (intent, specs, design) and at the
end (review); agents do the implementation in between. Here the planning is unchanged, but the
build runs one test at a time, in the manner of Kent Beck's augmented coding.

Each change picks its mode when it is created: **build** (the default, as above), **learning**
(build plus a stop before every command) or **autopilot** (the upstream flow, for work the human
does not need to follow step by step).

## Sources

1. **Intent-driven template** — everything upstream: the OpenSpec `proposal → specs → design →
   adr → tasks` workflow, Gherkin-style scenarios, and its skills and agents. Kept as is.
2. **Kent Beck, [Augmented Coding: Beyond the Vibes](https://tidyfirst.substack.com/p/augmented-coding-beyond-the-vibes)**
   (25-Jun-2025) — the human says "go" and the agent takes the next test from the plan; Tidy
   First (structural changes in their own commits); and his warning signs: loops, functionality
   nobody asked for, tests disabled or deleted.

## What we changed, and why

| Change | Why |
|---|---|
| **The human controls every step that has something to decide**, planning and build alike (build and learning modes). Planning: the agent drafts each artifact, the human reviews and edits, one commit per artifact. | The person working with the agent must understand and own every step, not just the result. |
| **The agent stops only when the stop has content** — a draft, a real decision, a prediction, a named smell. Never "say go", "shall I commit?", "shall I write the code?". Formatting and tooling choices have defaults and are never asked. | A stop that only asks permission gets approved by reflex and wears down attention for the stops that matter. |
| **A change reaches `main` in two PRs — the upstream `openspec-git-discipline`.** The proposal (planning artifacts) merges first; the implementation follows in a second PR; archive runs on `main` after it merges. Replaces our earlier one-PR rule. The agent still commits each approved step and opens each PR without asking; merging is the human's. | Proposals in progress are visible on `main`, so others can see and comment on what is planned before it is built. |
| **`tasks.md` is the test list** — one group per scenario, the outer acceptance test first, then the inner tests; no setup tasks. | It is Beck's `plan.md`. Setup that arrives with the test that needs it can be traced to that test; setup planned up front cannot. |
| **One stop per test** — the human's prediction — with **red, green and refactor as separate commits**. | Each step can be checked out and replayed. A combined commit cannot show that the test failed first, or how. |
| **We commit the red** — unlike Beck, who commits only when all tests pass. | The failing test, the prediction and the real failure are the record we want. |
| **The commit message carries what the code cannot show** — the command run, the prediction, the actual output, what was deliberately not built. | Diffs show what changed, never why or how it was decided. A separate diary drifts and costs more than a commit message. |
| **The human predicts every red; the agent predicts only when the human says "don't know"** (with a reason). A match is one line; a mismatch is recorded with why. | The human's prediction is the moment they model the system instead of watching it. The agent's prediction is almost always right and almost never informative — recorded every time, it buries the lines that matter. |
| **Prediction accuracy sets the step size.** | Predictions that always match mean the steps are too small; frequent misses mean slow down. |
| **Every change is mandated by a test** — code, dependencies, files. Config no test can drive states `Breaks without it:`. | Nothing arrives that nobody needed. |
| **`design.md` is a hypothesis.** When the build leads elsewhere, a `design:` commit records what changed and why. | That the plan changed, and why, is one of the most useful pages in the history. |
| **Reconcile at the end of each change**: coverage, unused code (knip), types, mutation testing (Stryker). Every finding is ratified into the spec, deleted, or put to the human. | Agents build more than they are asked. Line coverage cannot see an untested one-line branch; mutation testing can. |
| **Every red run is a commit**, `red: <behaviour> (run N)`, with `Ran:`, `Predict:`, `Actual:` and `Mandates:` — including runs that fail for the wrong reason. | The chain of failures shows why each package, file and stub exists; only the last run is the "right" red. |
| **`grill-me` on every planning artifact** — one decision per question, showing the text it will write into the artifact, with the alternatives. | The human settles each decision while reading the actual wording; a bulk draft hides the decisions that matter. |
| **Three modes, chosen per change when it is created** (not per repo). **Autopilot** is the upstream flow: the human reviews the planning artifacts; the build runs through the `senior-dev` and `senior-qa` subagents with acceptance tests first (`spec-as-source`), parallel worktrees allowed — `openspec new change <name> --schema intent-driven`. **Build** (default) stops only for a draft, a decision, a prediction or a smell. **Learning** also stops before every command — even read-only ones — to show it, explain it, wait for "run", then interpret the result; related commands may share one "run". The proposal's first line, `**Mode:**`, records the choice. See `CLAUDE.md` § Modes. | Not every change needs the human at every step: routine work can run on autopilot, a maintainer wants few stops, a learner learns from every command. Before, acceptance-first never ran on Claude Code, because `spec-as-source` and the subagent routing were commented out; autopilot turns them on for its changes only. |
| **Adversarial authoring on Claude Code uses the Codex CLI as the reviewer** when it is installed, otherwise a different Claude model than the author — and runs only when the human agrees, proposed when the agent judges the work complex or critical (security, permissions, data). | A Claude Code subagent can only run Claude models, so the upstream cross-model council would collapse to one family. Running it on every artifact costs more than it finds. |
| **Review guidelines in `AGENTS.md`**: Codex reviews locally if available, otherwise another Claude model; at most 3 rounds, then the human decides; every finding triaged — fix now (breaks core functionality), fix if cheap or one issue (plausible, limited impact), or one line under "Deliberately not handled" (rare or theoretical). | The product evolves fast: fitting the need matters more than perfect code. Without a cap and a triage, reviews keep finding theoretical edge cases and each one turns into code or an issue. |
| **Formats are enforced by review, not a git hook** — for now. | We will learn which fields hold up after 100–200 commits, then decide what to enforce. |

The process itself lives in [`.agents/skills/outside-in-tdd/SKILL.md`](.agents/skills/outside-in-tdd/SKILL.md);
the build-phase changes to the schema in
[`openspec/schemas/intent-driven-with-human-control/`](openspec/schemas/intent-driven-with-human-control/).
Claude Code wiring: `CLAUDE.md`, `.claude/skills/` (links to the upstream skills, plus a Claude
Code version of `adversarial-authoring`) and `.claude/agents/`.

---

*The upstream README follows.*

# Intent-Driven Template

An [OpenSpec](https://github.com/Fission-AI/OpenSpec) and [OpenCode](https://opencode.ai/) template with the `proposal -> specs -> design -> adr -> tasks` workflow enabled, plus optional collaboration workflows for teams that choose to adopt them.

## Walkthrough

[![Spec-Driven Development with OpenSpec and OpenCode](https://img.youtube.com/vi/o1w3V3DElV4/maxresdefault.jpg)](https://www.youtube.com/watch?v=o1w3V3DElV4)

Read the full walkthrough:

[Spec-Driven Development with OpenSpec and OpenCode](https://intent-driven.dev/blog/2026/05/10/spec-driven-development-openspec-opencode/)

[SDD with Multi-Model Spec Review and Glossary](https://intent-driven.dev/blog/2026/06/27/sdd-adversarial-authoring-glossary/)

## How To Use This Template

### Start A New Project From This Template

Clone this repository, open it with OpenCode, and start working from the bundled
OpenSpec configuration, commands, skills, and schema.

### Add This Template To An Existing Project

Open your existing project with OpenCode and ask it to install the template:

```text
Read and understand https://raw.githubusercontent.com/intent-driven-dev/intent-driven-template/refs/heads/main/INSTALL_TEMPLATE.md and follow the instructions there.
```

## What This Template Uses

- OpenSpec for setup, proposal, specification, design, ADR, and task artifacts.
- Custom schemas from https://github.com/intent-driven-dev/openspec-schemas.
- A bundled local copy of the `intent-driven` custom schema from
  https://github.com/intent-driven-dev/openspec-schemas/tree/main/openspec/schemas/intent-driven
  for the full `proposal -> specs -> design -> adr -> tasks` lifecycle.
- OpenSpec git discipline so proposals land on `main` before apply, and
  implementation lands on `main` before archive.
- OpenCode skills for repeatable collaboration and implementation workflows,
  including C4 diagrams, ADR authoring, and OpenSpec lifecycle commands.
- A bundled `test-driven-development` skill plus `senior-dev` and `senior-qa`
  subagents in `.opencode/agent/` for test-first implementation and
  acceptance-test work, routed via the `context` block in `openspec/config.yaml`.
- A `grill-me` style of rigorous design interrogation, inspired by
  https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md.
- ADRs for durable architectural decisions.
- C4 diagrams for communicating architecture boundaries and relationships.
- Gherkin-style requirements and scenarios for observable behaviour.

The bundled OpenSpec schema is a local copy of the `intent-driven` schema from
https://github.com/intent-driven-dev/openspec-schemas/tree/main/openspec/schemas/intent-driven.

## Optional: Enable Superpowers

[Superpowers](https://github.com/obra/superpowers) provides guided practices such
as brainstorming, planning, debugging, verification, worktrees, and
subagent-driven parallel work. It is not enabled by default in this template.

To enable it, add the plugin entry to `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

If your configuration already has a `plugin` array, append the entry while
preserving your existing plugins and other settings. Restart OpenCode to load it.

## Workflow

The intent-driven workflow moves through these artifacts in order:

```text
proposal -> specs -> design -> adr -> tasks
```

- `proposal` captures why the change matters.
- `specs` describe observable behaviour with Gherkin-style scenarios.
- `design` explains the implementation approach and trade-offs.
- `adr` records durable architectural decisions.
- `tasks` turn the accepted intent, behaviour, design, and decisions into work.

## Schema

`openspec/config.yaml` already selects the bundled [`intent-driven` schema](openspec/schemas/intent-driven/schema.yaml). Learn how to tailor artifact workflows in [OpenSpec Custom Schemas](https://intent-driven.dev/blog/2026/02/12/openspec-custom-schemas/).

## Skills

Standard OpenSpec lifecycle skills in `.opencode/skills/` — names are self-explanatory:
`openspec-new-change`, `openspec-propose`, `openspec-continue-change`, `openspec-explore`,
`openspec-apply-change`, `openspec-verify-change`, `openspec-sync-specs`, `openspec-archive-change`

| Skill | Location | Purpose | Enabled Or Updated By |
|-------|----------|---------|-----------------------|
| `openspec-bulk-apply-change` | `.opencode/skills/` | Applies multiple active changes concurrently in isolated worktrees with parallel verification. | Invoke directly with `/opsx:bulk-apply` command when applying multiple active openspec changes. |
| `adversarial-authoring` | `.opencode/skills/` | Runs author and reviewer agents in sequence to reduce model bias in drafts. | Refer to the skill under rule in the rules section of `openspec/config.yaml`. |
| `grill-me` | `.agents/skills/` | Interrogates plans and designs with probing questions to surface hidden assumptions. | Refer to this skill under rules section in `openspec/config.yaml`. |
| `c4-diagrams` | `.agents/skills/` | Visualises system architecture using C4 model levels in ASCII or Mermaid. | Refer to this skill under `design` rule in `openspec/config.yaml`. |
| `architectural-decision-records` | `.agents/skills/` | Captures architectural decisions with rationale, tradeoffs, and supersession chains. | Automatically invoked or can be explicilty referred to under `adr` rule in `openspec/config.yaml`. |
| `gherkin-authoring` | `.agents/skills/` | Drafts and improves Gherkin scenarios for observable, domain-language behaviour. | Required by `spec-as-source` during the `specs` phase. |
| `glossary` | `.agents/skills/` | Maintains business and technical terminology and companion glossary references for specification artifacts. | Can be referred to under `proposal` and/or `design` rule in `openspec/config.yaml`. |
| `openspec-git-discipline` | `.agents/skills/` | Enforces that proposals reach `main` before apply, and implementation merges before archive. | Enabled through `AGENTS.md`. |
| `spec-as-source` | `.agents/skills/` | Adds executable acceptance specifications to the intent-driven workflow. | Explicitly opt in by uncommenting both the `specs` and `tasks` rules in `openspec/config.yaml`. |
| `acceptance-test-authoring` | `.agents/skills/` | Configures the acceptance runner, extraction, linting, and step definitions for `spec-as-source`. | Required by `spec-as-source` when configuring or changing acceptance-test infrastructure. |
| `test-driven-development` | `.agents/skills/` | Guides strict red-green-refactor TDD: one behaviour per test, minimal implementation to pass, refactoring on green, with best-practice patterns and collaborator mocking guidance. | Loaded by the `senior-dev` agent in `.opencode/agent/senior-dev.md`; enable by uncommenting the `context` lines in `openspec/config.yaml` that route src work to `senior-dev`. |

Example configurations are available in `openspec/config.yaml` and `AGENTS.md`. You can comment/uncomment the skill references to enable them.

## Experimental: Spec As Source

The `spec-as-source` skill is an opt-in layer on the intent-driven schema that uses fenced Gherkin in OpenSpec specifications as the source for generating acceptance tests. To enable `spec-as-source` capability add the skill reference under both the `specs` and `tasks` rules in `openspec/config.yaml`.

During `specs`, `spec-as-source` requires `gherkin-authoring` to author the fenced Gherkin scenarios. During `tasks`, it replaces the standard task template with acceptance-test-first work: configure and run the acceptance suite, then implement application work. Use `acceptance-test-authoring` to configure the runner, extraction, linting, and step definitions that execute the specifications. See [Behavior-Driven Development and Spec-Driven Development with OpenSpec](https://intent-driven.dev/blog/2026/07/17/behavior-driven-development-sdd-openspec/).

## Further Reading

- Template overview: [Spec-Driven Development with OpenSpec and OpenCode](https://intent-driven.dev/blog/2026/05/10/spec-driven-development-openspec-opencode/)
- Schema customization: [OpenSpec Custom Schemas](https://intent-driven.dev/blog/2026/02/12/openspec-custom-schemas/)
- Durable architecture: [Architectural Decision Records with Spec-Driven Development using OpenSpec](https://intent-driven.dev/blog/2026/04/29/spec-driven-development-with-adr/)
- Multi-model review and glossary: [SDD with Multi-Model Spec Review and Glossary](https://intent-driven.dev/blog/2026/06/27/sdd-adversarial-authoring-glossary/)
- Parallel implementation: [OpenSpec, Git WorkTrees and OpenCode](https://intent-driven.dev/blog/2026/04/01/openspec-git-worktrees-opencode/)
- Spec-As-Source Spec-Driven Development: [Behavior-Driven Development and Spec-Driven Development with OpenSpec](https://intent-driven.dev/blog/2026/07/17/behavior-driven-development-sdd-openspec/)
- Brownfield adoption: [Spec-Driven Development with Brownfield Projects](https://intent-driven.dev/blog/2026/03/10/spec-driven-development-brownfield/)
- SDD + BDD + TDD: [How TDD and BDD Actually Fit Into Spec-Driven Development](https://intent-driven.dev/blog/2026/08/23/tdd-bdd-spec-driven-development/)

## Agents

Specialist agents used within skills, in `.opencode/agent/`:

| Agent | Purpose |
|-------|---------|
| `adversarial-author` | Writes an initial draft of a specification artifact or design document. |
| `adversarial-reviewer` | Reviews the author's draft with challenges and improvement suggestions. |
| `senior-dev` | Implements src work test-first through strict red-green-refactor, following the `test-driven-development` skill. |
| `senior-qa` | Authors acceptance tests, step definitions, and runner configuration, following the `acceptance-test-authoring` skill. |
