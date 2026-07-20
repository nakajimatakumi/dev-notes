---
name: git-commit-push
description: Safely review changes, create a focused Git commit, and push it from this project to its configured remote in one workflow. Use when the user asks to commit and push, save and publish changes, or comprehensively send the current project work to the connected Git repository.
---

# Git Commit and Push

Create a safe, focused commit in this project and push it to the configured remote.

## Repository guard

1. Resolve the repository root with `git rev-parse --show-toplevel`.
2. Confirm the root is the project that contains this skill under `.agents/skills/git-commit-push`.
3. Stop if Git is unavailable, the directory is not a worktree, or the resolved root is a different repository. Never accept an alternate repository path for this skill.

## Commit phase

1. Inspect `git status --short`, `git diff`, and `git diff --cached`.
2. Preserve unrelated user changes. Flag suspicious secrets, credentials, large generated files, or conflict markers.
3. Stage only task-related paths with explicit `git add -- <paths>`. Use `git add -A` only when the user explicitly requests all changes and inspection shows it is safe.
4. Review `git diff --cached --check` and `git diff --cached`. Stop if nothing is staged unless suitable commits already exist and the user clearly wants only those pushed.
5. Choose a concise imperative subject consistent with repository conventions and run `git commit -m <subject>`. Honor hooks and never bypass them.
6. Verify the commit with `git show --stat --oneline --decorate HEAD`.

## Push phase

1. Inspect `git branch --show-current`, `git remote -v`, and `git status --branch --short`.
2. Stop on a detached HEAD, unresolved conflicts, or ambiguous destination.
3. Resolve the upstream and review outgoing commits with `git log --oneline @{upstream}..HEAD` when one exists.
4. Push normally to the configured upstream. If none exists, use `origin` only when it is the single unambiguous destination and run `git push -u origin <current-branch>`.
5. Never force-push or push tags unless separately and explicitly requested.
6. Verify with `git status --branch --short`.

## Failure handling and report

- If commit fails, do not push.
- If push fails, preserve the successful local commit and clearly report that it remains unpushed.
- Never amend, reset, restore, clean, rebase, delete branches, or change Git configuration without separate explicit authorization.
- Report the commit hash and subject, committed paths, remote and branch, push result, and any remaining local changes.
- Redact credentials and remote URLs containing embedded secrets.
