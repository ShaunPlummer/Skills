---
name: android-code-review
description: >-
  Orchestrates a multi-angle Android/Kotlin Multiplatform code review by
  dispatching architecture, recommendations, test-coverage, and
  Kotlin/coroutines reviewers as parallel subagents, then synthesizing one
  consolidated Markdown report. Bug and security review are a separate
  dedicated pass. Use for full code review, pre-merge review, or
  multi-perspective review of local changes.
metadata:
  version: "1.0"
---

# Android Code Review (Claude coordinator)

You are an **editor, not a reviewer**: dispatch specialists, merge their reports, surface overlaps. Do not add your own findings or resolve disagreements between reviewers.

**Shared knowledge** — each specialist's actual criteria and report template live in its own agent-agnostic skill (`review-architecture-guide`, `review-architecture-recommendations`, `review-test-coverage`, `review-kotlin-coroutines`). This skill only orchestrates; it holds no review criteria of its own.

**Bugs and security** are out of scope for this panel — point the user at `/code-review` or `/security-review`, run separately.

## Step 1 — Resolve scope once

- User-named revision range or file list takes precedence.
- Otherwise, compare the current working tree (including staged, unstaged, and untracked) against `origin/main`.
- Resolve once; pass the identical scope to every reviewer below.

**Adapter note (Git):** `git fetch origin main` → `git rev-parse origin/main` for the base SHA → diff as `git diff <SHA>`; untracked files via `git status --porcelain`.

## Step 2 — Dispatch four reviewers in parallel

Use the Agent tool with `subagent_type: general-purpose`, one call per row below, all issued in a single message so they run in parallel. Each reviewer is blind to the others' output.

| Reviewer | Skill to load |
|---|---|
| Architecture guide reviewer | `review-architecture-guide` |
| Architecture recommendations reviewer | `review-architecture-recommendations` |
| Test coverage reviewer | `review-test-coverage` |
| Kotlin & coroutines reviewer | `review-kotlin-coroutines` |

### Dispatch prompt template

Fill in `<skill-name>` per row above, and pass the resolved scope from Step 1:

```text
Repository: <absolute repo path>
Scope: <resolved base SHA / file list, plus the Adapter note above if no scope was given>

Load the `<skill-name>` skill via the Skill tool and apply everything from Role through Output in it.
This review is read-only: use Read, Grep, Glob, and read-only Bash only. Do not edit, write, or run mutating commands.
Return only the completed report template as your entire response — no extra commentary.
```

On failure: retry once, then record that role as "did not complete" and continue. Never invent findings for a role that didn't return, and never feed one reviewer's output into another.

## Step 3 — Identify overlaps

Cross-index findings by file and location across two or more reviewers. Surface disagreements as-is; do not adjudicate.

## Step 4 — Synthesize report

Normalize, deduplicate, and cross-reference findings. Do not create new findings or resolve conflicts between reviewers.

Fill in the template in `report-template.md` (alongside this file).

**Optional side effect:** write the report to `<repo-root>/build/reports/code-review-report.md` only when explicitly requested.

## Single-lens

To run just one lens, load that `review-*` skill directly instead of dispatching this coordinator.
