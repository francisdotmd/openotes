---
title: Self-Healing Technical Documentation
description: Project note on building self-healing technical documentation with AI.
tags:
  - ai-engineering
  - documentation
  - self-healing
---

# Self-Healing Technical Documentation

A project exploring how to make technical documentation **self-healing** — docs that detect staleness, drift, and inaccuracies on their own and propose or apply corrections without manual upkeep.

## Problem

Technical docs rot. Code changes, APIs evolve, and the written guides fall behind. The cost of keeping docs accurate is high and usually deferred, so users hit outdated instructions.

**How it shows up in real life:**

- **Renamed API field, stale doc.** A backend renames `user_id` → `userId` in a response. The OpenAPI spec updates via codegen, but the hand-written "Getting Started" guide still shows `user_id`. A new integrator copies the example, gets `undefined`, files a support ticket.
- **Removed flag, lingering instructions.** A feature flag `enableBetaSearch` is deleted. The runbook still says "toggle `enableBetaSearch` to enable search." An on-call engineer follows it during an incident and wastes 20 minutes before realizing the flag is gone.
- **Changed default, wrong expectation.** A library bumps a default timeout from 30s to 10s. The docs say "default 30s." Users build retry logic around the wrong assumption; subtle latency bugs appear in production.
- **Moved file, broken example.** A refactor moves `src/utils/auth.ts` → `src/auth/index.ts`. A tutorial's import snippet `from '../utils/auth'` now fails. The doc example is never run in CI, so nobody notices until a reader complains.
- **Stale onboarding.** A new hire follows the "local setup" guide; step 3 installs a deprecated tool. They spend a day debugging before a teammate says "oh, we switched to X months ago."

## Goal

Build a system where documentation:

- **Monitors the codebase / source of truth it describes.** Continuously (or on-trigger) ingests code, schemas, types, tests, and changelogs so the system always has a current picture of reality to compare against.
- **Detects drift between docs and reality.** Compares each doc's claims against the source of truth and flags mismatches, stale sections, and broken examples — using machine-readable signals (types/schemas/tests) first.
- **Generates or proposes updated content automatically.** Uses an LLM, constrained to the changed facts, to draft corrections or open update PRs rather than just reporting staleness.
- **Keeps a human in the loop for review before publishing.** Surfaces drafts via PRs/inline suggestions for approval; reserves auto-merge only for mechanical, high-confidence, reversible edits.

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

## Open Questions

- Which signals best indicate drift (tests, types, changelogs)?
  - **Recommendation:** ranked by reliability — (1) types/schemas (strongest, machine-readable contracts, cheap to compare); (2) tests (encode intended behavior, change → docs likely should too); (3) changelogs/release notes (coarse triage trigger); (4) code diffs on touched paths (heuristic, broad but noisy); (5) build/CI failures from runnable doc examples (direct breakage). Practical combo: types as hard signal, tests as behavior signal, changelogs as triage trigger. Start with types+tests on one doc set.
- How to handle docs with no clear code counterpart?
  - **Recommendation:** don't force a code link — classify by `doc_type` and apply different rules: conceptual/architecture docs map to decision records/owners (drift signal = design decision changed, not code); runbooks map to process (drift signal = commands still run, via CI smoke tests); onboarding/narrative docs use a staleness clock + periodic human review (`last_verified` in frontmatter); tutorials/examples become runnable (doctests) so they break loudly on drift. Only enforce code-mapping + drift detection on `api-reference` and runnable tutorials.
- What's the review UX (PRs, inline suggestions, or auto-merge with audit)?
  - **Recommendation:** staged — start with PRs (builds trust, familiar, auditable), add inline suggestions for low-friction small edits, reserve auto-merge-with-audit for mechanical, high-confidence, tiny diffs (renamed params, version bumps). Autonomy scales with reversibility + mechanicalness, not LLM quality.
- What is the unit of drift (whole doc, section, or single claim)?
  - **Recommendation:** section-level. A whole doc is too coarse (one stale line blocks the rest); a single claim is too fine (mapping cost explodes and reviews become noise). Section-level matches how docs are authored and reviewed, and keeps the mapping layer tractable.
- Who owns the mapping layer (manual annotation or auto-inference)?
- How do we measure "healing" quality (false positives, false negatives, reviewer accept rate)?
- What is the rollback story for a wrong auto-drafted edit?
- Which doc types do we skip in v1?
- Who pays for LLM calls (CI or async) and what is the latency budget?
- Is adoption opt-in per repo or an org-wide default?

- What's the unit of drift — whole doc, section, or single claim?
- Who owns the mapping layer (doc ↔ source) — manual annotation or auto-inference?
- How do we measure "healing" quality (false positives/negatives, reviewer accept rate)?
- What's the rollback story for a wrong auto-drafted edit?
- Scope guardrails — which doc types do we NOT touch in v1?
- Cost/latency budget — who pays for LLM calls, CI or async?
- Adoption path — opt-in per repo or org-wide default?
