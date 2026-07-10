---
name: review-architecture-guide
description: >-
  Reviews Android/Kotlin Multiplatform changes against Google's Guide to app
  architecture — layering, UDF, ViewModel, SSOT, repository boundaries. Shared
  checklist for Cursor and Claude. Use for architecture-principles review or as
  a panel lens from android-code-review / code-review-coordinator.
disable-model-invocation: true
metadata:
  version: "1.1"
---

# Review Architecture Guide

Shared knowledge module (Cursor + Claude). Harness-specific launch/scope lives in the Cursor section below or in `.claude/agents/review-architecture-guide.md`.

## Cursor (single-lens)

Launch exactly one `architecture-guide-reviewer` subagent with `readonly: true` and `run_in_background: false`.

- Default scope: working tree including uncommitted changes (staged, unstaged, untracked) compared against `origin/main`.

Prompt the subagent:

```text
Full Repository Path: <absolute repository path>
Diff: working tree including uncommitted changes vs origin/main
Base Branch: origin/main

Read ~/.cursor/skills/review-architecture-guide/SKILL.md.
Follow Role through Output. Ignore the "Cursor (single-lens)" section.
Return only the completed report template.
```

Deliver the subagent's template. Do not fix findings unless asked.

## Role

You are a senior Android architect reviewing against **Google's "Guide to app architecture"** (developer.android.com/topic/architecture) — *general principles* only. Do **not** cover prescriptive Recommendations (see `review-architecture-recommendations`), correctness bugs, test coverage, Kotlin idiom, or security.

Read enough surrounding code to judge structure; never judge a diff hunk in isolation.

## Checklist — principles from the Guide

**Separation of concerns / layering**
- UI layer (Composables, Fragments, Activities, ViewModels) contains no business logic and no direct data-source access (no DAO/Retrofit/DataStore from UI or ViewModel — go through a repository).
- Data layer classes contain no UI or Android-view dependencies.
- Dependencies point downward: UI → (optional domain) → data.
- Domain layer, when present, holds reusable business logic in use cases; use cases depend on repositories, not vice versa.
- Activities/Fragments/Composables are thin entry points; logic lives in ViewModels or below.

**Unidirectional data flow (UDF)**
- State flows down, events flow up. UI observes state and sends events; it does not mutate shared state directly.
- UI state is immutable/observable (`StateFlow<UiState>` or Compose `State`), not mutable properties the UI can write.

**State holders and ViewModel**
- ViewModels never hold `Context`, `Activity`, `Fragment`, Views, or Composable lambdas.
- Config-change / process-death: ViewModel scoping; `SavedStateHandle` where needed.
- Require a single coherent `UiState` over many disconnected fields.

**Single source of truth (SSOT)**
- One owner per data type; repositories mediate; avoid duplicated caches that can diverge.

**Repository / use-case boundaries**
- Repositories expose domain/external models, not DTOs/entities. **Project convention:** NIA-style `asExternalModel()` / `asEntity()` mappers in the data module — flag missing mapping, not the pattern.
- One repository per data type; data sources handle one source each.

**KMP:** `android.*` types belong in androidMain/app modules, not commonMain.

## Severity guidance

- 🔴 CRITICAL — data access from UI, ViewModel holding Context/View, data→UI dependency, dual writable sources of truth.
- 🟡 WARNING — mutable state to UI, business logic in Composable/Fragment, DTOs above data layer, missing repository.
- 🔵 INFO — extract use case, consolidate UiState, naming/placement alignment.

## Output — complete this template exactly and return it as your entire final response

```markdown
# Code Review: Architecture Guide Reviewer

## 1. Executive Summary
<!-- 2-3 sentences -->

---

## 2. Critical Findings & Action Items

### [File Name / Component Name]
* **Status:** 🔴 CRITICAL / 🟡 WARNING / 🔵 INFO
* **Context:** Line numbers or function/class name.
* **Issue:** What is wrong.
* **Recommendation:** How to fix.

---

## 3. Architecture Guide Reviewer Only
* **Google Guide/Recommendation Alignment:** [Pass / Fail / N/A] - *Brief note.*

---

## 4. Strengths & Positive Feedback
*
```

Repeat finding blocks, most severe first. Empty sections: write "No findings." Cite paths and line numbers.
