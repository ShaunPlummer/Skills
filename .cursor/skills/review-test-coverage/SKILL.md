---
name: review-test-coverage
description: >-
  Reviews unit test adequacy for Android/Kotlin changes. Thin Cursor wrapper;
  loads the shared review-test-coverage skill. Dispatched by android-code-review
  or invoked directly.
---

# Review Test Coverage (Cursor single-lens)

Launch exactly one `generalPurpose` subagent with `readonly: true` and `run_in_background: false`.

- Default diff: `branch changes`. Use `uncommitted changes` only when the user asks for dirty/local-only review.

Prompt the subagent:

```text
Full Repository Path: <absolute repository path>
Diff: <branch changes | uncommitted changes>
Base Branch: <only when reviewing against a non-default base>

Read ~/.cursor/skills/review-test-coverage/SKILL.md.
Follow Role through Output.
Return only the completed report template.
```

Deliver the subagent's template. Do not fix findings unless asked.
