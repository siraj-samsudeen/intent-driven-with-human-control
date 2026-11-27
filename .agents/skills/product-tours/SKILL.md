---
name: product-tours
description: Use when writing, changing or running a product tour - a demo-length browser test that tells one story through the screen, doubles as the product's demo video, and is the project's only browser test. Also use when a change's proposal names a tour chapter, when a demo is being prepared, or when the owner asks for a demo video.
---

# Product Tours

## What a tour is

A **tour** is one long browser test that tells one story, the way the owner demos the product.
It is the project's only browser test. When it passes, the path the owner demos works. The same
run, slowed down and captioned, is the demo video.

Narrow promises are proven by scenario tests at the fastest layer that can prove them (the
`spec-as-source` skill). A tour proves what those tests can't: that the features hand over to
each other, for a person who isn't an administrator, with realistic amounts of data, quickly
enough.

## The tours a project keeps

- **One tour of the platform itself**, from first contact to its deepest idea.
- **One tour per app** built on it. An app's tour shows the platform features that app relies
  on, inside the app's own story, rather than in a separate tour.
- Later tours for particular audiences or power users go deeper. They don't repeat the first
  tour.

A tour is written like the script of a video: a title, then chapters, each a few steps. Write the
script first and the test second, as a press release comes before the product.

## Rules

1. **Through the screen only.** The one setup is the people who sign in, created the way a fresh
   install would. Everything else is done as a person does it. A later chapter uses what an
   earlier chapter made, never data seeded behind the screen.
2. **A normal person leads.** The lead is a person with the role the story needs, not the
   administrator who installed the instance.
3. **Realistic data.** The data is the size and mess real users bring: inconsistent names, odd
   spellings, a few hundred rows. It is generated, not committed, and plainly made up. A size
   knob runs the tour at demo scale.
4. **A watchdog.** The tour fails on any uncaught page error, any server error (5xx) and any
   request that gets no answer, even when every assertion passes. A harness test proves the
   watchdog still fires.
5. **Time limits.** Each chapter has a limit of about three times its real time, so a slowdown
   fails the tour before it fails a demo. The limits stretch with the data size.
6. **Gaps, not gaps hidden.** A step the product can't do yet is a `gap` chapter naming its
   issue. It shows as a card in the video and a note in the report, and never fails the tour. A
   known defect is not a gap: it is pinned as a failing test with its issue number.
7. **Built and served as an instance is.** The tour builds the app and serves it as production
   does, on its own database, reset once per run. A flag points it at a running instance
   instead (a preview, before a demo).
8. **Every CI run.** Tours run on every push and pull request, whatever the change touches. A
   tour that takes more than a few minutes in CI is two tours.

## The video

With the video switch on (`VIDEO=1`), the same test runs slowly. It shows each chapter as a
title card and each step as a caption, and records one video per tour. Captions are the step
texts, so write them in the product's words, for the person watching.

## A change and its tour

When a change's proposal names a tour chapter it adds or changes, the change's last task
updates that chapter. At archive, the owner watches the tour's video. A change that touches no
tour says so in its proposal.

## Red flags

- A tour that seeds data through an API to save time: the hand-over between features is what it
  is for.
- A tour signed in as the administrator.
- A chapter with no time limit, or a limit far above its real time.
- Narrow checks piling up in a tour. They belong in scenario tests.
- A browser test that isn't a tour.
