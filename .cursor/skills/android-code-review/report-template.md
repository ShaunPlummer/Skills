`<template>` = that reviewer's own completed report, returned verbatim per its `review-*` skill's Output section (see Step 2's dispatch prompt).

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
