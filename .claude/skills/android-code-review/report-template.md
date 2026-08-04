`<template>` = that reviewer's own completed report, returned verbatim per its `review-*` skill's Output section (see Step 2's dispatch prompt).

```markdown
# Consolidated Code Review

**Scope:** origin/main vs working tree (including uncommitted), <file count> files
**Reviewers:** 4 lenses — <list any that did not complete>
**Out of scope:** bug correctness and security (run `/code-review` or `/security-review` separately)

## Overall Summary
<!-- severity counts -->

### Conflicting or Overlapping Findings
<!-- or none -->

---

## Architecture Guide Review
<template>

---

## Architecture Recommendations Review
<template>

---

## Unit Test Coverage Review
<template>

---

## Kotlin & Coroutines Review
<template>
```
