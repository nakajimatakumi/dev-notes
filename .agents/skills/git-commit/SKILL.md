---
name: git-commit
description: Safely create a Git commit in this project without pushing. Use when the user asks to commit, create a commit, stage and commit changes, or save the current work in Git while leaving the remote unchanged.
---

# Git Commit

Create one safe, focused commit in this project. Do not push.

## Repository guard

1. Resolve the repository root with `git rev-parse --show-toplevel`.
2. Confirm the root is the project that contains this skill under `.agents/skills/git-commit`.
3. Stop if Git is unavailable, the directory is not a worktree, or the resolved root is a different repository. Never accept an alternate repository path for this skill.

## Workflow

1. Inspect `git status --short`, `git diff`, and `git diff --cached`.
2. Identify task-related files. Preserve unrelated user changes and flag suspicious secrets, credentials, large generated files, or conflict markers.
3. Stage only task-related paths with explicit `git add -- <paths>`. Use `git add -A` only when the user explicitly requests all changes and inspection shows it is safe.
4. Review `git diff --cached --check` and `git diff --cached`. Stop if nothing is staged.
5. Choose a concise imperative commit subject that describes the staged change. Follow an established repository convention when visible.
6. Run `git commit -m <subject>`. Honor hooks; never bypass them. If a hook changes files, inspect and restage only appropriate changes before retrying.
7. Verify with `git show --stat --oneline --decorate HEAD` and `git status --short`.
8. Report the commit hash, subject, committed paths, and remaining uncommitted changes.

## Safety

- Never push, amend, reset, restore, clean, rebase, force, delete branches, or change Git configuration unless the user explicitly requests that separate action.
- Never include secrets. Redact sensitive values from output.
- Do not create an empty commit unless explicitly requested.

