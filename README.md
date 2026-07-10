# Code review skills

Agent-agnostic review checklists in the root; agent-specific orchestration in `.cursor/` and `.claude/`.

## Shared knowledge (`review-*/`)

| Skill | Lens |
|---|---|
| `review-architecture-guide/` | Google's Guide to app architecture |
| `review-architecture-recommendations/` | Android Recommendations |
| `review-test-coverage/` | Unit test adequacy |
| `review-kotlin-coroutines/` | Kotlin + coroutines/Flow idiom |

Each root `SKILL.md` holds the checklist and report template only — no agent-specific instructions.

## Agent-specific orchestration

| | Claude Code | Cursor |
|---|---|---|
| Entry (full panel) | `.claude/agents/review-coordinator.md` | `.cursor/skills/android-code-review/SKILL.md` |
| Specialist runners | `.claude/agents/review-*.md` (thin wrappers, `tools:`, Skill tool) | `.cursor/skills/review-*.md` (thin wrappers, Task subagents) |
| Diff default | `git diff <origin/default SHA>` (single-ref, WT-inclusive) | `branch changes` / `uncommitted changes` |
| Bugs | Built-in `/code-review` | Built-in Bugbot (`review-bugbot`) |
| Security | Same built-in review surface | Built-in Security Review (`review-security`) |

## Layout

```
review-*/                      # agent-agnostic checklists
.cursor/skills/
├── android-code-review/       # Cursor multi-lens coordinator
└── review-*/                  # Cursor single-lens wrappers
.claude/
└── agents/                    # Claude coordinator + thin reviewers
```

This repo is an **archive/documentation** copy — not an install via symlink. Copy what you need to `~/.cursor/skills/` or `~/.claude/` to make skills live.

## Usage

**Cursor full panel:** "Run android-code-review on my branch changes"  
**Claude full panel:** "Use the code-review-coordinator"  
**Single lens:** invoke the matching `.cursor/skills/review-*` skill (Cursor) or `.claude/agents/*-reviewer` agent (Claude).

## Intentionally removed

- Fat `agents/` checklist duplicates
- Custom `bug-review` / `android-security-review` (favour built-ins)
- Old `*-review/` directory names (renamed to `review-*`)
