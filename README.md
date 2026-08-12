# Code review skills

Agent-agnostic review checklists in the root; agent-specific orchestration in `.cursor/` and `.claude/`.

## Shared knowledge (`review-*/`)

| Skill | Lens |
|---|---|
| `review-architecture-guide/` | Google's Guide to app architecture |
| `review-architecture-recommendations/` | Android Recommendations |
| `review-test-coverage/` | Unit test adequacy |
| `review-kotlin-coroutines/` | Kotlin + coroutines/Flow idiom |
| `review-kotlin-coroutines-cancellation/` | Cooperative cancellation and cleanup |

Each root `SKILL.md` holds the checklist and report template only — no agent-specific instructions.

## Orchestration

Each platform has one coordinator skill that dispatches parallel specialist reviewer subagents by prompt — the prompt tells each subagent which root `review-*` skill to load.

| | Claude Code | Cursor |
|---|---|---|
| Entry (full panel) | `.claude/skills/android-code-review/SKILL.md` | `.cursor/skills/android-code-review/SKILL.md` |
| Specialist dispatch | Agent tool, `subagent_type: general-purpose`, prompted to load the matching root `review-*` skill | Task tool, `readonly: true` subagents, prompted to load the matching `review-*` skill (installed at `~/.cursor/skills/`) |
| Diff default | Working tree including uncommitted vs `origin/main` | Working tree including uncommitted vs `origin/main` |
| Bugs | Built-in `/code-review` | Built-in Bugbot (`review-bugbot`) |
| Security | Built-in `/security-review` | Built-in Security Review (`review-security`) |

## Layout

```
review-*/                      # agent-agnostic checklists (loaded via Skill tool)
.cursor/skills/
└── android-code-review/       # Cursor multi-lens coordinator
.claude/skills/
└── android-code-review/       # Claude multi-lens coordinator
```

## Install

This repo is an archive copy — not live via symlink.

**Cursor:** copy root `review-*/` into `~/.cursor/skills/`, and `.cursor/skills/android-code-review/` into `~/.cursor/skills/android-code-review/`.

**Claude Code:** copy root `review-*/` into `~/.claude/skills/`, and `.claude/skills/android-code-review/` into `~/.claude/skills/android-code-review/`.

## Usage

**Cursor full panel:** "Run android-code-review"  
**Claude full panel:** "Run android-code-review" (loads the `android-code-review` skill)  
**Single lens (either platform):** invoke `review-architecture-guide`, `review-architecture-recommendations`, `review-test-coverage`, `review-kotlin-coroutines`, or `review-kotlin-coroutines-cancellation` directly
