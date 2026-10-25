---
title: Self-Healing Technical Documentation
description: Project note on building self-healing technical documentation with AI.
tags:
  - ai-engineering
  - documentation
  - self-healing
---

# Self-Healing Technical Documentation

A project exploring how to make technical documentation self-healing: docs that detect staleness, drift, and inaccuracies on their own and propose or apply corrections without manual upkeep.

## Problem

Technical docs rot. Code changes, APIs evolve, and the written guides fall behind. The cost of keeping docs accurate is high and usually deferred, so users hit outdated instructions.

How it shows up in real life:

- Renamed API field, stale doc. A backend renames `user_id` to `userId` in a response. The OpenAPI spec updates via codegen, but the hand-written "Getting Started" guide still shows `user_id`. A new integrator copies the example, gets `undefined`, files a support ticket.
- Removed flag, lingering instructions. A feature flag `enableBetaSearch` is deleted. The runbook still says "toggle `enableBetaSearch` to enable search." An on-call engineer follows it during an incident and wastes 20 minutes before realizing the flag is gone.
- Changed default, wrong expectation. A library bumps a default timeout from 30s to 10s. The docs say "default 30s." Users build retry logic around the wrong assumption; subtle latency bugs appear in production.
- Moved file, broken example. A refactor moves `src/utils/auth.ts` to `src/auth/index.ts`. A tutorial's import snippet `from '../utils/auth'` now fails. The doc example is never run in CI, so nobody notices until a reader complains.
- Stale onboarding. A new hire follows the "local setup" guide; step 3 installs a deprecated tool. They spend a day debugging before a teammate says "oh, we switched to X months ago."

## Goal

Build a system where documentation:

- Monitors the codebase and source of truth it describes. Continuously (or on-trigger) ingests code, schemas, types, tests, and changelogs so the system always has a current picture of reality to compare against.
- Detects drift between docs and reality. Compares each doc's claims against the source of truth and flags mismatches, stale sections, and broken examples, using machine-readable signals (types, schemas, tests) first.
- Generates or proposes updated content automatically. Uses an LLM, constrained to the changed facts, to draft corrections or open update PRs rather than just reporting staleness.
- Keeps a human in the loop for review before publishing. Surfaces drafts via PRs or inline suggestions for approval; reserves auto-merge only for mechanical, high-confidence, reversible edits.

## Approach (draft)

1. Ingest the relevant source of truth (code, schemas, APIs).
2. Compare current docs against ingested facts.
3. Flag mismatches and stale sections.
4. Draft corrections with an LLM.
5. Review and merge via a human-in-the-loop workflow.

## How It Works

Core loop: detect, diagnose, draft, review, publish.

1. Ingest the source of truth. Pull code, schemas, types, tests, changelogs into a structured form the system can compare against. This is the "ground truth" snapshot.
2. Map docs to sources. Each doc section is tagged with the code or symbol it describes (e.g., an endpoint doc links to its route handler plus OpenAPI entry). Without this mapping, you can't know what "should" be true.
3. Detect drift. Compare the doc's claims against the current source truth: schema diff, test changes, referenced-file changes, failing runnable examples.
4. Diagnose and scope. Decide which parts of a doc are affected and how stale (minor wording vs. breaking change). Route to the right owner.
5. Draft corrections with an LLM. Given the diff plus current source, generate updated prose and code examples, constrained to the changed facts.
6. Human-in-the-loop review. Open a PR or inline suggestion; a maintainer approves. Auto-merge only for low-risk, high-confidence edits.
7. Publish and re-baseline. Update doc, re-snapshot the source truth, reset the drift clock.

