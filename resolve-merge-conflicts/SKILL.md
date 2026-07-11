---

name: resolve-merge-conflicts
description: Resolve conflicts in an in-progress Git merge or rebase. Use when a repository has unmerged paths, conflict markers, or a paused merge/rebase that must be investigated, resolved, tested, staged, and continued without discarding either side's intended behaviour.
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Resolve Git merge and rebase conflicts

Resolve the repository's existing merge or rebase operation. Do not start a new merge or rebase.

## Safety rules

* Preserve unrelated working-tree and staged changes.
* Do not use destructive commands such as `git reset --hard`, `git checkout -- .`, `git restore .`, or force pushes.
* Do not resolve a conflict by choosing “ours” or “theirs” wholesale unless the history and intent clearly justify it.
* Do not invent new behaviour.
* Stage only resolved files and other changes intentionally required by the merge or rebase.

## 1. Inspect the operation

Determine whether Git is currently performing a merge or rebase.

Inspect:

* `git status`
* recent history and the commits involved
* `git diff --name-only --diff-filter=U`
* `git ls-files -u`
* staged, unstaged, and untracked changes
* each conflicting file and its surrounding code

Identify all conflict types, including content, add/add, modify/delete, rename, and file/directory conflicts.

If no merge or rebase is in progress and there are no unmerged paths, stop and report that there is no active conflict to resolve.

## 2. Establish the intent of both sides

For every conflicting file or logical change, determine why each side changed.

Use the strongest available sources:

1. The conflicting commits and their messages
2. The parent commits and nearby history
3. The base, ours, and theirs versions of the file
4. `git log`, `git show`, `git blame`, and related tests
5. Pull requests, issues, or tickets, when accessible
6. Relevant documentation and surrounding implementation

Do not rely only on the text between conflict markers.

Summarize the intent of each side before choosing a resolution when the reason is not obvious.

## 3. Resolve every conflict

Resolve each hunk so that both intended changes are preserved whenever they are compatible.

Also check for semantic conflicts: code may merge cleanly while one change invalidates assumptions made by the other.

Follow these rules:

* Remove all conflict markers.
* Preserve project style and existing abstractions.
* Update tests only when required to preserve the established behaviour of both sides.
* Do not introduce unrelated refactors.
* Do not silently choose one incompatible behaviour over another.

When the two intents are genuinely incompatible and the repository history does not establish the correct result, stop before staging that resolution. Explain:

* the two competing behaviours,
* why they cannot both be preserved,
* the practical consequences of each option, and
* the specific decision needed from the user.

## 4. Validate the resolution

After all conflicts are resolved, confirm that Git reports no unmerged paths and search the affected files for leftover conflict markers.

Run:

```sh
./gradlew check
```

If the command cannot run because the wrapper is absent, non-executable, or the repository is not a Gradle project, report that clearly rather than substituting an unrelated command.

For each failure:

1. Determine whether it was introduced by the conflict resolution or exposed by the merged changes.
2. Fix merge-related failures while preserving both original intents.
3. Re-run the relevant failing check, then re-run `./gradlew check`.
4. Do not silently fix unrelated pre-existing failures; report them separately.

## 5. Complete the Git operation

Review the final diff before staging.

Stage the resolved files and any intentional supporting changes. Do not stage unrelated user changes.

Then complete the operation:

* For a merge, use `git merge --continue` when supported, or create the merge commit with the existing/default merge message.
* For a rebase, run `git rebase --continue`.

A rebase may stop on additional commits. Repeat the inspection, intent analysis, resolution, and validation process until the rebase finishes.

Do not amend unrelated commits or force-push unless the user explicitly requests it.

## Final report

Report:

* whether a merge or rebase was completed,
* the conflicts resolved,
* how both sides' intent was preserved,
* validation commands and results,
* any unresolved or unrelated failures, and
* the resulting commit or rebased commit range.
