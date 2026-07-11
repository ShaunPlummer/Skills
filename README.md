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
| Entry (full panel) | `.claude/agents/review-coordinator.md` (`code-review-coordinator`) | `.cursor/skills/android-code-review/SKILL.md` |
| Specialist runners | `.claude/agents/review-*.md` (thin wrappers; Skill tool loads root `review-*`) | `.cursor/skills/review-*/SKILL.md` (combined Cursor launch + checklist; Task subagents) |
| Diff default | Working tree including uncommitted vs `origin/main` | Working tree including uncommitted vs `origin/main` |
| Bugs | Built-in `/code-review` | Built-in Bugbot (`review-bugbot`) |
| Security | Same built-in review surface | Built-in Security Review (`review-security`) |

## Layout

```
review-*/                      # agent-agnostic checklists (Claude Skill tool)
.cursor/skills/
├── android-code-review/       # Cursor multi-lens coordinator
└── review-*/                  # Cursor single-lens (launch + checklist)
.claude/
└── agents/                    # Claude coordinator + thin reviewers
```

## Install

This repo is an archive copy — not live via symlink, and never run in place. Skills are always copied out to a real skills directory before use, either a target project's own `.claude/skills/` (project-scoped, checked into that project's version control) or the user-level `~/.claude/skills/` (global, available across all projects). Pick whichever matches how you want the review lenses shared.

**Cursor:** copy each folder under `.cursor/skills/` into the project's `.cursor/skills/` or the user-level `~/.cursor/skills/` (overwrites same-named skills). Those folders are self-contained; do not copy root `review-*/` into either location or you will lose the Cursor launch section.

**Claude Code:** copy root `review-*/` into the project's `.claude/skills/` or the user-level `~/.claude/skills/`, and `.claude/agents/*.md` into the corresponding `.claude/agents/` (project-level or `~/.claude/agents/`).

## Usage

**Cursor full panel:** "Run android-code-review"  
**Claude full panel:** "Use the code-review-coordinator"  
**Single lens (Cursor):** invoke `review-architecture-guide`, `review-architecture-recommendations`, `review-test-coverage`, or `review-kotlin-coroutines`  
**Single lens (Claude):** invoke `architecture-guide-reviewer`, `architecture-recommendations-reviewer`, `test-coverage-reviewer`, or `kotlin-coroutines-reviewer`
