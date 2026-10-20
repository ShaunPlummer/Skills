# Code review skills

Shared review checklists, with platform coordinators in `.cursor/` and `.claude/`.

## Review skills

| Skill | Lens | Invocation |
|---|---|---|
| `review-architecture-guide/` | Google's Guide to app architecture | Full panel |
| `review-architecture-recommendations/` | Android Recommendations | Full panel |
| `review-test-coverage/` | Unit test adequacy | Full panel |
| `review-kotlin-coroutines/` | Kotlin and coroutines/Flow idiom | Full panel |
| `review-kotlin-coroutines-cancellation/` | Cooperative cancellation and cleanup | Full panel |
| `review-coroutines-flow-tests/review-coroutines-flow-tests/` | Coroutine and Flow test correctness | Direct |

Each `SKILL.md` contains that lens's checklist and output template.

## Orchestration

Each platform has one coordinator. It dispatches parallel specialist subagents, and each prompt names the `review-*` skill to load.

| | Claude Code | Cursor |
|---|---|---|
| Entry | `.claude/skills/android-code-review/SKILL.md` | `.cursor/skills/android-code-review/SKILL.md` |
| Specialists | Five parallel general-purpose subagents | Five parallel read-only subagents |
| Diff default | Working tree, including uncommitted changes, vs `origin/main` | Working tree, including uncommitted changes, vs `origin/main` |
| Bugs | `/code-review`, run separately | Bugbot (`review-bugbot`), included in the full panel |
| Security | `/security-review`, run separately | Security Review (`review-security`), included in the full panel |

## Layout

```
review-architecture-guide/
review-architecture-recommendations/
review-test-coverage/
review-kotlin-coroutines/
review-kotlin-coroutines-cancellation/
review-coroutines-flow-tests/
└── review-coroutines-flow-tests/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── assets/icon.svg
.cursor/skills/
└── android-code-review/
    ├── SKILL.md
    └── report-template.md
.claude/skills/
└── android-code-review/
    ├── SKILL.md
    └── report-template.md
```

## Install

**Cursor:** copy the five full-panel directories into `~/.cursor/skills/`. Copy `review-coroutines-flow-tests/review-coroutines-flow-tests/` to `~/.cursor/skills/review-coroutines-flow-tests/`. Copy `.cursor/skills/android-code-review/` to `~/.cursor/skills/android-code-review/`.

**Claude Code:** copy the five full-panel directories into `~/.claude/skills/`. Copy `review-coroutines-flow-tests/review-coroutines-flow-tests/` to `~/.claude/skills/review-coroutines-flow-tests/`. Copy `.claude/skills/android-code-review/` to `~/.claude/skills/android-code-review/`.

## Usage

- **Cursor full panel:** "Run android-code-review"
- **Claude full panel:** "Run android-code-review"
- **One lens:** invoke `review-architecture-guide`, `review-architecture-recommendations`, `review-test-coverage`, `review-kotlin-coroutines`, `review-kotlin-coroutines-cancellation`, or `review-coroutines-flow-tests` directly.