Key design choices: prefer machine-readable signals (types, schemas) for detection, use the LLM only for drafting; keep a doc-to-source mapping layer (that's the real asset); start narrow (one doc set, one signal type) and measure false positives before scaling.

## Value for Software Engineers

- Less doc debt, more shipping. Docs rot silently; self-healing automates the part nobody volunteers for.
- Onboarding stays accurate. New devs trust the docs instead of reverse-engineering the code.
- Review becomes verification. PRs check doc drift alongside tests, so "docs updated?" stops being a manual checklist.
- Incident reduction. Outdated runbooks cause outages; auto-flagging stale steps catches them before use.
- Knowledge scales. As systems grow, manual doc maintenance breaks down; automation keeps pace.

## Tech Stack

**Detection (machine-readable signals):**
- Schemas and types: OpenAPI, GraphQL, JSON Schema, TypeScript types, DB migrations (Prisma/Drizzle); diff via `openapi-diff`, `graphql-inspector`, or schema diff tools.
- Tests: integration and E2E runners (Jest, Vitest, Playwright) as drift signals; runnable doc examples via doctests (mdx code fences executed in CI).
- Changelogs and git: git diff plus changed-file heuristics to triage which docs to re-check.

**Mapping layer (doc to source):**
- A small metadata store (YAML frontmatter or sidecar JSON) linking doc sections to source symbols and file paths.
- LLM-assisted auto-inference for the first pass; human confirm persists.

**Drafting layer (LLM):**
- An LLM API (Claude or GPT) constrained with the diff plus source truth plus the existing doc section (prompt: "update only the facts that changed").
- Structured output (JSON) for section-level edits, applied as PR patches.

**Integration and orchestration:**
- CI: GitHub Actions (or GitLab) runs detection on PRs and on schedule; posts draft PRs or comments.
- Git: all edits are commits and PRs for audit and rollback.
- Cost control: async background jobs, per-repo caps, kill switch via feature flag (LaunchDarkly, Unleash, or a simple config).

**Storage and state:**
- Git for docs; a small DB (Postgres or SQLite) for mapping metadata, metrics, and job state if needed.

**Minimal v1 (your own, better than generic doc platforms):** An installable, drift-first tool (CLI + MCP server) that does one thing well: detect and repair doc drift. Not a general knowledge base. Key properties:

- **Install, don't host:** `npm install` / `npx` (or a global CLI) wires it into the repo; runs locally against your files.
- **Bring your own key:** configure any LLM provider (OpenAI, Anthropic, or a local model via Ollama) with your own API key; optionally reuse a Codex or Claude subscription via their CLIs/APIs. No vendor lock-in.
- **Local-first:** detection and drafting run on your machine; nothing leaves except LLM calls you control. No hosted backend, no account required.
- **Drift-first, not doc-first:** purpose-built for drift detection and repair. Every feature serves that loop; not a generic doc platform.
- **Stronger mapping layer:** explicit doc-to-source-symbol mapping with confidence scores, the real asset for drift detection (beyond plain wiki-links).
- **Section-level granularity:** edits are surgical and reversible, not whole-page rewrites.
- **Signal ranking:** types and schemas first, tests second, changelogs as triage. Machine-readable truth prioritized over narrative changes.
- **Review UX as the product:** side-by-side diff with accept, edit, reject, plus a kill switch. The core experience, not an afterthought.
- **Metrics built in:** track false-positive rate and accept rate from day one; most tools ship blind.
- **Surfaces:** a CLI for detection and drafting (`self-heal check`, `self-heal draft`), an MCP server exposing tools to agents (drift list, draft, accept/reject), and a lightweight local preview UI for the side-by-side diff and mapping editor.
- **Publishing:** accepted fixes become commits or PRs against your repo; auto-merge off by default.
- **Open source:** MIT/Apache license, no hosted dependency.

## Open Questions

- Which signals best indicate drift (tests, types, changelogs)?
  - **Why ask**: the choice of signal determines precision and cost; weak signals create alert fatigue, strong signals may miss narrative drift.
  - **Recommendation**: ranked by reliability. (1) types and schemas (strongest, machine-readable contracts, cheap to compare); (2) tests (encode intended behavior, change means docs likely should too); (3) changelogs and release notes (coarse triage trigger); (4) code diffs on touched paths (heuristic, broad but noisy); (5) build and CI failures from runnable doc examples (direct breakage). Practical combo: types as hard signal, tests as behavior signal, changelogs as triage trigger. Start with types plus tests on one doc set.
- How to handle docs with no clear code counterpart?
  - **Why ask**: not every doc maps to code (concepts, runbooks, onboarding); forcing a mapping creates false confidence and ignores the real drift signals for those types.
  - **Recommendation**: don't force a code link. Classify by `doc_type` and apply different rules: conceptual and architecture docs map to decision records or owners (drift signal is design decision changed, not code); runbooks map to process (drift signal is commands still run, via CI smoke tests); onboarding and narrative docs use a staleness clock plus periodic human review (`last_verified` in frontmatter); tutorials and examples become runnable (doctests) so they break loudly on drift. Only enforce code-mapping plus drift detection on `api-reference` and runnable tutorials.
- What's the review UX (PRs, inline suggestions, or auto-merge with audit)?
  - **Why ask**: the review UX determines adoption and trust; too heavy (PRs for everything) kills velocity, too light (auto-merge) risks publishing wrong fixes.
  - **Recommendation**: staged. Start with PRs (builds trust, familiar, auditable), add inline suggestions for low-friction small edits, reserve auto-merge-with-audit for mechanical, high-confidence, tiny diffs (renamed params, version bumps). Autonomy scales with reversibility plus mechanicalness, not LLM quality.
- What is the unit of drift (whole doc, section, or single claim)?
  - **Why ask**: granularity determines mapping cost, review UX, and signal-to-noise; too coarse blocks good docs, too fine overwhelms reviewers.
  - **Recommendation**: section-level. A whole doc is too coarse (one stale line blocks the rest); a single claim is too fine (mapping cost explodes and reviews become noise). Section-level matches how docs are authored and reviewed, and keeps the mapping layer tractable.
- Who owns the mapping layer (manual annotation or auto-inference)?
  - **Why ask**: the mapping layer is the foundation of the whole system; without it you can't know what "reality" to compare a doc against, yet manual tagging has adoption friction and auto-inference has noise.
  - **Recommendation**: hybrid. Auto-infer a first pass (cheap, broad), let engineers confirm or correct with one click, and persist corrections as the source of truth. Over time confirmed mappings become a reliable asset and inference improves from them. Start manual-only on a tiny pilot to validate the signal before investing in inference.
- How do we measure "healing" quality (false positives, false negatives, reviewer accept rate)?
  - **Why ask**: without a baseline you can't tell whether the system actually reduces doc debt or just adds noise; a high false-positive rate erodes trust and a low accept rate means the drafter wastes reviewer time.
  - **Recommendation**: track detection metrics (false positive rate equals dismissed or ignored alerts divided by total; false negative rate via periodic manual sampling or proxy of doc-related support tickets and incidents; precision and recall per signal type to drop noisy ones) and remediation metrics (reviewer accept rate equals accepted-as-is vs. edited vs. rejected; time-to-fix; doc-related incidents and support tickets trending down). Baseline: manually label about 50 drift cases, compute precision and recall per signal, set an accept-rate threshold (at least 70% accept, at most 20% edit) before enabling auto-drafting; re-measure monthly.
- What is the rollback story for a wrong auto-drafted edit?
  - **Why ask**: auto-merge raises the stakes; a bad fix can ship silently, and without a fast rollback one bad auto-edit erodes trust in the whole system.
  - **Recommendation**: git is the rollback (every auto-drafted edit is a commit or PR, so revert is one command; keep auto-merged edits as individual commits, not squashed). Keep edits small and scoped (section-level) so reverts are surgical. Re-baseline after revert: revert the doc, re-snapshot the source truth, reset the drift clock so the system doesn't re-flag the now-correct doc. Tag auto-drafted commits (e.g., `auto-doc-fix`) to find and batch-revert if a prompt or model change was systematically wrong. Add a per-repo kill switch to disable auto-drafting instantly and fall back to "flag only" mode.
- Who pays for LLM calls (CI or async) and what is the latency budget?
  - **Why ask**: LLM calls cost money and take time; without a clear owner and budget, costs surprise teams and slow adoption, and synchronous drafting blocks the merge flow.
  - **Recommendation**: run async by default (background job on PR open or on schedule, post results as a PR comment or draft) to keep the merge flow fast; reserve sync for tiny high-value triggers (e.g., a schema change touching a mapped doc). Set a latency budget (draft available within \~5 minutes) and a cost cap (N calls per PR, monthly per-repo ceiling); drop low-confidence drafts rather than spend to refine. Attribute cost to the repo or team that owns the doc and make it visible in a dashboard.
- Is adoption opt-in per repo or an org-wide default?
  - **Why ask**: org-wide default risks blast radius from a buggy drafter or noisy signal; opt-in slows adoption but limits damage; repos also differ in doc maturity and signal availability.
  - **Recommendation**: opt-in per repo to start (lower risk, lets willing teams validate the signal and build trust, surfaces real edge cases before scaling). Provide a starter config (mapped doc types, enabled signals, auto-merge off by default) so opt-in is a one-line change. Graduate to org-wide default only after a few repos hit the accept-rate threshold and false-positive rate is low; keep a per-repo escape hatch even then. Tier by repo maturity: auto-enable on repos with strong CI and schema signals; leave narrative-heavy repos opt-in.
