---
title: Self-Healing Technical Documentation
description: Project note on building self-healing technical documentation with AI.
tags:
  - ai-engineering
  - documentation
  - self-healing
---

# Self-Healing Technical Documentation

## Overview

A project exploring how to make technical documentation **self-healing** — docs that detect staleness, drift, and inaccuracies on their own and propose or apply corrections without manual upkeep.

## Problem

Technical docs rot. Code changes, APIs evolve, and the written guides fall behind. The cost of keeping docs accurate is high and usually deferred, so users hit outdated instructions.

## Goal

Build a system where documentation:

- Monitors the codebase / source of truth it describes.
- Detects drift between docs and reality.
- Generates or proposes updated content automatically.
- Keeps a human in the loop for review before publishing.

## Approach (draft)

1. **Ingest** the relevant source of truth (code, schemas, APIs).
2. **Compare** current docs against ingested facts.
3. **Flag** mismatches and stale sections.
4. **Draft** corrections with an LLM.
5. **Review & merge** via a human-in-the-loop workflow.

## How It Works

Core loop — **detect → diagnose → draft → review → publish**:

1. **Ingest the source of truth.** Pull code, schemas, types, tests, changelogs into a structured form the system can compare against. This is the "ground truth" snapshot.
2. **Map docs to sources.** Each doc section is tagged with the code/symbol it describes (e.g., an endpoint doc links to its route handler + OpenAPI entry). Without this mapping, you can't know what "should" be true.
3. **Detect drift.** Compare the doc's claims against the current source truth: schema diff, test changes, referenced-file changes, failing runnable examples.
4. **Diagnose & scope.** Decide which parts of a doc are affected and how stale (minor wording vs. breaking change). Route to the right owner.
5. **Draft corrections with an LLM.** Given the diff + current source, generate updated prose/code examples, constrained to the changed facts.
6. **Human-in-the-loop review.** Open a PR or inline suggestion; a maintainer approves. Auto-merge only for low-risk, high-confidence edits.
7. **Publish & re-baseline.** Update doc, re-snapshot the source truth, reset the drift clock.

Key design choices: prefer machine-readable signals (types/schemas) for *detection*, use the LLM only for *drafting*; keep a doc↔source mapping layer (that's the real asset); start narrow — one doc set, one signal type — and measure false positives before scaling.

## Value for Software Engineers

- **Less doc debt, more shipping.** Docs rot silently; self-healing automates the part nobody volunteers for.
- **Onboarding stays accurate.** New devs trust the docs instead of reverse-engineering the code.
- **Review becomes verification.** PRs check doc drift alongside tests, so "docs updated?" stops being a manual checklist.
- **Incident reduction.** Outdated runbooks cause outages; auto-flagging stale steps catches them before use.
- **Knowledge scales.** As systems grow, manual doc maintenance breaks down; automation keeps pace.

## Drift Signals

Ranked by reliability:

1. **Types / schemas (strongest).** OpenAPI, GraphQL, DB schemas, TS types — machine-readable contracts. If the doc contradicts the schema, that's objective drift. Cheap to compare.
2. **Tests.** A doc claiming "endpoint returns X" can be checked against integration tests. Tests encode intended behavior; when they change, docs likely should too.
3. **Changelogs / release notes.** Coarse but directional — good for "something changed here, re-check these docs," weak on specifics.
4. **Code diffs on touched paths.** Heuristic: if a doc references a file/module that changed, flag it. Noisy but broad.
5. **Build / CI failures.** Doc examples actually run (doctests, code fences) failing = direct drift signal.

Practical combo: types as the hard signal, tests as behavior signal, changelogs as a triage trigger. Start with types+tests on a single doc set.

## Open Questions

- Which signals best indicate drift (tests, types, changelogs)? *(See Drift Signals above.)*
- How to handle docs with no clear code counterpart? **Recommendation:** don't force a code link — classify by `doc_type` and apply different rules: conceptual/architecture docs map to decision records/owners (drift signal = design decision changed, not code); runbooks map to process (drift signal = commands still run, via CI smoke tests); onboarding/narrative docs use a staleness clock + periodic human review (`last_verified` in frontmatter); tutorials/examples become runnable (doctests) so they break loudly on drift. Only enforce code-mapping + drift detection on `api-reference` and runnable tutorials.
- What's the review UX — PRs, inline suggestions, or auto-merge with audit? **Recommendation:** staged — start with PRs (builds trust, familiar, auditable), add inline suggestions for low-friction small edits, reserve auto-merge-with-audit for mechanical, high-confidence, tiny diffs (renamed params, version bumps). Autonomy scales with reversibility + mechanicalness, not LLM quality.

## Next Steps

- Watch the source video and capture concrete techniques.
- Prototype a small drift detector on one doc set.
- Define evaluation: how do we measure "healing" quality?
