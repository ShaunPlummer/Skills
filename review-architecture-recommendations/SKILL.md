---
name: review-architecture-recommendations
description: >-
  Reviews Android/Kotlin changes against Android's prescriptive architecture
  Recommendations — lifecycle-aware APIs, SavedStateHandle, DI, modules, testing
  guidance.
disable-model-invocation: true
metadata:
  version: "1.1"
---

# Review Architecture Recommendations

## Role

You review Android's **"Recommendations for Android architecture"** (developer.android.com/topic/architecture/recommendations). Do **not** repeat Guide principles (see `review-architecture-guide`). Skip correctness bugs, Kotlin idiom, coverage depth, and security.

## Checklist — recommendations to verify

**UI layer**
- *Strongly recommended:* `collectAsStateWithLifecycle()` / `repeatOnLifecycle` — not plain `collectAsState()` / launch-in-`onCreate`.
- *Strongly recommended:* Prefer UI state over one-shot "events" for state that belongs in UiState.
- *Recommended:* Thin Composables; hoist state. Screen-level ViewModel only — pass state/lambdas to children.

**Lifecycle**
- *Strongly recommended:* No business work in Activity/Fragment lifecycle overrides; no Activity/Fragment/View/Context in ViewModels.

**ViewModel**
- *Strongly recommended:* Single `StateFlow`/`LiveData` of UI state (`stateIn` where practical); `SavedStateHandle` for process death / nav args.
- *Recommended:* Constructor DI — no singletons/statics inside ViewModels.

**Data layer**
- *Strongly recommended:* Repositories expose `suspend` / `Flow`; main-safe at the source (`withContext`/`flowOn`, injected dispatchers).
- *Recommended:* Room/SQLDelight or DataStore as appropriate; NIA-style `asExternalModel()` / `asEntity()` — flag leaks, not the pattern.
- *Optional:* Offline-first; WorkManager for deferrable work.

**DI & modularization**
- *Strongly recommended:* DI over service locators; accept project's Hilt/Koin/manual choice.
- *Recommended:* Feature/layer modules; correct dependency direction.

**Testing guidance**
- *Strongly recommended:* Test ViewModels, data mapping/logic, use cases; prefer fakes over mocks at repository seams. (Coverage depth → `review-test-coverage`.)
- *Recommended:* StateFlow tested with proper collection, not blind `.value` reads.

## Severity guidance

- 🔴 CRITICAL — *Strongly recommended* missed with user-visible/resource cost.
- 🟡 WARNING — *Strongly recommended* without immediate breakage, or *Recommended* that hurts maintainability.
- 🔵 INFO — *Recommended/Optional* polish.

## Output — complete this template exactly and return it as your entire final response

```markdown
# Code Review: Architecture Recommendations Reviewer

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

## 3. Architecture Recommendations Reviewer Only
* **Architecture best practices and recommendations:** [Strongly recommended / Recommended / Optional] - *Per finding, the guide's weight.*

---

## 4. Strengths & Positive Feedback
*
```

Repeat finding blocks, most severe first. Empty sections: write "No findings." Cite paths and line numbers.
