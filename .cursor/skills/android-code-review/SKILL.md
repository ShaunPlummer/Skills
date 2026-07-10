---
name: android-code-review
description: >-
  Orchestrates a multi-angle Android/Kotlin Multiplatform code review for Cursor.
  Dispatches review-architecture-guide, review-architecture-recommendations,
  review-test-coverage, and review-kotlin-coroutines as read-only subagents, plus
  built-in Bugbot and Security Review. Synthesizes one consolidated Markdown report.
  Use for full code review, pre-merge review, or multi-perspective review of local changes.
metadata:
  version: "1.1"
---

# Android Code Review (Cursor coordinator)

You are an **editor, not a reviewer**: orchestrate specialists and merge reports. Do not add your own findings or resolve disagreements.

**Shared knowledge** — `review-*` skills installed at `~/.cursor/skills/`. **Bugs and security** use Cursor built-ins — follow `review-bugbot` and `review-security` for launch/retry.

## Step 1 — Establish scope once

- User-named diff mode / range / files win.
- Default: **`branch changes`** (merge-base with default branch; committed + staged + unstaged).
- **`uncommitted changes`** only when the user asks for dirty/local-only review.

Record absolute repo path and diff mode. Do not pre-compute the diff for Bugbot/Security Review.

Optional: if `<repo>/.cursor/skills/android-code-review/conventions.md` exists, pass its path to every custom reviewer.

## Step 2 — Dispatch six reviewers in parallel

One batch of Task calls, all `readonly: true`, all `run_in_background: false`.

| Reviewer | Type | Loads |
|---|---|---|
| Architecture Guide | `generalPurpose` | `review-architecture-guide` skill (checklist sections only) |
| Architecture Recommendations | `generalPurpose` | `review-architecture-recommendations` |
| Test Coverage | `generalPurpose` | `review-test-coverage` |
| Kotlin & Coroutines | `generalPurpose` | `review-kotlin-coroutines` |
| Bugbot | `bugbot` | `review-bugbot` skill |
| Security Review | `security-review` | `review-security` skill |

### Custom lens prompt

```text
Full Repository Path: <absolute repository path>
Diff: <branch changes | uncommitted changes>
Base Branch: <only if non-default base>

Read ~/.cursor/skills/<review-*-name>/SKILL.md.
Follow Role through Output.
Return only the completed report template.
```

### Built-ins

Follow `review-bugbot` and `review-security` with the same repo path and diff mode. Collect findings for the consolidated report (do not stop at their one-line summaries).

**Independence:** never feed one reviewer's output into another. Retry once on failure; then "Reviewer did not complete".

## Step 3 — Overlaps

Cross-index same file+line/function across 2+ reviewers. Surface severity disagreements as-is. Include Bugbot/Security table rows.

## Step 4 — Consolidated report

```markdown
# Consolidated Code Review

**Scope:** <diff mode, repo path>
**Reviewers:** 6 dispatched — <list any that did not complete>

## Overall Summary
<!-- severity counts -->

### Conflicting or Overlapping Findings
<!-- or none -->

---

## Architecture Guide Reviewer
<template>

---

## Architecture Recommendations Reviewer
<template>

---

## Bug Reviewer (Bugbot)
<!-- summary + Severity | Location | Finding table -->

---

## Unit Test Coverage Reviewer
<template>

---

## Kotlin & Coroutines Reviewer
<template>

---

## Security Reviewer
<!-- summary + Severity | Location | Finding table -->
```

Response = report. Optional write: `code-review-report.md` in repo root only. Do not fix or rerun unless asked.

## Single-lens

Use `review-architecture-guide`, `review-architecture-recommendations`, `review-test-coverage`, `review-kotlin-coroutines`, `review-bugbot`, or `review-security` directly.
