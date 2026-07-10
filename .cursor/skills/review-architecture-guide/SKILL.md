---
name: review-architecture-guide
description: >-
  Reviews Android/Kotlin Multiplatform changes against Google's Guide to app
  architecture. Thin Cursor wrapper; loads the shared review-architecture-guide
  skill. Dispatched by android-code-review or invoked directly.
---

# Review Architecture Guide (Cursor single-lens)

Launch exactly one `generalPurpose` subagent with `readonly: true` and `run_in_background: false`.

- Default diff: `branch changes`including any uncommitted changes.

Prompt the subagent:

```text
Full Repository Path: <absolute repository path>
Diff: <branch changes | uncommitted changes>
Base Branch: <only when reviewing against a non-default base>

Read ~/.cursor/skills/review-architecture-guide/SKILL.md.
Follow Role through Output.
Return only the completed report template.
```

Deliver the subagent's template. Do not fix findings unless asked.
