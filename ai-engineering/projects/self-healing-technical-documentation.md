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
- How to handle docs with no clear code counterpart?
- What's the review UX — PRs, inline suggestions, or auto-merge with audit?

## Next Steps

- Watch the source video and capture concrete techniques.
- Prototype a small drift detector on one doc set.
- Define evaluation: how do we measure "healing" quality?
