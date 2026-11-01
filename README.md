# Intent-Driven With Human Control

The shared process template for all of Siraj's projects. It starts from
[intent-driven-dev/intent-driven-template](https://github.com/intent-driven-dev/intent-driven-template)
and changes it for Claude Code, with one change of philosophy: **the human approves every step
that has something to decide**, and the git history records each step well enough that a
reviewer — or a learner — can follow exactly how the code was built.

The upstream template brings the human in at the beginning (intent, specs, design) and at the
end (review); agents do the implementation in between. Here the planning is the same, and the
build can run one test at a time, in the manner of Kent Beck's augmented coding. The session
picks the mode, and can switch it at any time: **build** (the default), **learning** (build plus
a stop before every command) or **autopilot** (the upstream flow).

## Install and update — Copier

Projects are linked to this template through [Copier](https://copier.readthedocs.io/), so a
process improvement made here reaches every project with one command.

```bash
uvx copier copy --trust gh:siraj-samsudeen/intent-driven-with-human-control <project-folder>
```

```bash
uvx copier update --trust
```

`copier copy` asks the questions below and writes the answers to `.copier-answers.yml`.
`copier update` brings the project to the template's newest tag with a 3-way merge: the
project's own edits are kept, and a clash shows as a conflict to resolve. Copier takes the newest
tag, so a change here reaches projects once it is tagged (`git tag -a vX.Y.Z`, then push the tag).
`--trust` is needed because `copier.yml` sets template options; nothing runs on install.

| Answer | Default | Read by |
|---|---|---|
| `repo` — GitHub `owner/name` | (asked) | `issue-workflow`, `scripts/create-labels.sh` |
| `extra_labels` — labels beyond red/yellow/green/chore | `[]` | `scripts/create-labels.sh` |
| `acceptance_stack` — `javascript` or `python` | `javascript` | `stack:` in `openspec/config.yaml` |
| `desktop_first` | `true` | `issue-workflow` § UI route |
| `ui_verifier_width` | `1280` | `issue-workflow` § UI route |
| `screenshots_in_prs` | `false` | `issue-workflow` § UI route |
| `adr_dir` | `adr` | `openspec/config.yaml`, `change-modes` |

Change an answer with `uvx copier update --trust --defaults --data <answer>=<value>`.

What a project gets: `.agents/`, `.claude/`, `.opencode/`, `openspec/`, `AGENTS.md`,
`CLAUDE.md`, `scripts/create-labels.sh`, `skills-lock.json`, plus `.gitignore` and
`opencode.json` when it has none. This README, the licence and `INSTALL_TEMPLATE.md` (upstream's
OpenCode install guide) stay here. A project adds its own rules under `## Project` at the end of `AGENTS.md`
and in `context:` in `openspec/config.yaml`; a change to the shared process is made here, then
pulled.

## Where the process lives

| What | Where |
|---|---|
| Modes, defaults never asked, the git flow (one worktree, branch and PR per change) | [`change-modes`](.agents/skills/change-modes/SKILL.md) |
| The build loop in build and learning modes | [`outside-in-tdd`](.agents/skills/outside-in-tdd/SKILL.md) |
| Issues, ratings, blocked by, manager and task sessions, worktrees, the UI route | [`issue-workflow`](.agents/skills/issue-workflow/SKILL.md) |
| Per-artifact rules: proposal, specs, design, adr, tasks | [`openspec/config.yaml.jinja`](openspec/config.yaml.jinja) |
| Reviews and finding triage | [`AGENTS.md`](AGENTS.md) § Review guidelines |
| The schema | [`openspec/schemas/intent-driven/`](openspec/schemas/intent-driven/), upstream's, unchanged |

## Sources

1. **Intent-driven template** — everything upstream: the OpenSpec `proposal → specs → design →
   adr → tasks` workflow, its schema (from
   [intent-driven-dev/openspec-schemas](https://github.com/intent-driven-dev/openspec-schemas)),
   and its skills and agents. Kept as is, so upstream updates merge cleanly.
2. **Kent Beck, [Augmented Coding: Beyond the Vibes](https://tidyfirst.substack.com/p/augmented-coding-beyond-the-vibes)**
   (25-Jun-2025) — the human says "go" and the agent takes the next test from the plan; Tidy
   First (structural changes in their own commits); and his warning signs: loops, functionality
   nobody asked for, tests disabled or deleted.
3. **Featherbase** — the process the owner settled while building it: ratings, the issue body,
   manager and task sessions, the UI route, the spec and design rules.

## What we changed, and why

| Change | Why |
|---|---|
| **Upstream's `intent-driven` schema, unchanged.** No schema of our own and no patch to upstream's; our rules live in `openspec/config.yaml` and skills. | Upstream updates pull cleanly. An earlier fork of the schema had to be merged by hand. |
| **The mode belongs to the session, never to the change.** Autopilot, build and learning use the same planning artifacts and the same task list; switching changes no file. | The owner switches mode mid-change, depending on how much of a hurry he's in. Recording the mode in the change made each switch rewrite planning files. |
| **The human controls every step that has something to decide**, planning and build alike (build and learning modes). The agent drafts each artifact, the human reviews and edits, one commit per artifact. | The person working with the agent must understand and own every step, not just the result. |
| **The agent stops only when the stop has content** — a draft, a real decision, a prediction, a named smell. Never "say go", "shall I commit?". Formatting and tooling choices have defaults and are never asked. | A stop that only asks permission gets approved by reflex and wears down attention for the stops that matter. |
| **Fenced Gherkin in every mode** (`spec-as-source`); the acceptance runner's own parse is the spec check (no gherkin-lint: [intent-driven-template#9](https://github.com/intent-driven-dev/intent-driven-template/issues/9)). | Specs are written the same way whatever the mode, and every scenario is tied to the test that runs it. |
| **Spec rules**: requirements in the order a person meets them, the common case before edge cases, one example running through every scenario, scenario names that say what the user sees, an independent MECE review before approval. Proposals list capabilities in journey order, and the suite runs and records videos in that order. | Specs are read by product people. Review videos should play the way a person meets the feature. |
| **Acceptance tooling from featherbase** (`acceptance-test-authoring`, both stacks where it is the contract): a prose scenario (no gherkin fence) is left out and named, never an error; a change's scenarios run in its proposal's capability order; no spec linter. JavaScript starting points in `references/javascript/support/`: the app boots on its own ports and refuses a second run, people start signed in through the API, and `HEADED`, `PWDEBUG`, `VIDEO` (review videos with step captions, `VIDEO_SLOWMO`, and `npm run video:journey` for one `journey.mp4`; never recorded by default) and `TRACE` (CI) switches. | Specs written before spec-as-source still run their fenced scenarios. The owner reviews a change by watching its videos in journey order. |
| **`design.md` holds only real decisions** — a real alternative, and costly if wrong — each approved by the owner on its own. | A design padded with the spec restated hides the decisions that need judgement. |
| **ADRs follow upstream's rule**: top-level `adr/`, never edited once accepted, superseded by a new ADR. | Nothing in the schema is patched, and an ADR's history stays readable. |
| **One issue, one worktree, one branch, one PR per change**, from proposal to archive. The manager session stays on `main`; each issue's task session works in its own worktree with its own ports and database. Archive after the owner watches the review videos; then the owner merges with a merge commit. Replaces upstream's two-PR flow. | The owner sees a change as one unit, and the manager keeps working on `main` while workers build. |
| **Issues carry a rating** (`red`, `yellow`, `green`, `chore`; `scripts/create-labels.sh`) on their first line, with "blocked by" as GitHub's native dependency, and go one at a time per worker. | Urgency and what an issue waits on are visible at a glance. |
| **UI changes take the UI route**: mockups, a numbered checklist in the issue, then an independent verifier at the answers' width, desktop first. | The owner agrees what he sees, and the build is checked against it by someone who didn't build it. |
| **In build mode, upstream's task list is the test list.** Step-definition tasks are the outer tests; setup arrives with the first test that needs it; inner tests are added as `(added)`. | Beck's `plan.md`, without a task format of our own. |
| **One stop per test** — the human's prediction — with **red, green and refactor as separate commits**, and every red run committed with `Ran:`, `Predict:`, `Actual:` and `Mandates:`. | Each step can be checked out and replayed; the chain of failures shows why each package, file and stub exists. |
| **The human predicts every red; the agent predicts only when the human says "don't know"**. Prediction accuracy sets the step size. | The human's prediction is the moment they model the system instead of watching it. |
| **Every change is mandated by a test** — code, dependencies, files. Config no test can drive states `Breaks without it:`. | Nothing arrives that nobody needed. |
| **Reconcile at the end of each change**: coverage, unused code, types, mutation testing. Every finding is ratified into the spec, deleted, or put to the human. | Agents build more than they are asked. |
| **`grill-me` on every planning artifact** — one decision per question, showing the text it will write. | The human settles each decision while reading the actual wording. |
| **Adversarial authoring on Claude Code uses the Codex CLI as the reviewer** when installed, otherwise a different Claude model — only with the human's consent. | A Claude Code subagent can only run Claude models. Running it on every artifact costs more than it finds. |
| **Review guidelines in `AGENTS.md`**: at most 3 rounds; every finding triaged — fix now, fix if cheap or one issue, or one line under "Deliberately not handled". | Without a cap and a triage, reviews keep finding theoretical edge cases. |

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
