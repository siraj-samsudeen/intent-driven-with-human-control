# JavaScript Stack - cucumber-js

For `stack: javascript` in `openspec/config.yaml`. Read the skill's `SKILL.md` first for the spec format and runner invariants, plus [../EXTRACTION.md](../EXTRACTION.md) and [../COMPOSITION.md](../COMPOSITION.md) — the two normative contracts this pack implements.

## Files To Copy

Copy into `acceptance-tests/` at the repo root. Destination filenames are load-bearing.

| Source | Destination | Why the name is fixed |
| --- | --- | --- |
| `javascript/extract-gherkin.cjs` | `extract-gherkin.cjs` | `cucumber.cjs` requires it as `./extract-gherkin.cjs` |
| `javascript/cucumber.cjs` | `cucumber.cjs` | cucumber-js discovers its configuration by this exact name |
| `javascript/openspec-effective-paths.cjs` | `openspec-effective-paths.cjs` | required by `cucumber.cjs` under this name |

Then copy these starting points into `acceptance-tests/support/` and fill in the lines marked `PROJECT:`. They are not verbatim copies; they are the patterns every project needs:

| Source | Destination | What it gives |
| --- | --- | --- |
| `javascript/support/stack.js` | `support/stack.js` | Boots the app on its own database and ports (`ACCEPTANCE_API_PORT`, `ACCEPTANCE_WEB_PORT`), so it runs beside the dev server and other worktrees; refuses to start, naming the port, while another run holds them |
| `javascript/support/auth.js` | `support/auth.js` | People start signed in: a Playwright `storageState` from the same request the sign-in form makes, so the form is typed only in sign-in scenarios; one API session per run for setup |
| `javascript/support/world.js` | `support/world.js` | The World: one browser context per person, desktop at the answers' `ui_verifier_width`, `usePhone()`, a screenshot of every failed scenario, and the watching and debugging switches below |

`acceptance-tests/.gitignore`: `node_modules/`, `.extracted/`, `reports/`.

## Project Setup

`acceptance-tests/` is an independent Node project with its own `package.json`. Its hooks boot the application before the suite and shut it down after, so the suite runs with one command.

devDependencies: `@cucumber/cucumber` (^13.3), `glob` (^14), `cheerio`, and `playwright` pinned to the exact version the project already uses, so the browsers it downloads match. No spec linter (the skill's "No Separate Spec Lint").

```json
"scripts": {
  "test": "cucumber-js",
  "test:specs": "cucumber-js -p specs"
}
```

`npm test` stays a plain `cucumber-js` call: `cucumber.cjs` runs `extractAll()` synchronously at config load, so both profiles always see a freshly rebuilt `.extracted/`.

## Exclusion

cucumber-js filters at discovery time via line-targeted paths (`spec.feature:27:33` loads only scenarios starting at those lines), so superseded scenarios are never loaded and never appear as skipped. `effectivePaths()` computes these paths; nothing is written back to `.extracted/`.

Never use negated (`!`) globs to exclude the archive. cucumber-js does not support negation in `paths` and silently ignores such patterns.

## Verification

```sh
npx cucumber-js --dry-run
node -e "console.log(require('./cucumber.cjs').default.paths)"
```

Resolved paths must show only `.extracted/` entries, some line-targeted, and nothing under `changes/archive/`.

For a name-level parity diff against another stack, read `testCase` envelopes, not `pickle` envelopes.

```sh
npx cucumber-js --dry-run -f message | \
  node -e "let p={},s=[];require('readline').createInterface({input:process.stdin}).on('line',l=>{try{const m=JSON.parse(l);if(m.pickle)p[m.pickle.id]=m.pickle.name;if(m.testCase)s.push(m.testCase.pickleId)}catch{}}).on('close',()=>console.log(s.map(i=>p[i]).sort().join('\\n')))"
```

## Watching And Debugging

For a person at their own screen; agents and CI set none of these, and browser checks run headless.

| Setting | What it does |
| --- | --- |
| `HEADED=1` | A visible browser, slowed down so each action can be followed. |
| `PWDEBUG=1` | Playwright's Inspector (headed), pausing before each action; no step timeout. |
| `VIDEO=1` | A slowed-down, headless video of every scenario, one per person's browser: `reports/videos/<NN>-<scenario>[-<who>].webm`, numbered in run order. Only a `VIDEO=1` run replaces the videos, so a later plain run leaves the ones to review. A change's scenarios run in its proposal's capability order (COMPOSITION.md), so the videos play the way a person meets the feature. |
| `TRACE=1` | Traces every browser; a failed scenario keeps `reports/traces/<scenario>[-<who>].zip`. On in CI by default. Open one with `npx --prefix acceptance-tests playwright show-trace <zip>`. |

The owner watches the `VIDEO=1` videos before a change is archived (the `change-modes` skill).

## CI

Run the suite in the job that already has the app's database, after any other browser suite (it boots its own stack on its own ports), and upload the report and traces whatever the result:

```yaml
- name: Run acceptance suite
  run: |
    npm --prefix acceptance-tests ci
    npx --prefix acceptance-tests playwright install --with-deps chromium
    npm --prefix acceptance-tests test

- name: Upload acceptance report
  if: ${{ !cancelled() }}
  uses: actions/upload-artifact@v4
  with:
    name: acceptance-report
    path: acceptance-tests/reports/
    if-no-files-found: ignore
```

If CI picks jobs by changed paths, route `openspec/**/spec.md` and `acceptance-tests/**` (except its `.md` files) to this job: a spec's fenced Gherkin is the suite's source, not prose.

## Page Object Model

Page objects live in `acceptance-tests/support/pages/`, one per screen or flow. Parse responses with `cheerio`; never with regexes over raw HTML. The World stays a thin HTTP client holding page-object instances.

```js
When('they submit a valid email and password', async function () {
  this.result = await this.signupPage.submitSignup({
    email: 'user@example.com',
    password: 'correct-horse-battery-staple',
  });
});

Then('an error message is shown', function () {
  assert.ok(this.signupPage.errorMessage());
});
```
